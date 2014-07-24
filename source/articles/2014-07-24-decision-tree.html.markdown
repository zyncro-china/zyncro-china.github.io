---
title: Decision Tree
date: 2014-07-24 02:26 UTC
tags:
---

## 数据分析服务: BigML
今天我看到一个数据分析的服务，可以上传CSV的数据，分析后可以预测新的结果。

它有几个步骤

1. 上传CSV数据,并设定每行的数据类型，如数字，字符。数据最后一列必须是结果列
1. 建立数据集，这个过程需要几秒或几小时。需要选定需要分析的属性，最终会给出一个属性列表，显示属性的一些统计数据：缺失数据，错误和直方图
1. 建立模型，通过数据集生成模型
1. 为新输入预测结果
1. 建立ensembles，可以用所有的输入行和列来生产多个model，合成一个ensemble,用户可以这样比较哪种组合比较合理
1. 评估模型或ensembles

## 背后的机器学习算法 decision tree
decision tree是一种很直观的机器学习算法。给定一些training data,
里面包含了一些属性，算法的目的是学习隐藏在数据里的决策函数(decision
function),并可以用它来分类新的输入。
decision
tree算法不想用到所有的属性，而是想去掉那些弱因素，只保留强因素，并把强因素放在上方。

![Decision Tree Example](/images/xdiscrete-tree.png.pagespeed.ic.png)

每个属性的强弱可以用information gain表示，可以用公式计算。

![Information Gain](/images/xinformation-gain-formula.png.pagespeed.ic.png)

## Ruby gem for decision tree: decisiontree

```ruby
require 'rubygems'
require 'decisiontree'

attributes = ['Age', 'Education', 'Income', 'Marital Status']
training = [
  ['36-55', 'Masters', 'High', 'Single', 1],
  ['18-35', 'High School', 'Low', 'Single', 0],
  ['< 18', 'High School', 'Low', 'Married', 1]
  # ... more training examples
]

# Instantiate the tree, and train it based on the data (set default to '1')
dec_tree = DecisionTree::ID3Tree.new(attributes, training, 1, :discrete)
dec_tree.train

test = ['< 18', 'High School', 'Low', 'Single', 0]

decision = dec_tree.predict(test)
puts "Predicted: #{decision} ... True decision: #{test.last}";

# Graph the tree, save to 'discrete.png'
dec_tree.graph("discrete")

```




