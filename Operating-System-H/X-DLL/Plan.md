# 计划安排
### 说明
成员：董恒(DH)、李楠(LN)、连家诚(LJC)

### 分部
不同的DFS有不同的特性，比如HDFS适合存储大文件，TFS适合小文件，但是在小文佳存取方面都有问题，我们的核心在于使用RNN预测，因而我们希望我们的策略能够适应不同的文件系统情景，而不单单适用于某个DFS（比如我们初期希望仅仅对HDFS进行优化）。

为了达到这个目的，我们使用2个分部策略。
- 阶段分部
  - 本地模拟
  - DFS适用
- 模块分部
  - DFS模块
  - 预测模块

其中模块分部是在DFS适用阶段使用的。

鉴于我们目前对LSTM能够对小文件预测的效果没有很高的期待，所以可能会花比较长的时间去优化，这种情况下，第二阶段的完成度可能不太理想。

### 具体安排以及执行情况
###### week 9
- DH：构建测试数据
- LN：了解DFS相应API以及搭建集群
- LJC：做中期报告

###### week 12
- DH: 学习搭建基本的LSTM
  - 在LSTM目录记录相关工作，命名为Simple_LSTM.md
  - 【完成情况】完成
- LN: 探索DFS(不限于HDFS)添加模块的方式
  - 使用markdown，在DFS目录记录相关工作，命名为Add_Module.md
  - 【完成情况】进行中
- LJC: 寻找是否存在可用的DFS(不限于HDFS),而不必自行搭建集群。　
  - 如果找到，请研究
    - 相关使用方式
    - 添加模块方式
  - 如果没找到，请结合LN 在week９的工作，整理DFS搭建方式与流程
  - 使用markdown，在DFS目录记录相关工作,命名为Other_DFS.md
  - 【完成情况】未知

###### week 13
- DH: 将预测模块封装成函数
  - 在LSTM目录记录相关工作。
  - 使用LSTM, GRU, Bidirectional
  - 【完成情况】进行中
- LN:顺延上周
  - /DFS/Add_Module.md
  - 【完成情况】完成
- LJC: 顺延上周
  - /DFS/JAVA_API.md
  - 【完成情况】完成

###### week 14
- DH:　
  - 继续完成上周任务
  - 寻找学术数据集
  - 【完成情况】完成上周任务，未找到可用数据集，具体参见[README](./LSTM/README.md)
- LN:练习使用java/python 混合编码
  - 【完成情况】未知
- LJC: 和LN合作，具体工作和LN商定，并且补充在此处
  - 【完成情况】未知

###### week 15 6/3-6/9
** 注1 ** 为了不严重影响后续考试，需要在本周完成代码任务，下周完成报告书写

** 注2 ** 为了催促本周繁重工作的顺利进行，请每日自行在下方记录每日所做。请先git pull再git push
- DH:
  - 使用并且改进数据集
  - 完成模型构建与改进
  - 完成本地测试
  - 请用markdown 详细记录所做，所做记录在/LSTM
  - 【完成情况】
    - 6/3:　完成数据集生成,[代码](./LSTM/generate_data.py)
    - 6/4: ignore(name created modified access),完成对其他部分的模型构建以及测试
    - 6/5: 对directory预测改进
    - 6/5: 由论文《Design and Implememtation of a Predictive File Prefetching Algorithm》　获得启发，将使用strace构建application-based　benchmark
    - 6/7: 无
    - 6/8: 无
    - 6/9: 解析strace 输出
  - 【最终完成情况】完成
- LN:
  - 调用预测模块完成DFS模块的添加
  - 完成测试
  - 请用markdown 详细记录所做，所做记录在/DFS
  - 【完成情况】
    - 6/3:  对于LSTM目录下的test.py，给出了用shell脚本调用的方式
    - 6/4:
    - 6/5: 这几天忙于拆炸弹，过几天再给出进展。。。
    - 6/5:
    - 6/7:
    - 6/8:
    - 6/9:
  - 【最终完成情况】未知

- LJC:
  - 和LN合作，具体工作和LN商定，并且补充在此处
  - 请用markdown 详细记录所做
  - 【完成情况】
    - 6/3:
    - 6/4:
    - 6/5:
    - 6/5:
    - 6/7:
    - 6/8:
    - 6/9:
  - 【最终完成情况】未知

###### week 16 6/10-6/16
总结报告的书写，使用markdown,记录在/Report4/目录
- DH:
  - 写预测模块的报告
  - 写在/Report4/LSTM.md
  - 图片保存在/Report4/LSTM_img
  - 【完成情况】完成
- LN:
  - 写模块添加的相关报告
  - 写在/Report4/DFS.md
  - 图片保存在/Report4/DFS_img
- LJC:
  - 写模块添加的相关报告
  - 和LN合作，具体工作和LN商定，并且补充在此处

16周开始进入考试周，力求在这两周内完成全部任务。