
## 论文的基本信息

* 作者：Michael R. Zielewski && Hiroyuki Takizawa
* 发表时间：2022， January， 07
* 会议：HPC Asia2022: International Conference on High Performance Computing in Asia-Pacific Region Virtual Event Japan January 12 - 14, 2022
* 出版商：Association for Computing Machinery


## 研究对象
已知在量子退火过程中引入pause会提高找到解的效率，然而难以确定合适的暂停时间点 如果暂停点离合适的位置相差甚远反而会降低QA的求解效率、使TTS增加，且每次去求解最佳的暂停时间会重复许多工作，因此本文的研究对象是：探究到能快速求解合适暂停点的方法，如何用尽可能少的退火次数尽快确定合适的（带有pause的）退火计划。

支撑本文研究的论据：
1. 相似的优化问题的暂停时间点也相似
2. 规模较大的优化问题可以分解成多个相似的子问题，原始问题的暂停点可以用子问题的暂停点表示

## 研究方法
1. Anneal with schedules parameterized by various pause durations and locations。
用各种不同的退火计划（暂停起始位置和暂停时间各不相同）去执行量子退火
2. Aggregate results and evaluate schedules
汇总结果并评估计划（合理性）
3. Determine a class-wise optimal schedule
确定一类问题的最佳合理计划
4. Anneal with the estimated optimal schedule
用估计的最佳退火计划为实际问题进行量子退火，即在相似的问题上，以求出的最佳退火计划作为初始值，以此来更加快速地找到实际问题地最佳退火计划，而不需要每次都从0开始去探索。

## 使用的数据
生成100组可以分解为子问题（SSP）的数据，20%的数据用于确定最佳退火计划，剩下的80%数据用得到的最佳退火计划执行退化，并于不执行暂停的退火效率进行比较，用于评估方法的有效性。

## 主要结论
提出的方法能快速得出最佳退火计划

## limitations and Future Work
1. targets problems that can be decomposed into multiple similarly structured sub-problems
大问题如何分解成相似的小问题，肯定存在许多数据无法分解
2. 数据的局限性，在本文的数据中相似的问题用同一退火计划都能提高退火效率，但在其它数据上的表现结果未知。





