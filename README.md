# Learn to Think Like an Architect

> **For every developer — junior or senior — who wants a strong foundation in system design.**

Most developers learn how to write code.  
Architects learn how to think about systems.

This series bridges that gap.

It is not about memorizing design patterns or drawing boxes on a whiteboard. It is about developing the **mental models, instincts, and habits** that separate engineers who build things that work from engineers who build things that *last*.

---

## Who Is This For?

| You Are | What You Will Get |
|---|---|
| **Junior Developer** | A mental framework that most seniors took years to build through painful incidents |
| **Mid-Level Engineer** | The vocabulary and reasoning to participate in architecture discussions with confidence |
| **Senior Engineer** | A structured way to articulate decisions you already make intuitively |
| **Engineering Manager** | The technical depth to evaluate design proposals and ask the right questions |
| **Interview Candidate** | The thinking patterns that differentiate a good answer from a great one |

---

## What Makes an Architect Different?

A developer asks: **"How do I build this?"**  
An architect asks: **"Should we build this? And if so, how will it fail?"**

The shift from developer to architect is not about years of experience. It is about changing the questions you ask before you write a single line of code.

This series teaches you to ask those questions.

---

## The Series

### Part 1 — Foundations of Architectural Thinking

The 10 mental models every developer must internalize before calling themselves an architect.

| # | Lesson | Core Idea |
|---|--------|-----------|
| 01 | Stop solving the problem — start understanding it | The wrong solution to the right problem costs more than no solution |
| 02 | Every technical decision is a business decision | If you can't explain it to a non-engineer, you haven't thought it through |
| 03 | Design for failure first, features second | The happy path is easy. The crash path is the real design |
| 04 | More is not better — more is the problem | More connections, more services, more complexity = diminishing returns |
| 05 | Consistency is a spectrum, not a switch | Strong, eventual, causal, read-your-writes — pick the right level |
| 06 | The boring choice is usually the right choice | Battle-tested tech that runs at 3 AM beats shiny tech that needs a PhD to debug |
| 07 | Draw the failure boundary before the system | Blast radius determines your SLA, your on-call load, your recovery strategy |
| 08 | Observability is not optional — it is the design | A system you cannot observe is a system you cannot operate |
| 09 | The best architecture is one your team can evolve | Complexity is a debt that arrives with interest |
| 10 | Trade-offs are the job. There are no free choices | Document every trade-off for the person who inherits your system |

---

### Part 2 — Real Problems. Real Architectures.

Each problem in this section is drawn from real-world incidents, postmortems, and design challenges at companies like Netflix, Google, Amazon, Uber, and Meta.

Every problem is presented as an architect would encounter it — with a story, a wrong approach, a correct approach, and the reasoning that connects them.

| # | Problem | Core Concept | Link |
|---|---------|--------------|------|
| P01 | Designing Google Docs — The Complete System Design | HLD, scale estimation, OT engine, sharding, fault tolerance, trade-offs | [Read](./P01_Google_Docs_System_Design.md) |
| P01b | Operational Transformation Deep Dive | OT algorithm, transformation rules, convergence proof, undo | [Read](./P01_Operational_Transformation_Google_Docs.md) |
| P02 | Netflix Chaos Monkey | Chaos engineering, resilience testing, steady-state hypothesis | _coming soon_ |
| P03 | Amazon Black Friday Connection Pool Meltdown | HikariCP, `(cores×2)+1`, PgBouncer, load testing | [Read](./P03_Amazon_Black_Friday_Connection_Pool_Meltdown.md) |
| P04 | The Kafka OOM Crash That Charged 1000 Customers Twice | At-least-once delivery, idempotency, offset commits | _coming soon_ |
| P05 | The Idempotency Key That Lied | Two-phase PENDING/COMPLETED, Redis SETNX flaw, partial failures | _coming soon_ |
| P06 | One Request. A Thousand Logs. Zero Answers. | Distributed tracing, TraceId/SpanId, OpenTelemetry, Jaeger | _coming soon_ |
| P07 | The Flash Sale That Took Down the Database | Rate limiting, queue-based load levelling, inventory reservation | _coming soon_ |
| P08 | The Notification Storm | Fan-out on write vs read, priority queues, delivery guarantees | _coming soon_ |
| _more coming_ | | | |

---

## How to Use This Series

**If you are starting out:**  
Read Part 1 first. Do not skip it. The mental models are the foundation. Everything in Part 2 is an application of those models.

**If you are preparing for interviews:**  
Every problem in Part 2 is a system design interview question. Read the problem, close the page, and try to design it yourself before reading the solution.

**If you are a senior engineer:**  
Use this as a reference and as a teaching tool. The best way to solidify architectural thinking is to explain it to someone else.

**If you are an engineering manager:**  
Focus on the trade-off sections. The goal is not to design systems yourself but to know which questions to ask when your team presents a design to you.

---

## What You Will Learn

- How to frame a problem before proposing a solution
- How to reason about failure modes before writing code
- How to choose between consistency models, storage engines, and messaging patterns
- How to make trade-offs explicit and defensible
- How to design systems that are observable, operable, and evolvable
- How to approach any system design interview question with a structured, architect-level response

---

## What This Series Is Not

- It is **not** a collection of algorithms or LeetCode problems
- It is **not** a list of design patterns to memorize
- It is **not** a "just use Kafka for everything" guide
- It is **not** beginner tutorials on how individual technologies work

It is about **how to think** — and that applies regardless of which technology stack you use.

---

## Structure of Each Post

Every problem or lesson follows the same structure:

```
1. The Story       — A real-world scenario that makes the problem concrete
2. The Mistake     — What a junior engineer gets wrong and why
3. The Diagnosis   — The root cause and why it matters
4. The Fix         — The architectural solution with code and diagrams
5. The Lesson      — The mental model to carry forward
6. Interview Q&A   — How to discuss this in a system design interview
```

---

## Contributing

Found a problem, a postmortem, or an architectural decision worth adding to the series?

Open an issue or a pull request. The best learning comes from real stories — and the best stories come from people who lived through the incident.

---

## Author

**Sunchit Dudeja**  
Engineering Leader · System Design Educator  
Building this series to give every developer the foundation that took most architects years of incidents to develop.

📺 [YouTube — CodeWithSunchitDudeja](https://www.youtube.com/@CodeWithSunchitDudeja)  
📸 [Instagram — @sunchitdudeja](https://www.instagram.com/sunchitdudeja/)

---

> *"A developer makes the code work. An architect makes the system last."*
