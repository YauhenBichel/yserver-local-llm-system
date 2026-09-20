# Words used here

First draft by my own local model, then corrected by hand: four of its fourteen lines were wrong for this system.

- **gateway**: A small service that receives every request first, checks it, and forwards it to the right model.
- **role**: A name for a job, such as "coding" or "vision", that points to the model which does this job today.
- **token**: A small piece of text, about three quarters of a word. Models count text in tokens.
- **context window**: The largest number of tokens that a model can read in one request. Here: 64,000.
- **prompt cache**: The model server keeps what it has read. If the next prompt starts with the same text, it reads only the new part.
- **time to the first token**: The time between sending a prompt and receiving the first piece of the answer.
- **mixture-of-experts model**: A large model that uses only a small part of its weights for each token, so it is fast for its size.
- **quantisation**: Storing a model's numbers with less precision, for example Q4_K_M, so that it needs less memory.
- **unified memory**: The CPU and the GPU share one memory. A large model fits, but each can take memory from the other.
- **embedding**: A list of numbers that represents the meaning of a text, used to find related texts.
- **queue**: The line of requests that wait for the model server, which handles one request at a time.
- **agent harness**: The program around a model that gives it a task, runs its tools, checks the results and saves every step.
- **deterministic check**: A check that a program runs, with the same result every time: a command, a file, a test. Not a model's opinion.
- **watchdog**: A timer inside the computer's chipset. If the system stops resetting it, the timer restarts the computer.
