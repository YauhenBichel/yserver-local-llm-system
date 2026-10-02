# One small computer, eight models

A talk version of this repository: what running my own LLM system at home taught me, and why most of it was not
about models.

[![The cover slide. Select it to watch the three-minute video](slides/01-cover.png)](https://cdn.jsdelivr.net/gh/YauhenBichel/yserver-local-llm-system@main/talk/one-small-computer-3-minutes.mp4)

**[Watch the three-minute video](https://cdn.jsdelivr.net/gh/YauhenBichel/yserver-local-llm-system@main/talk/one-small-computer-3-minutes.mp4)** · [the script](NARRATION.md) · download: [with narration](one-small-computer-3-minutes.mp4), [without sound](one-small-computer-3-minutes-silent.mp4)

GitHub does not play a video file that is stored in a repository, so the "watch" link opens the same file
through jsDelivr, a public CDN that serves this repository's files, and your browser plays it.

The video is an overview of the talk, made from the slides. Its narration is a computer voice reading my
script. The demo slide in it shows [llm-hops](https://github.com/YauhenBichel/llm-hops) with the tool's
built-in demo data; in the talk the demo runs live, on real requests.

## The talk

One AMD server at home, with 128 GB of memory shared by the processor and its built-in GPU, serves open
language models to Claude Code, my editor and my own agents. The first token arrives in 0.64 seconds, and the
coding model writes 50 tokens a second. Getting there took ten days, and most of that time went on operations,
not models.

The talk is the honest account of that work: a prompt that was cut silently and came back with another
request's answer and status 200, 35 to 42 GiB of GPU memory that appeared in no report, a night when the
machine stopped for eight hours while every check said it was fine, and a week in which my own safety guard
unloaded the model once a minute, so that one agent turn in three took twenty times longer to read.

Every number is measured, and each has its method and its limits in [docs/numbers.md](../docs/numbers.md).

**What you take away**

1. How to put one gateway in front of local models: names mapped to roles, a single queue, and a prompt-length
   check that rejects a long prompt instead of cutting it silently.
2. The operations checklist for shared-memory AI machines: GPU memory the normal tools do not show, a hardware
   watchdog that really loads, and alerts that live outside the box.
3. Why a lost prompt cache, not a long context, is what slows local agents, and how to see it with per-hop
   request tracing.

**Length:** 20 minutes plus questions. There is also a five-minute lightning version built on one story, the
prompt that was too long.

## The slides

| | |
|---|---|
| ![Cover](slides/01-cover.png) | ![Why I built it](slides/02-why.png) |
| ![The server](slides/03-computer.png) | ![Every request goes through one gateway](slides/04-architecture.png) |
| ![Roles and models](slides/05-roles.png) | ![How fast it is](slides/06-numbers.png) |
| ![One part models, nine parts operations](slides/07-ratio.png) | ![The server answered someone else's question](slides/08-long-prompt.png) |
| ![Shared memory hides GPU memory](slides/09-hidden-memory.png) | ![The night it stopped](slides/10-the-night.png) |
| ![Who else is using the GPU?](slides/11-cache.png) | ![Local models cannot review code yet](slides/12-code-review.png) |
| ![A finding is an assumption until a test confirms it](slides/13-assumption.png) | ![The write-up and five tools, all public](slides/14-tools.png) |
| ![Close](slides/15-close.png) | |

## Where each slide comes from

| Slides | Source in this repository |
|---|---|
| The server, roles and models | [README](../README.md) |
| Every request goes through one gateway | [docs/architecture.md](../docs/architecture.md), [docs/flows.md](../docs/flows.md) |
| How fast it is, the cache numbers | [docs/numbers.md](../docs/numbers.md) |
| The seven lessons | [docs/lessons.md](../docs/lessons.md) |

## Inviting this talk

I am based in London and glad to give it at a meetup. Write to yauhen.bichel@gmail.com.
