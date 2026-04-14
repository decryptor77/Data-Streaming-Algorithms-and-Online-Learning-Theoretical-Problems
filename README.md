# Data-Streaming-Algorithms-and-Online-Learning-Theoretical-Problems

A public collection of theoretical solutions in **Data Streaming Algorithms and Online Learning**, with emphasis on sketching, approximation, sliding-window models, and rigorous asymptotic analysis.

Streaming algorithms study how to process large or potentially unbounded data streams under strict resource constraints. Instead of storing the full input, the algorithm maintains a compact summary, or **sketch**, that supports approximate answers with provable guarantees. Typical goals in this area include:
- using sublinear memory
- supporting fast updates, ideally $O(1)$ or amortized $O(1)$
- proving explicit error bounds
- analyzing the tradeoff between accuracy, memory, and update cost

This repository is focused on the **theoretical side** of the field. The goal is not just to present final answers, but to present the underlying algorithmic design, proof ideas, and complexity arguments in a clean and readable way.

---

## Repository structure

Each solution in this repository is published as a standalone document with its own descriptive title.

Current and planned topics include:
- sliding-window counting with Exponential Histograms
- frequency-moment estimation with AMS sketches
- additional proof-based questions in streaming algorithms

---

## Included solution

### Exponential Histograms in the Sliding Window Model

#### General problem

A central problem in streaming algorithms is maintaining statistics over the **last $W$ elements** of a stream, rather than over the entire prefix. This is harder than the standard prefix-stream setting because older elements must effectively expire, while the algorithm still operates in one pass and with small memory.

This solution considers a binary stream $a_1, a_2, \dots$ and studies how to estimate the number of $1$s in the last $W$ elements under an approximation parameter $\varepsilon$, using the Exponential Histograms method. The uploaded solution explicitly defines the binary-stream setting, the bucket sizes $2^i$, the timestamp-based representation of buckets, and the sliding-window membership condition. :contentReference[oaicite:0]{index=0}

#### What the solution addresses

The write-up solves the problem through four parts:

**1. Sketch construction**  
The solution defines the Exponential Histogram sketch by grouping $1$s into buckets of exponentially increasing sizes:
$1, 2, 4, 8, \dots$

For each bucket, the sketch stores:
- the bucket size
- the timestamp of the most recent $1$ in that bucket

Buckets are stored from newest to oldest, and the number of buckets per size is bounded by $O(1/\varepsilon)$. The solution also discusses how timestamps can be represented relative to the window size, so that timestamp storage depends on $W$ rather than on unbounded absolute time. :contentReference[oaicite:1]{index=1}

**2. Update procedure**  
Upon the arrival of a new bit, the algorithm first removes expired buckets that no longer belong to the last-$W$ window. If the new bit is $0$, no new bucket is created. If the new bit is $1$, a new size-$1$ bucket is inserted. If too many buckets of the same size appear, the two oldest are merged into a bucket of the next size, and this merge process may continue upward in carry-like fashion. This update logic is described explicitly in the solution. :contentReference[oaicite:2]{index=2}

**3. Estimation and error analysis**  
To estimate the number of $1$s in the window, the solution sums all buckets except the oldest boundary bucket exactly, and takes a fraction $\alpha$ of the oldest bucket. It then analyzes the resulting error and argues that the minimax-optimal deterministic choice is:
$\alpha = \frac{1}{2}$

Using this choice, the write-up proves that the additive error is at most $\varepsilon W$ by showing that the oldest bucket has size at most $2\varepsilon W$, and therefore counting half of it contributes error at most $\varepsilon W$. :contentReference[oaicite:3]{index=3}

**4. Memory complexity**  
The solution shows that the number of bucket sizes is at most $\lfloor \log_2 W \rfloor + 1$, and with at most $O(1/\varepsilon)$ buckets per size, the total number of buckets is:
$O\left(\frac{1}{\varepsilon}\log W\right)$

It then refines this into a bit-level bound by accounting for both bucket size representation and timestamp representation, obtaining:
$O\left(\frac{1}{\varepsilon}(\log W)^2\right)$ bits overall. :contentReference[oaicite:4]{index=4}

#### Solution approach in one paragraph

The core idea is to use **exponential bucket sizes** together with controlled merging to compress the recent history of the stream. This allows the algorithm to preserve enough information to estimate the number of recent $1$s, while keeping memory sublinear in the window size. The approximation error is concentrated entirely in the oldest boundary bucket, and the choice $\alpha=\frac{1}{2}$ balances the worst-case uncertainty in that bucket. The resulting sketch achieves efficient updates, compact storage, and a provable additive error guarantee. :contentReference[oaicite:5]{index=5} :contentReference[oaicite:6]{index=6}

#### Main guarantees

The solution derives the following key guarantees:
- additive error at most $\varepsilon W$
- $O\left(\frac{1}{\varepsilon}\log W\right)$ buckets
- $O\left(\frac{1}{\varepsilon}(\log W)^2\right)$ bits of memory

---

## Notes

This repository is intended as a clean public presentation of theoretical streaming-algorithms work. The emphasis is on:
- formal reasoning
- sketch design
- approximation guarantees
- asymptotic analysis

Additional solutions can be added in the same format:
- title
- general problem
- what the solution addresses
- solution approach
- main guarantees
