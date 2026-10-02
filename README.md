This is an archive of my "standard CLAUDE.md files". It is very much designed for me and my code style, and I would expect you to find things you disagree with in it, but that's OK. You're welcome to cherrypick! It tends to get updated intermittently at best.

Things of note:

* I'm pretty sure this has lineage back to Claude 4.0. There's probably stuff in here that could be stripped out today. I haven't done that yet.
* The hostile-review workflow is an absolute token hog. It's also *really* useful, it increases quality dramatically.
* The hostile-review workflow's model suggestion has been tested as of Fable 5.1/Opus 5.5/Sonnet 5.0/Haiku 4.5, using Opus as a "host". Yes, Opus deciding that Opus is the best reviewer is very Obama-giving-Obama-a-medal. When I was using Fable 5.1, Opus 5.0 was a better reviewer than Fable 5.1 was, though I guess technically Opus 5.5 might be worse (I haven't tested this). I never use Sonnet as the primary model so I never tested those; if you do, you should do your own analysis.
* My normal method for adding code policy is to recognize an issue I keep running into, tell Claude to fix it, tell Claude to find more examples of it in the codebase and propose fixes, point out some issues in those choices, eventually settle on a set that we both agree should be changed, ask Claude to write up the general long-term policy that we arrived on, and then edit that as well. As a result, most of this was written *by Claude*. It is . . . very Claudey. I've been kinda afraid of tinkering with it.
* Yes, it's probably bigger than it should be, both because of "stuff that could be stripped out" and "very Claudey".

Whole thing is licensed under [CC0](https://creativecommons.org/publicdomain/zero/1.0/) if you're into that. If you want to use this but for some reason don't like CC0, let me know and I'll multilicense it under something else.
