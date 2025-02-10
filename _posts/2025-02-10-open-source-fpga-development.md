---
layout: post
title: "How I got started with open-source FPGA development"
tag: "FPGA"
---

In the software world, open-source toolchains are taken for granted.
In the FPGA/hardware world, the situation is not as good, but with the right choice of FPGA, it is feasible.
Allow me to tell my tale about how I succeeded in bringing up my Artix devboard with only open-source programs.


# Writing VHDL in Neovim

My go-to text editor is Neovim.
I've been using it for years (I think I picked it up in 2022), and it stuck with me.
I'm not particularly good at it (I only know the basic keybinds), but it's already better than anything else I've tried.
The most important tools for VHDL are the language server: `vhdl-ls` (also known as `rust_hdl`), the Treesitter VHDL grammar, and my snippets.
My config—based on NvChad—can be found in my dotfiles repository.

For the sake of trying out the toolchains, I made the simplest possible LED blinking example:
```vhdl
-- Test entity for synthesis

library ieee;
use ieee.std_logic_1164.all;
use ieee.NUMERIC_STD.all;

entity blink is
    port (
        led_o  : out std_logic;
        clk    : in  std_logic;
        areset : in  std_logic
    );
end entity blink;

architecture rtl of blink is
    signal counter   : unsigned(23 downto 0); -- around 1 Hz with 12 MHz oscillator
    signal led_state : std_logic;
begin

    L_BLINK_PROC: process(clk)
    begin
        if rising_edge(clk) then
            if areset = '1' then
                counter <= (others => '0');
                led_state <= '0';
            else
                counter <= counter + 1;
                if counter = 0 then
                    led_state <= not led_state;
                end if;
            end if;
        end if;
    end process L_BLINK_PROC;
    led_o <= led_state;

end architecture rtl;
```
I also made a simple testbench so I could try simulation too:
```vhdl
-- Testbench for the blink example

entity tb_blink is
end entity tb_blink;

library ieee;
use ieee.std_logic_1164.all;

architecture tb of tb_blink is
    signal clk      : std_logic := '0';
    signal areset_n : std_logic;
    signal led      : std_logic;
begin
    areset_n <= '1', '0' after 100 ns;
    L_STIM: process
    begin
        for i in 0 to 2**24 + 5 loop
            wait for 10 ns;
            clk <= '1';
            wait for 10 ns;
            clk <= '0';
        end loop;
        wait;
    end process L_STIM;

    L_DUT: entity work.blink
        port map(
            led_o    => led,
            clk      => clk,
            areset   => areset
        );
end architecture tb;
```
The two files are named `blink.vhd` and `tb_blink.vhd`.
For `vhdl-ls`, I also made a descriptor file (`vhdl-ls.toml`):
```toml
[libraries]
defaultlib.files = [
    "*.vhd",
]
```


# Simulation with GHDL and GTKWave

Apart from creating my sources, simulating was one of the easiest steps—it just worked.
The only requirements here are GHDL for elaboration and running the simulation, and GTKWave for viewing the output waveform.
The process is well documented in GHDL's documentation (see #Sources).
All I had to do was:
```sh
mkdir ./workdir
ghdl -a --workdir=./workdir blink.vhd tb_blink.vhd
ghdl -e --workdir=./workdir tb_blink
ghdl -r --workdir=./workdir tb_blink --wave=wave.ghw
```
The first GHDL command (`-a`) analyzes the source files into the directory we just created (`./workdir`).
The second GHDL command (`-e`) elaborates the top-level entity, creating the simulation binary.
The third one runs the simulation, writing the output waveform into `wave.ghw`.

Do note that for analysis, source files are referenced, but for elaboration and running, entity names have to be specified instead of filenames.
Additionally, elaboration and running can be combined into a single command:
```sh
ghdl --elab-run --workdir=./workdir tb_blink --wave=wave.ghw
```
To view the output, I could open GTKWave graphically and then open the file from the picker, or run it from the CLI like this (this still opens a GUI window):
```sh
gtkwave wave.ghw
```
GTKWave is an OK viewer; at least I don't have to re-run the simulation when I want to look at a new signal, unlike with ModelSim.


