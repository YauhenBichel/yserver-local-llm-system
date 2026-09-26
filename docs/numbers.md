# Numbers

Every number here was measured on my own computer. Each table says how. They describe one machine and one
setup, so use them as an example, not as a benchmark of the models.

## Speed of the coding model

Tool: [inference-benchmarker](https://github.com/huggingface/inference-benchmarker), 19 September 2026. It ran
as a Kubernetes job on the same computer, through the gateway, with one virtual user.

| Measurement | Result |
|---|---|
| Requests | 132, with 0 failures |
| Time to the first token, 200-token prompt, model loaded | 0.64 s (90 % under 0.73 s) |
| Generation speed | 50 tokens per second (20 ms between tokens) |
| Generation speed with an 8,000-token context | 45 tokens per second |
| Reading an 8,000-token prompt | 13.7 s, about 590 tokens per second |
| Three requests at the same time | first token 13 times slower, total speed +6 % |

A note on the tool: its ready-made profiles first load the server to its maximum. On a server that handles one
request at a time, this measures only the queue. I set the options by hand.

## The prompt cache in long agent sessions

Source: my agent runner saves the prompt-reading time of every decision. 188 decisions from 38 tasks, 11 to 17
September 2026, almost all on the 120B reasoning model. No new test was needed; the data was already there.

**The rule.** A turn "lost the cache" when its reading time, less the time that its new tokens explain, is at
least half of a full read of the part that should have been cached, and at least 3 seconds. Only turns with at
least 3,000 cached tokens are judged: 150 turns.

| Measurement | Result |
|---|---|
| New tokens in a later turn | 74 (median), 243 (90th percentile) |
| Largest prompt in the whole record | 17,620 tokens, of a 64,000-token window |
| Turns that kept the cache | 104, with 211 s of reading in total, median 1.25 s |
| Turns that lost the cache | 46, with 1,185 s of reading in total, median 25.6 s, plus a model load each time |
| Share of turns that lost the cache | **31 %** |
| Their share of all reading time | **85 %** |

| Prompt size | Kept the cache: median | Lost the cache: median |
|---|---|---|
| 3,000 to 4,000 tokens | 0.97 s (53 turns) | 12.5 s (9 turns) |
| 4,000 to 8,000 tokens | 1.26 s (37 turns) | 23.1 s (21 turns) |
| 8,000 to 12,000 tokens | 1.37 s (11 turns) | 35.5 s (15 turns) |

Above 12,000 tokens there are only four turns, which is too few to report.

The result does not depend on the exact thresholds of the rule: nine combinations give 83 to 85 % of the reading
time. **A correction:** the first version of this page said "one third of the turns, 92 %". That came from a
quicker rule, which also counted turns that were slow because they really had much new text. The rule above
subtracts that time.

**Limits.** This is one machine, one week and mostly one model, and it is an observation, not an experiment.
The reading speed of the model is estimated from its first turns. Controlled experiments are planned.

The cause of the lost caches was one of my own guards, which unloaded the model once a minute. See lesson 5 in
[lessons.md](lessons.md).

## Loading and changing models

| Measurement | Result |
|---|---|
| Loading the 52 GB coding model, nothing else running | about 30 s |
| The same, while another program keeps loading another model | 41 to 53 s, before every request |
| Model loads on a quiet day | 6 to 8 |
| Model loads on the three bad days | 130 to 208 each |

## Ten days, first day and tenth day

| | Day 1 | Day 10 |
|---|---|---|
| Where the models run | CPU | built-in GPU |
| The same 30B model | 42 tokens per second | 94 tokens per second |
| Main coding model | 30B | 80B, at 50 tokens per second |
| Context | 4,096 tokens | 64,000 tokens |
| A short request | 7.5 s | 0.6 s |
| When the computer stops responding | it stays stopped | it restarts itself and alerts my phone |

## Embeddings: my service or a dedicated server

Tool: [text-embeddings-inference](https://github.com/huggingface/text-embeddings-inference), CPU build, against
my current in-process service. Same model (bge-m3), same texts.

| Measurement | Result |
|---|---|
| Speed, four kinds of input | 1.3 to 3.3 times faster |
| Are the vectors the same? | yes: cosine similarity 0.99999999999 |
| Memory of the dedicated server | about 4.5 GB, all the time |
| Decision | I keep my current service. Nobody waits for embeddings here |
| A free improvement found on the way | on a CPU, eight separate requests are faster than one request with eight long texts |

## Against cloud models

My own set of 16 graded tasks, 2 points each.

| System | Score |
|---|---|
| Claude models | 28 to 30 of 32 |
| My system | clearly lower. It failed on special cases, such as quoting rules in CSV files, and once returned incomplete JSON |

So I use my system for the work it does well: first drafts, routine code, summaries, and tests for a function
that I paste in. I make the decisions myself.

## Local models as code reviewers

Four reviews of the same change with [serge](https://github.com/huggingface/serge). The tool worked on the
first attempt. The models did not: one short approval, one list of twelve comments that all said the code was
good, one that did not finish in time, and one that reported errors which did not exist (the same code passes
the linter, the type checker and 413 tests). I removed the review workflow.
