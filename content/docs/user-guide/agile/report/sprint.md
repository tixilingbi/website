+++
title = "冲刺报告"
description = "了解每个冲刺中完成、进行和退回待办的工作。这有助于您确定您团队的工作量是否超额，更直观的查看冲刺的范围与工作量。"
weight = 1
+++

# 冲刺报告

冲刺报告会显示每个冲刺中完成、进行和退回待办的工作。这有助于您确定您团队的工作量是否超额，更直观的查看冲刺的范围与工作量。

## 操作步骤
* 选择报告
    1. 点击`报告`
    2. 选择`冲刺报告`
* 切换报告
    1. 点击`切换报表`按钮
    2. 选择`燃尽图`或者`累计流量图`等其他报表

## 迭代冲刺
* `迭代冲刺`：当前项目所有的冲刺，包括已完成和未完成的冲刺

## 图表
* 横坐标代表时间
* 纵坐标代表问题数量

![](/docs/user-guide/agile/report/img/sprint-chart.jpg)

## 报告详情
1. 报告详情总共有三个模块，分别为`已完成的问题`，`未完成的问题`，`从Sprint中删除的问题`
    - `已完成的问题`：指查询冲刺报告中已完成的issue列表，根据冲刺开启、关闭时间，查找该时间段最后一次issue状态变更后状态为`设置为已完成`的issue，但子issue除外
        {{< note >}}如果状态被删除为空，则继续向上查找，如果在开启、关闭时间段issue变更过的状态都为空则根据issue当前的状态进行判断{{< /note >}}
    - `未完成的问题`：指查询冲刺报告中未完成issue列表，根据冲刺开启关闭时间，查找该时间段最后一次issue状态变更后状态不为`设置为已完成`的issue，但子issue除外
        {{< note >}}如果状态被删除为空，则继续向上查找，如果在开启、关闭时间段issue变更过的状态都为空则根据issue当前的状态进行判断{{< /note >}}
    - `从Sprint中删除的问题`：指冲刺开启前存在而现在不存在的issue，但子issue除外

2. 字段描述
    - `关键字`：表示问题编号
    - `概要`：表示问题概要
    - `问题类型`：表示问题类型，包括史诗、故事、任务、缺陷
    - `优先级`：表示问题优先级，包括高、中、低
    - `状态`：表示问题关联的状态
    - `故事点`：如果问题类型是故事，将显示该问题的故事点。其他类型不显示该字段
    
{{< note >}}
* 冲刺报告中开启之后加入的issue：指的是冲刺中现在存在而冲刺开启前不存在的issue，子issue除外
* 新增的issue在issue编号右边标注*号表示
{{< /note >}}
![](/docs/user-guide/agile/report/img/sprint-detail.jpg)


## 其他报表

- [燃尽图](../burn-down)
- [累积流量图](../cumulative-flow)
- [版本报告](../version-report)