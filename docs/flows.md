# Flows

Each flow has a diagram and a short explanation. The diagrams are written in Mermaid, so GitHub draws them.

1. [A request, from client to model](#1-a-request-from-client-to-model)
2. [From a name to a model](#2-from-a-name-to-a-model)
3. [The prompt-length check](#3-the-prompt-length-check)
4. [The queue, and why it prefers the loaded model](#4-the-queue-and-why-it-prefers-the-loaded-model)
5. [The agent loop](#5-the-agent-loop)
6. [Local model or cloud model](#6-local-model-or-cloud-model)
7. [How heavy jobs share the GPU](#7-how-heavy-jobs-share-the-gpu)
8. [How the system was built: an agent, a plan, and checks it cannot influence](#8-how-the-system-was-built)
9. [When the computer stops](#9-when-the-computer-stops)
10. [Backups](#10-backups)

---

## 1. A request, from client to model

![A request through the system](../diagrams/system-request-flow.png)

```mermaid
sequenceDiagram
    participant C as Client
    participant G as Gateway
    participant Q as Queue
    participant M as Model server (GPU)
    C->>G: request, Anthropic or OpenAI format
    G->>G: check access
    G->>G: name to role, role to model
    G->>G: add memory, if the role uses it
    G->>G: check the prompt length
    G->>Q: wait for the slot
    Q->>M: forward the request, unchanged except for the model name
    M-->>C: the answer streams back through the gateway
    G->>G: write one log line: model, waiting time, tokens, result
```

The gateway forwards requests and does not rebuild them. It changes the model name and passes the data through.
It does not interpret streaming or tool calls, because it could get them wrong.

## 2. From a name to a model

```mermaid
flowchart LR
    R["The client asks for a name"] --> K{"What kind of name?"}
    K -- "a name of a cloud model,<br/>for example claude-sonnet-4-5" --> P["Pattern table"]
    K -- "role:coding" --> RO["Role"]
    K -- "auto" --> CT{"What is in the request?"}
    K -- "a real local model name" --> D["Used as it is"]
    P --> RO
    CT -- image --> V["vision or OCR role"]
    CT -- audio --> AU["audio role"]
    CT -- everything else --> CO["coding role"]
    V --> RO
    AU --> RO
    CO --> RO
    RO --> MO["The local model of that role"]
    D --> MO
```

Two steps, name to role and role to model, mean that a change of model is one line in one file. No client
needs a change.

## 3. The prompt-length check

![The gateway's check](../diagrams/story1-gateway-check.png)

```mermaid
flowchart TD
    S["A request arrives"] --> I["Ignore images and audio when counting"]
    I --> E["Estimate the tokens from the number of characters"]
    E --> N{"Near the limit?"}
    N -- "clearly under" --> OK["Send it to the model"]
    N -- "near or over" --> X["Ask the model server for an exact token count"]
    X --> F{"Does it fit?"}
    F -- yes --> OK
    F -- no --> ER["Return the normal API error:<br/>prompt is too long"]
```

Why this exists: a model server once cut a long prompt to half of its limit, returned status 200, and sent
back the answer to the previous request. The full story is lesson 2 in [lessons.md](lessons.md).

Two details cost me a day each:

- An image in a request is a long text in base64. My first check counted it as normal text and refused every
  image. Test both sides of a limit, and test every kind of input.
- The check is a second line of defence. The first line is the model server itself, which now refuses instead
  of cutting (see the `ollama` branch in the README).

## 4. The queue, and why it prefers the loaded model

The model server handles one request at a time, and one large model fits in memory at a time. Two facts follow:

- More parallel requests do not make it faster. They only wait. So the queue does not remove waiting, it
  makes waiting visible: every answer carries its waiting time.
- A change of model costs 20 to 50 seconds of loading, and the new model starts with an empty cache. Two
  programs that alternate between two models pay this on every request.

```mermaid
flowchart TD
    F["The slot becomes free"] --> W{"Is anybody waiting?"}
    W -- no --> ID["The slot stays free"]
    W -- yes --> O["Look at the oldest waiter"]
    O --> S{"Does it need the model<br/>that is loaded now?"}
    S -- yes --> GO["Serve it"]
    S -- no --> T{"Has it waited longer<br/>than the limit, 2 minutes?"}
    T -- yes --> GO
    T -- no --> A{"Is there a waiter for<br/>the loaded model?"}
    A -- yes --> J["Serve that one first"]
    A -- no --> GO
```

Four waiters for models A, B, A, B then cost one change of model, not three. Nobody waits longer than the limit
plus one request because of this rule.

## 5. The agent loop

![The agent loop](../diagrams/story5-agent-loop.png)

```mermaid
flowchart LR
    OB["Observe<br/>the last result"] --> DE["Decide<br/>the model fills a JSON form"]
    DE --> PO{"Allowed by<br/>the policy?"}
    PO -- "no: the reason goes back<br/>to the model" --> OB
    PO -- "needs a human" --> HU["Wait for approval"]
    HU --> AC
    PO -- yes --> AC["Act<br/>run one tool in the workspace"]
    AC --> SA["Save the step"]
    SA --> FI{"Did the model<br/>say finish?"}
    FI -- no --> OB
    FI -- yes --> VE["Verify<br/>automatic checks of the goal"]
    VE -- "checks fail" --> OB
    VE -- "checks pass" --> DONE["Done"]
    SA --> BU{"Budget left?<br/>turns, tokens, time"}
    BU -- no --> STOP["Stop with a reason"]
```

The decisions that matter:

- **The model fills a form; it does not call tools.** The form is a JSON schema with a fixed list of actions.
  The model server forces the answer into this form. In my tests, one local model never used tool calls and
  another invented tool names. With a fixed list, neither can happen.
- **"Finish" is a request, not a fact.** The task ends only when automatic checks pass.
- **The history only grows.** Nothing in it is edited. The model server can then reuse what it has already
  read, and a turn costs about 1 second instead of 12 to 35.
- **Every step is saved.** After a crash the task continues from the last step.
- **A budget stop is a stop reason, not a success.**

## 6. Local model or cloud model

![One endpoint, two models](../diagrams/story3-one-endpoint-two-models.png)

On my laptop, a small router sits in front of both my local system and the cloud. My tools talk to one address.

```mermaid
flowchart TD
    P["A prompt"] --> PR{"Privacy rules:<br/>must this text stay here?"}
    PR -- yes --> LO["Local model. Always"]
    PR -- no --> RR{"Routing rules"}
    RR -- local --> LO
    RR -- cloud --> CK{"Is a cloud key configured?"}
    CK -- no --> LO2["Local model, and the answer says why"]
    CK -- yes --> CL["Cloud model"]
```

The privacy rules are a hard gate and they come first. They are a file that I can read, not code that I must
trust. I can also ask where a prompt would go, without sending it.

## 7. How heavy jobs share the GPU

The CPU and the GPU share one memory. When the GPU part is full, the driver moves data into normal memory, and
this memory appears in no report. Language models, image generation and training cannot all run together.

```mermaid
sequenceDiagram
    participant J as A heavy job
    participant L as Lease
    participant S as Model server
    participant W as Memory watcher
    J->>L: may I use the GPU?
    L->>S: unload the resident model
    L-->>J: yes, for this job
    loop every 2 seconds
        W->>W: GPU memory of each program, and hidden memory
        W-->>J: stop, if a limit is passed
    end
    J->>L: finished
    Note over S: the resident model loads again on the next request
```

Four things work together: a fixed upper limit for the hidden memory, a kernel control group that protects the
GPU memory of each workload, the rule that every heavy job asks first, and the watcher.

One more rule, learned later: a guard must know who is using the model before it unloads it. One of my guards
unloaded a large model once a minute in the middle of agent tasks, because the model alone was over the guard's
limit. See lesson 5.

## 8. How the system was built

An AI coding agent did much of the building, on the computer itself.

```mermaid
flowchart TD
    PL["A written plan with numbered steps"] --> ST["The agent takes the next step"]
    ST --> WK["It works, without administrator rights"]
    WK --> CH{"An automatic check<br/>that the agent cannot influence"}
    CH -- fails --> RT{"Attempts left?<br/>Money left?"}
    RT -- yes --> WK
    RT -- no --> WA["Stop and wait for me"]
    CH -- passes --> RE["Write a report for this step"]
    RE --> ST
    WK -. "needs administrator rights" .-> SC["A script that first prints<br/>what it will do. I run it myself"]
```

There is a spending limit for each attempt and for the whole plan. Every step leaves a report. One of the steps
was the speed test in the README: I started it, and then I switched off my laptop.

## 9. When the computer stops

![The night my server stopped](../diagrams/story6-the-night.png)

```mermaid
flowchart TD
    FR["The computer stops responding"] --> WD["Nobody resets the watchdog timer"]
    WD --> RB["The timer, inside the chipset, restarts the computer<br/>in about 30 seconds"]
    RB --> BT["The system starts"]
    BT --> AL["The alert check sees that the last stop was not clean"]
    AL --> PH["An alert goes to my phone"]
    FR --> LP["My laptop sees that the computer is silent"]
    LP --> LN["After 5 minutes it tells me"]
```

A watchdog is a timer inside the computer's chipset. The system must reset it every few seconds. A program
cannot do this job, because a program stops together with the computer.

The alerts run on the computer that they watch, so a watcher outside it is needed. Today that is my laptop. A
heartbeat to an outside service, which reports when the heartbeat stops, is built and not connected yet.

## 10. Backups

```mermaid
flowchart LR
    U["My work on the computer"] --> N["Every night: an encrypted backup"]
    SY["System settings"] --> N
    N --> SC["A second copy on my laptop"]
    N --> ST["A stamp: last success"]
    ST --> AC["The alert check reads the stamp,<br/>not the timer's status"]
    SC --> ST2["A stamp: last good copy"]
    ST2 --> AC
```

The backup had failed every night for three days before I noticed. The timer was "active" the whole time. Now
the alert reads when the last backup really succeeded.
