# Fast Search of the Optimal Contraction Sequence in Tensor Networks

## Introduction

### Applications

- quantum physics:
  - many body quantum physics
  - matrix product states
  - projected entangled states
  - multiscale entanglement renormalization
  - quantum circuit design
- IC Modeling
- EDA problems
- Neural Networks
- Signal Processing

### Problem


- finding the contraction sequence with optimal compute cost is NP-hard
- but: optimal storage cost of a contraction sequence equals the treewidth of its linegraph
  - computing this is also NP-hard
- hypothesis based on the above two: finding the contraction sequence with optimal storage cost is NP-hard
- this means we can only use heuristics to tackle the problem

### Prior Work

  - depth-first constructive search. breadth-first constructive search, dynamic programming
    - Problem: search space grows exponentially with network size
  - algorithms to shrink original search space
    - compute: cost capping and outer product constraints
    - storage: methods exist, but they don't necessarily lead to an optimal outcome
    - some specialized algorithms for closed tensor networks (one that contracts into a scalar)

### Goals

- find efficient data formats and data structures for the computation
- solution should: 
  - find optimal solution (in the chosen data format/structure!)
  - shrink search space
  - fit all tensor networks without further constraints
  - search time should be superior

### Contributions

1. Search algorithm based on adjacency matrix
2. efficient algorithm to find tensors that can be pruned from the search space (prunable tensors, see prior work)
3. Multithreading of their algorithms to further improve speed

## Preliminaries

- a tensor network with $V$ tensors requires $V-1$ contraction steps 
- compute cost: number of multiplications
- order of the tensor is denoted by it's logarithm (basis doesn't matter, as long as it stays the same). This helps later, because multiplications can be transformed to additions

$$FO_{\Tau_I}=\sum_m\log_kN^m_{\Tau_I}, \{N^m_{\Tau_I}\in \text{ free orders of } \Tau_I\}$$
$$SO_{\Tau_I\Tau_J}=\sum_m\log_kN^m_{\Tau_I\Tau_J}$$
$$S_{\Tau_I}=FO_{\Tau_I}+\sum_JSO_{\Tau_I\Tau_J}, \text{ where } J\notin I$$
$$SE_{\Tau_I\Tau_J}=S_{\Tau_I}+S_{\Tau_J}-SO_{\Tau_I\Tau_J}=S_{\Tau_{IJ}+SO_{\Tau_I\Tau_J}}$$

### Contraction sequence with lowest maximum expense

- Generally, one can always evaluate a contraction sequence by either total contraction expense (sum of all contraction expenses) or maximum contraction expense ($n\times$ the worst contraction)
- the total comes close to maximum when the length of each order is large
- argument?: maximum is closer to reality because the tensor contractions don't all fit on the chip and a lot of communication with off-chip resources is needed
- authors choose maximum contraction expense
  - more precisely: maximum storage expense (MS) and maximum compute expense (MC)
- goal: find contraction sequence with lowest MS or MC
$$MS= \max_tSE(sq_t), MC= \max_tCE(sq_t)$$
- what?: From the instances, it is easy
to observe that the contraction sequence with the lowest MS
(or MC) cannot guarantee that the sequence has the lowest
MC (or MS). Thus, the metric should be selected in advance
to evaluate the contraction cost in different scenarios.

### BFS Search Algorithm