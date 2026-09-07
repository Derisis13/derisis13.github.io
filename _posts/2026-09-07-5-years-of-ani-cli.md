# Reflecting on (almost) five years of ani-cli development

I was invited to ani-cli's maintainer team on 25th of January, 2022 by port19.
My first contribution to the project was merged on the 6th of November, 2021.
From this August, I'm resigning from my position as maintainer, and I wish to elaborate why.

Before going any further, I'd like to thank my time for the rest of the team.
I got to know some seriously amazing people, who I continue to love.
We do not harbor any harsh feelings for each other, and I enjoyed working with them.
Without my time with them, working on this project and even just casually chatting, I wouldn't be who I am right now.


## Why I joined ani-cli

When port19 invited me to the maintainer team, I was quite surprised.
He had just recently taken on the project as lead, and saw potential in my prior work.
At this time, ani-cli was very immature, it had non-posix parts, lots of repeating code, and overall bad practices.
I on the other hand have just started figuring out shell scripting after a video on Youtube has brough ani-cli to my attention.
I didn't know better, and my earlier PRs were similarly full of immature issues, that had to be ironed out over the following months.

My initial role was quality assurence.
This mostly meant refactoring and testing by hand: ani-cli will always be too small for proper testing, and I also knew nothing about CI pipelines at that time.
I also didn't know anything about conventional commits (sadly picked up by LLMs, but ani-cli introduced them before vibecoding became a thing), git best practices and usecases, and static analysis.
Maintaining ani-cli has also given me a lot of general coding collaboration skills that followed better standards than 99% of OSS projects with multiple developers and than most people at small companies (e.g. my current workplace).

Then as the project matured (v2, v3, v4), our initial roles became blurred.
The team has evolved as well; we had to say a sad goodbye to Ray, and welcomed Vorlie as a fellow maintainer.
The repo has gathered more than 13500 stars by now, my Copr package has over 10000 downloads (and more than 30000 total package downloads, whatever that means), and we also have a discord server with almost 3000 members, of which 20-30 are regulars.
In the end we had very similar understanding of the codebase, and even though we weren't active simultaneously, we carried a shared load.
It was a particularly light load (thanks to the v4 provider being stable for *years*, which I think was a miracle); but it was shared mostly equally.


## Primary reason: disintrest in the project

But when allanime started acting up, and we found that re-writing the scraper for a different site is easier than solving their challenge in shell, I no longer felt urged to fix it.
I did start a posix implementation of the allanime encryption, but it was exhausting (maths hasn't been my strong suite for a while, and doing it in shell is a massive chore).

We have agreed on a partial rewrite, meaning a major version release, but while the others were busy coding up the new scraper, I didn't take part.
It was not out of spite, (at this point I didn't even realize that they used LLMs, but more on that later); it was because my interests have changed, and ani-cli was no longer a focus for me.

When I realized this, I asked the others for a month - as a deadline - to elaborate on my status as a maintainer.
During this period I felt relief in my decision, the responsibility of the project (that has more than enough capable maintainers) no longer weighing on my shoulders.


## Secondary reason: disgust with LLMs

I have written about AI previously on this blog, but to summarize it in one word, my feelings towards them is currently disgust.
Ani-cli 5.0.0 is the first release which employed LLM generated code, so far only little snippets.
Things are complicated for the project as of now, there was [talk about openly embracing AI](https://github.com/pystardust/ani-cli/pull/1787), which they retracted.
Additionally, [using Claude for automated reviews](https://github.com/pystardust/ani-cli/pull/1861) (which was also retracted) would've been an absolute taboo to me.

Because I dragged this post's publication out for so long, I'm already too outside of the loop to really grasp what the situation is.
In the end, it doesn't matter to me: my change of interests is the reason I'm leaving, and it's not impacted that much by whichever way the others are taking the project.


## Moving forward

While my journey as an ani-cli maintainer has came to an end, I'm not going away.
First, having finished university, my job and private life is demanding more than ever: my free time is sparse.
Most of it is being channeled into my main hobby, which I picked up around the same time I started ani-cli: being a radio amateur.
If I'll have another major contribution to open-source, then it'll most likely be aligned with this field.

I'm really greatful that I've been part of ani-cli, it has taken me to places I've never imagined.
I don't know if I'll ever see such success in another pet project.
But even so, it's time for me to say goodbye to ani-cli.

