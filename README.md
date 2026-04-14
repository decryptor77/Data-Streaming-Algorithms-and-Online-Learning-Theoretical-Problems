# Data-Streaming-Algorithms-and-Online-Learning-Theoretical-Problems

A public collection of theoretical solutions in **Data Streaming Algorithms & Online Learning**, with emphasis on sketching, approximation, sliding-window models, and rigorous asymptotic analysis.<br>
Completed during my MSc in Data Science & Machine Learning at Reichman University, in the frame of a Data Streaming Algorithms & Online Learning course.

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
- Exponential Histograms in the Sliding Window Model
- Estimating Hamming Distance Between Two Streams Without Storing Them
- Additional proof-based questions in data streaming algorithms

---

## Included solutions

### Exponential Histograms in the Sliding Window Model

#### General problem

A central problem in streaming algorithms is maintaining statistics over the **last $W$ elements** of a stream, rather than over the entire prefix. This is harder than the standard prefix-stream setting because older elements must effectively expire, while the algorithm still operates in one pass and with small memory.

This solution considers a binary stream $a_1, a_2, \dots$ and studies how to estimate the number of 1s in the last $W$ elements under an approximation parameter $\varepsilon$, using the Exponential Histograms method. The uploaded solution explicitly defines the binary-stream setting, the bucket sizes $2^i$, the timestamp-based representation of buckets, and the sliding-window membership condition.

#### What the solution addresses

The write-up solves the problem through four parts:

**1. Sketch construction**  
The solution defines the Exponential Histogram sketch by grouping 1s into buckets of exponentially increasing sizes:
$1, 2, 4, 8, \dots$

For each bucket, the sketch stores:
- the bucket size
- the timestamp of the most recent $1$ in that bucket

Buckets are stored from newest to oldest, and the number of buckets per size is bounded by $O(1/\varepsilon)$. The solution also discusses how timestamps can be represented relative to the window size, so that timestamp storage depends on $W$ rather than on unbounded absolute time.

**2. Update procedure**  
Upon the arrival of a new bit, the algorithm first removes expired buckets that no longer belong to the last-$W$ window. If the new bit is $0$, no new bucket is created. If the new bit is $1$, a new size-$1$ bucket is inserted. If too many buckets of the same size appear, the two oldest are merged into a bucket of the next size, and this merge process may continue upward in carry-like fashion. This update logic is described explicitly in the solution.

**3. Estimation and error analysis**  
To estimate the number of $1$s in the window, the solution sums all buckets except the oldest boundary bucket exactly, and takes a fraction $\alpha$ of the oldest bucket. It then analyzes the resulting error and argues that the minimax-optimal deterministic choice is:
$\alpha = \frac{1}{2}$

Using this choice, the write-up proves that the additive error is at most $\varepsilon W$ by showing that the oldest bucket has size at most $2\varepsilon W$, and therefore counting half of it contributes error at most $\varepsilon W$.

**4. Memory complexity**  
The solution shows that the number of bucket sizes is at most $\lfloor \log_2 W \rfloor + 1$, and with at most $O(1/\varepsilon)$ buckets per size, the total number of buckets is:
$O\left(\frac{1}{\varepsilon}\log W\right)$

It then refines this into a bit-level bound by accounting for both bucket size representation and timestamp representation, obtaining:
$O\left(\frac{1}{\varepsilon}(\log W)^2\right)$ bits overall.

#### Solution approach in one paragraph

The core idea is to use **exponential bucket sizes** together with controlled merging to compress the recent history of the stream. This allows the algorithm to preserve enough information to estimate the number of recent $1$s, while keeping memory sublinear in the window size. The approximation error is concentrated entirely in the oldest boundary bucket, and the choice $\alpha=\frac{1}{2}$ balances the worst-case uncertainty in that bucket. The resulting sketch achieves efficient updates, compact storage, and a provable additive error guarantee.

#### Main guarantees

The solution derives the following key guarantees:
- additive error at most $\varepsilon W$
- $O\left(\frac{1}{\varepsilon}\log W\right)$ buckets
- $O\left(\frac{1}{\varepsilon}(\log W)^2\right)$ bits of memory

---


### Estimating Hamming Distance Between Two Streams Without Storing Them

