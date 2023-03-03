
## [量子计算机的拓扑结构](https://docs.dwavesys.com/docs/latest/c_gs_4.html#pegasus-graph)
1. Chimera Graph: D-Wave 2000Q QPUs
2. Pegasus Graph: Advantage QPUs
3. Zephyr Graph: for next-generation QPUs currently under development

## [量子计算机的使用流程](https://docs.dwavesys.com/docs/latest/c_gs_workflow.html#getting-started-workflow)
1. Formulate your problem as an objective function
> for quantum computing, these are quadratic models that have lowest values (energy) for good solutions to the problems they represent.
>
> 因为quadratic中的一次项代表一个独立的量子比特，二次项代表两个量子比特之间的耦合，这种形式可以很好的用量子计算机去描述，
> 而常数项可以直接省去并不会改变变量之间的约束情况，不会影响最优解的求解
> 因此要把问题建模称quadratic的形式。
>
> Constraints for such models are typically represented by adding penalty models to the objective, as shown in the Constraints Example: Problem Formulation section.
> 最终把问题建模成QUBO(Quadratic Unconstrained Binary Optimization problem)的形式。
2. Find good solutions by sampling
> 采样的概念：
>
> some classical samplers actually brute-force solve small problems rather than sample, and these are also referred to as solvers
> 经典算法在小规模问题上采用暴力搜索的方法，而不是采样。
>
> 总共三种采样方法： D-Wave’s quantum, classical, and [hybrid quantum-classical samplers](#quantum-classic-hybrid-solver).

## [Embedding：把QUBO模型映射到量子比特QPU上](https://docs.dwavesys.com/docs/latest/c_gs_7.html#getting-started-embedding)
量子计算中的拓扑图理论上应该是个完全图，即每个量子比特都与其它所有的量子比特耦合。
但由于技术上的难度，只能实现部分耦合，如Chimera Graph结构，量子计算机的实际运算能力达不到理论水平
因此不能将QUBO问题直接映射到量子计算机上。
为了更契合量子计算机的实际拓扑结构，需要Embedding进行嵌入映射。

嵌入方法：
把一个变量用多个具有耦合关系的QPU-chain表示。如图所示，左边是待求解的QUBO问题，右边是量子计算机的拓扑结构。该图中的量子比特没有三角形的耦合结构，无法直接把变量与QPU一一映射，可以选择把其中一个变量映射到两个QPU上，并给原问题加上约束：一个chain上的QPU具有相同的值。


![Embedding](../../images/Quantum%20Annealing/Embedding.png)

图中b的系数是-1，用两个比特表示，则每个比特的偏置值bias应该是-0.5.由于引入了一个原问题中不存在的耦合，相当于在原问题上增加了一项$b^2$，为0与5之间的耦合值取任意值d=-3，则应该再补偿0号与5号的bias：$-0.5-\frac{d}{2}=1$，则Embedding后四个量子比特的取值为：
![](../../images/Quantum%20Annealing/Embed%20Reslut.png)

对于不同结构的量子计算机Embedding方式也不同。

## Unembedded
对采样得到的一系列解num_reads中，计算最终得到的问题的energy，将energy最低的解保留，其余的都舍弃。



## Quantum-Classic Hybrid Solver
quantum solver 和 classical solver都只能求解小规模问题，但是hybrid solver可以通过将大规模问题[分解](https://docs.dwavesys.com/docs/latest/handbook_decomposing.html#cb-decomposing)成小规模问题来求解大规模问题
> a hybrid workflow might break a large problem into smaller pieces that can be solved on a QPU and then recombined.


### 分解方法
select a subproblem of variables maximally contributing to the problem energy

1. 选择对问题的能量贡献最大的子部分(variables selected by highest impact on energy)进行分解，然后并行计算子问题。

如何找到对问题能量贡献最大的部分？

使用breadth-first (BFS) or priority-first selection (PFS) 遍历量子比特节点
```python
import dimod
import hybrid

# Construct a problem
bqm = dimod.BinaryQuadraticModel({}, {'ab': 1, 'bc': -1, 'ca': 1}, 0, dimod.SPIN)

# Define the workflow
# 采用decomposer | sampler | composer的形式
# decomposer(size): 要分解得到的问题规模大小。比如一个包含10个变量的大问题分解成包含6个变量的子问题，则size=6
# decomposer(rolling_history): 不同子问题之间允许重叠的变量的最大比例
# sampler(num_reads): 采样的次数
# ArgMin: selects a new current sample
# you can use the Loop to iterate a set number of times or until a convergence criteria is met. （可以使用 Loop 迭代一定次数或直到满足收敛条件.）
iteration = hybrid.RacingBranches(
    hybrid.InterruptableTabuSampler(),
    hybrid.EnergyImpactDecomposer(size=2, rolling_history=0.15, traversal='bfs')   
    | hybrid.QPUSubproblemAutoEmbeddingSampler(num_reads=2)
    | hybrid.SplatComposer()
) | hybrid.ArgMin()
workflow = hybrid.LoopUntilNoImprovement(iteration, convergence=3)

# Solve the problem
init_state = hybrid.State.from_problem(bqm)
final_state = workflow.run(init_state).result()

# Print results
print("Solution: sample={.samples.first}".format(final_state))

```

具体API调用参考[Using the Framework](https://docs.ocean.dwavesys.com/en/stable/docs_hybrid/intro/using.html)

hybrid的集成框架：Kerberos
```python
import dimod
from hybrid.reference.kerberos import KerberosSampler
with open('../problems/random-chimera/8192.01.qubo') as problem:  
    bqm = dimod.BinaryQuadraticModel.from_coo(problem)
len(bqm)          
solution = KerberosSampler().sample(bqm, max_iter=10, convergence=3)   
solution.first.energy     
```
2. 分析问题的结构，根据problem’s structure.进行分解

