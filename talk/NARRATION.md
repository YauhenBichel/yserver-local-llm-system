# The script of the three-minute video

What is said over each slide in [one-small-computer-3-minutes.mp4](one-small-computer-3-minutes.mp4). The video's narration is a computer voice reading this text.

## 1. Cover

Hello. I'm Yauhen Bichel, a software engineer in London. This is a proposal for a talk called One small computer, eight models. It is about running language models on my own machine at home, and about why most of what I learned was not about models.

## 2. The computer

The whole system is one small desktop with no separate graphics card. The processor and the built-in GPU share 128 gigabytes of memory. That is why a 52 gigabyte coding model fits on it and stays loaded all day, with a context of 64,000 tokens.

## 3. Architecture

Claude Code, my editor and my scripts all speak to one address. Every request goes through one gateway. It checks access, maps the name the client asked for to a model, checks that the prompt fits, and keeps a single queue. Around it sits the part that took most of the time: memory watchers, a hardware watchdog, alerts and backups.

## 4. How fast it is

These numbers are measured, not estimated. 0.64 seconds to the first token. 50 tokens a second. 132 benchmark requests with no failures. On day one, a short request took seven and a half seconds. On day ten, it took 0.6.

## 5. One part models, nine parts operations

Here is the main idea of the talk. One part models, nine parts operations. Choosing a model took hours. Making the computer safe to leave alone took days.

## 6. The server answered someone else's question

The best story is this one. I sent a prompt that was longer than the model's limit. The server cut it silently and returned status 200, which means success. But the answer was the answer to the previous request, word for word. On a shared server, one user could have seen another user's answer. I reported it, I run a fixed build, and my gateway now rejects a prompt that is too long. The rule is simple: reject, do not cut.

## 7. Who else is using the GPU?

The second story is about speed. I expected long prompts to be the problem. They were not. When the server keeps the conversation in its cache, a turn is read in about one second. When the cache is lost, it takes about twenty-five. 31 percent of turns lost the cache, and they took 85 percent of the reading time. The cause was my own safety guard, unloading the model once a minute. It was all in the logs. Nobody had counted.

## 8. llm-hops, the live demo

To see problems like that, I wrote a tracing tool called llm-hops. This is a real request shown as a waterfall: the router, the gateway, the prompt check and the model, with the time of every hop. In the talk I will show this live, with a real request, a rejected prompt and a model switch.

## 9. Six tools, all open source

Everything in the talk is open source: the write-up with the architecture, the flows and the numbers, the tracing tool, and the small checks for failures that stay silent. People can take them home and use them the same evening.

## 10. Close

Why is this useful for your audience? Many people are starting to run models themselves, and the hard part is not the model. This talk gives them real numbers, honest mistakes and working tools. It runs twenty minutes, or five as a lightning talk. Thank you for considering it.

## When each slide starts

| Slide | Starts at | Lasts |
|---|---|---|
| 1 | 0:00 | 18 s |
| 2 | 0:17 | 21 s |
| 3 | 0:38 | 22 s |
| 4 | 1:00 | 20 s |
| 5 | 1:19 | 12 s |
| 6 | 1:32 | 26 s |
| 7 | 1:58 | 27 s |
| 8 | 2:24 | 20 s |
| 9 | 2:44 | 15 s |
| 10 | 3:00 | 18 s |

Total: 3:18.