#### General problem

A basic question in streaming algorithms is how to compare two synchronized streams without storing them explicitly. In this problem, we receive two binary streams
$A = a_1, a_2, \dots, a_n$ and $B = b_1, b_2, \dots, b_n$,
where at each time step $t$ we observe the pair $(a_t, b_t)$ only once and cannot return to past updates.

The goal is to estimate the **Hamming distance**
$H$ between the two streams, defined as the number of indices $t$ such that $a_t \ne b_t$.

The challenge is to do this in the streaming model:
- with only a small sketch
- with one-pass updates
- with provable approximation guarantees
- without storing the full streams

This is a natural sketching problem, because the exact answer depends on all positions, yet the streaming setting forbids full storage.

#### What the solution addresses

The solution shows that the problem can be reduced to estimating the second frequency moment $F_2$, and therefore can be solved using an **AMS sketch**.

**1. Reduction to an $F_2$ problem**  
Define a derived stream
$x_t = a_t - b_t$.

Since $a_t, b_t \in \{0,1\}$, each value satisfies
$x_t \in \{-1,0,1\}$.

Now:
- if $a_t = b_t$, then $x_t = 0$
- if $a_t \ne b_t$, then $x_t = \pm 1$

Therefore,
$H = \sum_{t=1}^n x_t^2$.

So the Hamming distance is exactly the second moment of the derived stream, which makes the problem an instance of $F_2$ estimation.

**2. Sketch construction and update rule**  
The solution uses $k$ independent random sign hash functions and maintains $k$ AMS counters.
For each arriving pair $(a_t,b_t)$:
- compute $x_t = a_t - b_t$
- update every counter by adding the signed contribution of $x_t$

At the end, the estimator is the average of the squared counters:
$\hat H = \frac{1}{k}\sum_{i=1}^k Z_i^2$.

This gives a one-pass streaming algorithm with a small sketch and constant-time logic per counter update.

**3. Accuracy guarantee**  
Using the standard AMS fact that
$\mathrm{Var}(Z^2) \le 2F_2^2$,
and here $F_2 = H$, the solution shows
$\mathrm{Var}(\hat H) \le \frac{2H^2}{k}$.

Then, by Chebyshev’s inequality,
$\Pr(|\hat H - H| \ge \varepsilon H) \le \frac{2}{k\varepsilon^2}$.

Choosing
$k = \frac{20}{\varepsilon^2}$
yields
$\Pr(|\hat H - H| \le \varepsilon H) \ge 0.9$.

Thus the sketch achieves relative error $O(\varepsilon)$ with constant success probability.

**4. Memory complexity**  
Each counter stores a value whose magnitude is at most $n$, so each counter requires $O(\log n)$ bits.
With $k = O(\varepsilon^{-2})$ counters, the counter storage is
$O(\varepsilon^{-2}\log n)$ bits.

The random hash functions can also be stored with $O(\log n)$ bits each using standard bounded-independence constructions, so the total sketch size remains
$O(\varepsilon^{-2}\log n)$ bits.

#### Solution approach in one paragraph

The key idea is to transform the comparison problem into a moment-estimation problem. Instead of trying to remember where the two streams differ, the solution defines a derived stream whose entries are $-1$, $0$, or $1$ depending on the difference between the two current bits. The Hamming distance then becomes exactly the sum of squared derived values, namely an $F_2$ quantity. Once written in this form, the problem fits the AMS framework directly: maintain random signed linear projections of the stream, square them, and average. This yields a compact sketch with a clean proof of unbiasedness, variance control, and relative-error guarantees.

#### Main guarantees

The solution derives the following guarantees:
- one-pass streaming algorithm
- sketch update based on AMS counters
- relative error at most $\varepsilon H$ with probability at least $0.9$
- $O(\varepsilon^{-2}\log n)$ bits of memory

## Notes

This repository is intended as a clean public presentation of theoretical data streaming algorithms work. The emphasis is on:
- formal reasoning
- sketch design
- approximation guarantees
- asymptotic analysis

---

## Author

**Noor Nashef**  
MSc Data Science & Machine Learning student, Reichman University<br>
BSc in Information Systems Engineering specialized in Machine Learning, Technion
