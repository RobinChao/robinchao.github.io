---
layout: post
author: Robin
title: 8大场景数据清洗Python代码
tags: 机器学习 Python
categories:
- 机器学习
---

**数据清洗**是进行数据分析和使用数据训练模型的必经之路，也是最为耗费数据科学家、程序员的地方。

在数据清洗的过程中，绝大多数的场景下，所进行的清洗工作都是相似甚至是重复的，因此有必要将数据清洗工作的场景进行总结并给出对应的清洗代码，以便形成可适用于多数工程项目的工具箱。

___

# 涵盖8大场景的数据清洗代码

以下数据清洗代码，涵盖了8个数据清洗工作中常见的场景，分别是：

1. 删除多列
2. 转换数据类型
3. 将分类变量转换为数字变量
4. 检查缺失数据
5. 删除列中的字符串
6. 删除列中的空格
7. 用字符串连接两列（带条件）
8. 转换时间戳（从字符串到日期时间格式）

## 1. 删除多列

在进行数据分析时，可能并非所有的列都有用，此时可以使用`df.drop`方便地删除指定的列：

```python
def drop_multiple_col(col_name_list, df):
	df.drop(col_name_list, axis=1, inplace=True)
	return df
```

## 2. 转换数据类型

当数据集变大时，可能需要转换数据类型来节省内存空间：

```python
def change_dtypes(col_int, col_float, df):
	df[col_int] = df[col_int].astype('int32')
	df[col_float] = df[col_float].astype('float32')
```

## 3. 将分类变量转换为数字变量

在一些机器学习模型中，会要求变量采用数值格式。此时便需要将分类变量转换为数字变量，同时，也可以保留分类变量，以便进行数据可视化等：

```python
def convert_cat_2num(df):
	num_encode = {'col_1' : {'YES':1, 'NO':0},
				  'col_2' : {'WON':1, 'LOSE':0, 'DRAW':0}}
	df.replace(num_encode, inplace=True)
```

## 4. 检查缺失数据

如果要检查每列缺失数据的数量，可使用下面的代码，目前来看应该是最快的方法。可以更好地了解哪些列缺失的数据更多，从而确定怎么进行下一步的数据清洗和分析操作：

```python
def check_missing_data(df):
	return df.isnull().sum().sort_values(ascending=False)
```

## 5. 删除列中的字符串

有时，会有新的字符或者其他不需要的符号出现在字符串中，此时可以使用`df['col_1'].replace`将它们处理掉：

```python
def remove_col_str(df):
	df['col_1'].replace('\n', '', regex=True, inplace=True)
	df['col_1'].replace(' &#.*', '', regex=True, inplace=True)
```

## 6. 删除列中的空格

当数据混乱的时候，什么情况都有可能发生。字符串开头有时会有一些空格，在删除列中字符串开头的空格时，可使用下面的代码：

```python
def remove_col_white_space(col, df):
	df[col] = df[col].str.lstrip()
```

## 7. 用字符串连接两列（带条件）

当你想要有条件地用字符串将两列连接在一起时，这段代码很有帮助。比如，你可以在第一列结尾处设定某些字母，然后用它们与第二列连接在一起。

根据需要，结尾处的字母也可以在连接完成后删除。

```python
def concat_col_str_condition(df):
    mask = df['col_1'].str.endswith('pil', na=False)
    col_new = df[mask]['col_1'] + df[mask]['col_2']
    col_new.replace('pil', ' ', regex=True, inplace=True)  # replace the 'pil' with emtpy space
```

## 8. 转换时间戳（从字符串到日期时间格式）

在处理时间序列数据时，我们很可能会遇到字符串格式的时间戳列。

这意味着要将字符串格式转换为日期时间格式(或者其他根据我们的需求指定的格式) ，以便对数据进行有意义的分析。

```python
def convert_str_datetime(df): 
    df.insert(loc=2, column='timestamp', value=pd.to_datetime(df.transdate, format='%Y-%m-%d %H:%M:%S.%f')) 
```

以上便是针对8个常见场景的数据清洗代码，在部分场景下，你可能需要简单修改代码才可使用。在面对各种不同且复杂的数据时，可能需要先了解你的数据，然后再决定使用那个或者那些方法进行数据的清洗工作，使得数据能够更好的进入下一步的分析建模过程等。