# Synthesis with GHDL and Yosys

But GHDL isn't just a simulator.
It can also do—albeit experimental—synthesis and technology mapping by hooking into Yosys.
This is so much in development that not many packages are provided—I was lucky that someone had already made a [Copr repo](https://copr.fedorainfracloud.org/coprs/rezso/HDL/) for it, so I didn't have to compile it on my Fedora machine.
I also saw an AUR package for `ghdl-yosys-plugin`, so following along on Arch is probably easy too.
I do not know anything about Debian/Ubuntu; there may be a PPA, but if you have to build from source, check this page: [https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/toolchain\_arty.md](https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/toolchain_arty.md)

To keep the clutter away from source files, I made a build directory:
```sh
mkdir ./build
```
To use the GHDL Yosys plugin, I launched Yosys like this:
```sh
yosys -m ghdl
```
Unless you specify `-m ghdl`, its plugin will be missing when you run Yosys.
In its console, I first issued `ghdl blink.vhd -e blink` to elaborate my source(s), with `blink` as the top-level entity.
Then, I ran `synth_xilinx -json ./build/blink.json` to synthesize a netlist into a JSON file, using technology mapping to the Xilinx 7 family.

Alternatively, a single script for the same commands can be written like:
```sh
yosys -m ghdl -p "ghdl blink.vhd -e blink; synth_xilinx -json ./build/blink.json"
```
One downside is that VHDL 2008 is not—or not completely—supported.
To stay safe, I omitted the `std=08` flag everywhere.


# Place & route with NextPNR

Once I had a netlist, I proceeded to use NextPNR to place and route it to actual components in the FPGA.
My device is an Artix 7, specifically the XC7A35T CPG236-1 as part of a Digilent CMOD A7 devboard.
I first made the mistake of using the `nextpnr-xilinx` fork, which is not maintained regularly and thus much behind `nextpnr` and refused to work for me.
A bit of information that was unnecessarily hard to find is that the himbaechel backend of NextPNR (which is built into the `nextpnr` package provided by the copr repo) supports Xilinx 7 FPGAs—including my Artix 7.

An `xdc` file is required to map top-level inputs and outputs to physical pins.
I derived this from the [CMOD A7's xdc file](https://github.com/Digilent/digilent-xdc/blob/master/Cmod-A7-Master.xdc) provided by digilent:
```xdc
set_property LOC L17 [get_ports {clk}]
set_property IOSTANDARD LVCMOS33 [get_ports {clk}]
create_clock -add -name sys_clk_pin -period 83.33 -waveform {0 41.66} [get_ports {clk}]

set_property LOC A17 [get_ports {led_o}]
set_property IOSTANDARD LVCMOS33 [get_ports {led_o}]

set_property LOC A18 [get_ports {areset}]
set_property IOSTANDARD LVCMOS33 [get_ports {areset}]
```
I spent an embarrassing amount of time debugging an error caused by comments (and semicolons) at the end of the xdc file's lines.
These completely break NextPNR's xdc parser, so they had to go.

I also used the xdc file to specify my timing constraints (this time the 12 MHz clock) but due to limitations of NextPNR isn't used for static timing analysis (STA).
To check if my design can operate at the required frequency, I had to specify an additional argument.
NextPNR doesn't know the `-add`, `-name` and `-waveform` arguments (I guess they are used by Vivado for simulation), but only shows a warning if they are left in.
My place and route command looked like this:
```sh
nextpnr-himbaechel --device xc7a35tcpg236-1 --json ./build/blink.json -o xdc="Cmod-A7-Master.xdc" --write ./build/blink_routed.json  -o fasm=./build/blink.fasm --router router2 --freq 12
```
The `--router` argument had little effect on my design, but it was in the NextPNR GitHUB repo's example code so I left it there assuming it does no harm.
The `--freq` argument specifies the clock frequency (in megahertz) for STA.
The output file I'm going to work with in the following section is `blink.fasm`.


# Bitstream generation with Project X-Ray

To write the configuration to the FPGA I needed it in a loadable format.
For Xilinx devices it's a bitstream, also known as `.bit` files.
I achieved this in two steps: first I converted the `fasm` file to `frames`, then `frames` to `bit`.
The software collection that's going to help me generate programming files for the Artix 7 (and for Xilinx FPGAs in general) is called Project X-Ray.
I preformet the first step with Xray's `fasm2frames` tool, which is sadly broken in the package form the copr repo.
As a dirty fix, I cloned the [projectxray GitHUB repo](https://github.com/f4pga/prjxray) (to `~/.local/bin/build_stage/`) then used the script in its sources like this:
```sh
python ~/.local/bin/build_stage/prjxray/utils/fasm2frames.py --db-root /usr/share/xray/database/artix7 --part xc7a35tcpg236-1 ./build/blink.fasm ./build/blink.frames
```
I still had to source database from the `projectxray-data` package, as it's not stored in the GitHUB repo directly.

For step two, I employed the tool `xc7frames2bit` that worked from the installation.
This saved me some time as I would have had to compile this program otherwise (since it's written in c, unlike `fasm2frames`).
The command I used for the conversion is what you'd expect:
```sh
xc7frames2bit --part_file  /usr/share/xray/database/artix7/xc7a35tcpg236-1/part.yaml --frm_file ./build/blink.frames --output_file ./build/blink.bit
```
Note that depending on the installation of Project X-Ray, the database directory may be different from mine, find or locate commands can be used to determine the exact path.
Same goes for the part number: different FPGAs from different families need different databases/partfiles.


# Programming with openFPGAloader

At last I have my bitstream.
Getting this on the FPGA required the `openFPGAloader` package, also provided by the copr repo.
For now I loaded the configuration into SRAM, but the CMOD A7 also comes with a serial flash memory to store the config.
The command I used to write the sram looked like this:
```sh
/usr/bin/openFPGALoader -b cmoda7_35t ./build/blink.bit
```
And flashing would've looked like this:
```sh
/usr/bin/openFPGALoader -b cmoda7_35t -f blink.bit
```
Once this was complete, I had a led blinking at approximately 0.5 Hz.
Project success!


# Conclusion

While it's a far cry from proprietary integrated development environments, fully open-source FPGA synthesis is possible.
It's limited to Xilinx, Lattice and some other manufacturers (and even here the supported devices/families are limited too).
The weak points are no or limited VHDL 2008 support, the difficulty of hunting down every component (and their documentation).
Thus the barrier to entry is quite high, it took me roughly a day to get it all sorted out with prior knowledge of FPGAs and minimal prior knowledge of the software used.
Advanced features like post-layout synthesis, IP core wizzards and graphical pin planners are either non-existent or not practical.

On the other hand, these tools feel very fast, especially the synthesis workflow.
I haven't done any benchmarks, but especially with Modelsim's free edition slowing down significantly over 10000 lines GHDL should be able to compete with it.
Also, since all of them (except GTKWave) are command-line tools, their outputs are mostly in plain text, they fit the general tools of open-source development rather well (eg. git and make).
This make them feel more ergonomic for a nerd like me, who even edits text in the terminal.
All in all I wouldn't use them in my dayjob (they are just not on par with vendor IDEs), but they'll do fine for my hobby projects.


# sources:
nextpnr-himbaechel usage: https://github.com/YosysHQ/nextpnr/tree/master/himbaechel/uarch/xilinx/examples/arty-a35
general workflow: https://github.com/BrunoLevy/learn-fpga/blob/master/FemtoRV/TUTORIALS/toolchain_arty.md https://github.com/BrunoLevy/learn-fpga/blob/master/Basic/ARTY/ARTY_blink/makeit.sh
CMOD A7 docs: https://digilent.com/reference/_media/reference/programmable-logic/cmod-a7/cmod_a7_rm.pdf
CMOD a7 xdc file: https://github.com/Digilent/digilent-xdc/blob/master/Cmod-A7-Master.xdc
GHDL Simulation: https://ghdl.github.io/ghdl/using/Simulation.html
GHDL synthesis: https://github.com/ghdl/ghdl-yosys-plugin, https://wiki.f-si.org/images/b/b3/Ghdl-FSiC2022.pdf
Project X-Ray: https://github.com/f4pga/prjxray
