分解方法

## 1. Principal Component Decomposition (PCD)

Although the PCD algorithm can calculate global solutions for QUBOs of arbitrary size and structure,  it was developed particularly for application to problem graphs with time structure.

专用于具有时间结构的问题分解

### Spring Layout algorithm

### Principal Component Analysis (PCA)

---



##  2. Freeze-and-Anneal (FA)

 the FA algorithm presented in this work has been generalized and is capable of generating solutions for arbitrary QUBO problems.

可以用于分解任意QUBO问题





---



## 3. Qbsolv

created for the purpose of solving QUBO problems too large to natively fit on quantum annealing hardware. The Qbsolv algorithm leverages a quantum annealer’s ability to heuristically search large solution spaces, while also accounting for the annealer’s lack of precision through the use of a more precise, local minimization algorithm

该算法专门用于解决量子退火硬件能力不足的问题。同时在局部搜索过程中还使用了其它的经典算法如禁忌算法Tabu Search来解决退火器精度不足的问题。



---



## 4. Iterative Centrality Halo (ICH)

 is applied to decomposing the problem graph of a QUBO representation of the Maximum Clique problem







---



## 5. Random Solver

