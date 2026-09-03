---
title: New side project - Prequel
date: 2026-09-03
categories:
tags:
---
When reviewing Claude Code's work, I discovered something interesting...

After years (decades?) of reviewing pull requests, my brain is quite accustomed to Github's PR review interface. Just seeing the dual-pane layout and standard Github UI widgets moves my head into a "okay, we're reviewing code now" mental zone, and I'm able to focus and comment on what I'm seeing.

My head is so well trained, in fact, that I found myself doing something pretty ridiculous: I would create pull requests to review Claude's output, add my commentary, and then go back into Claude Code to tell it to fix what I had commented on.

And so was born my first entirely vibe-coded project: prequel. Prequel spins up a local node web server and renders all of your local changes just as though it were a pull request. You can add comments, and then invoke a `/prequel` skill in Claude Code. Claude will then dutifully read your comments, and address them one by one (and leave a comment if there's some reason it can't). 

It's available on NPM if y'all want to try it out: https://www.npmjs.com/package/@mdesjardins/prequel

<img alt="Prequel screenshot" src="/images/prequel-screenshot.jpg" border="0"/>
