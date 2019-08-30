---
layout: post
author: Robin
title: 如何分析iPhone设备中的计步数据 --- 非iOS开发者篇
tags: 开发知识 iOS
categories:
  - 开发知识 
  - 数据分析
--- 


### ∫∫∫ 开始

<img src="/assets/track-steps-iphone.jpg" width="80%" align="center">

移动互联网时代已经早早的来到了，目前的发展已经到几乎无所不能的时期，大多数情况下，人们对移动设备，特别是智能手机的依赖程度已经远远超越了桌面计算机设备了。

在智能设备发展的同时，为了能够让智能设备有更好的服务于人类的特性，各个厂商都在智能设备中或多或少的植入了一些传感器设备，这些传感器设备无时不收集着人类使用智能设备的数据，更进一步的有这些数据最终得到的结果服务于使用者，达到了最终数据的落地。

在本篇文章中，我将使用Python语言来尝试着分析iPhone收集设备中的计步器所累积的数据，使用到的是Python语言体系下的类库**[pandas](http://pandas.pydata.org/)**和图形类库**[ggplot](http://ggplot.yhathq.com/)**。

### ∫∫∫ 获取数据

第一步当然是收集数据。但是我又不希望在iPhone收集上去分析收集到的数据，因此就需要将iPhone设备上的相关计步数据导出。由于这篇文章是**非iOS开发者篇**，因此我这里会借助一款应用程序--- [ QS Access ](http://quantifiedself.com/access-app/app)，在这款应用中，你可以选择需要的数据，并最终导出csv格式的数据文件，非常有用！

下面是应用程序截图和导出数据表格的样例截图：

![](/assets/qs-access-snapshot.png)  

从上图可以看到，QS Access导出的CSV数据文件，包含三列数据：`start` 时间点；`finish`时间点；`steps (count)`该时间段内的步数。而且此软件支持按小时或者按天生成数据列表。

### ∫∫∫ 数据！

拿到了数据之后，我们需要一点一点的进行数据规整以达到最终我们需要的格式等等。在这里我主要使用的是[Wes McKinney](https://github.com/wesm)所发明的Python类库[pandas](http://pandas.pydata.org/)。如果还对此类库不太了解的，可以Google，这个类库在Python的世界里，是非常广泛使用的明星了。

在开始前先看看当前的形式，我们拿到的是CSV格式文件，因此需要能够读取CSV文件的方法，并且在拿到数据之后还需要对数据进行规则性的统计等等。而这些恰恰是[pandas](http://pandas.pydata.org/)比较擅长的。

在这里需要使用到的函数方法是`read_csv()`，它的具体定义如下：

 
 ![](/assets/read-csv.png)

 [查看官方文档](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.read_csv.html#pandas.read_csv)
 

接下来看是进行文件读取和解析：

第一：由于我们获取到的数据文件是时间序列性的，因此可以直接使用read_csv的参数`parse_dates`；

第二：结束时间对于我们来说，暂时并无太大的意义，因此先忽略结束时间列，这就使用到了`read_csv`方法的`usecols`参数，这个参数可以直接指定需要解析的列，而忽略不关心的列，提高解析速度和降低内存消耗；

第三：对于开始时间来说，可以直接设定为时间索引`DateTimeIndex`，这样能够使得接下来的工作简单点，不用再去定义新的索引变量。

开始编写代码之前，要清楚都需要使用到哪些类库？在本篇中用到的类库如下： 

{% highlight Python %} 
import pandas as pd
import numpy as np
import datetime
from ggplot import *
{% endhighlight %}

如果在导入这些类库之后，运行报错，可能是你的开发环境中没有安装这些类库导致的，因此你需要去对应的类库官网，按照安装指南进行安装和配置。

{% highlight Python %}
df_hour = pd.read_csv('health_data_hour.csv', parse_dates=[0,1], names=['start_time', 'steps'], usecols=[0, 2], skiprows=1, index_col=0)
# ensure the steps col are ints - weirdness going on with this one
df_hour.steps = df_hour.steps.apply(lambda x: int(float(x)))
df_hour.head()
type(df_hour.index)
type(df_hour.steps[1])
{% endhighlight%}
 

<table border="1" class="dataframe" style="width:auto;text-align:center">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>steps</th>
    </tr>
    <tr>
      <th>start_time</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-19 11:56:00</th>
      <td>24</td>
    </tr>
    <tr>
      <th>2016-06-19 14:26:00</th>
      <td>14</td>
    </tr>
    <tr>
      <th>2016-06-19 17:46:00</th>
      <td>9</td>
    </tr>
    <tr>
      <th>2016-06-19 18:32:00</th>
      <td>8</td>
    </tr>
    <tr>
      <th>2016-06-19 18:41:00</th>
      <td>8</td>
    </tr>
  </tbody>
</table>


从运行日志中，可以看到开始时间的数据类型为`pandas.tseries.index.DatetimeIndex`。这是因为在数据采集的过程中，给了我们一个更好的顺序以便于数据的使用。


### ∫∫∫ 每小时的步数

在初步的数据获取之后，我们就可以使用`ggplot`进行数据的第一步可视化了，首先来说原来的数据就是按照小时为单位的，因此要获取每小时的步数数据，只要直接开始时间和步数进行可视化即可。`ggplot`提供了很多视图样式，只需要进行简单的设置就可以。


{% highlight Python %}
p = ggplot(df_hour, aes(x = '__index__', y = 'steps')) + \
    geom_step() + \
    ggtitle("Hourly Step Count") + \
    xlab("Date") + \
    ylab("Steps")
print p
p.save("hourly_step_plot.png")
{% endhighlight%}

![](/assets/hourly_step_plot.png)

有了这些数据之后，我们看到了每小时用户步数的统计图表，在这些数据的基础上，如何进行`下采样`来提取更多的数据呢？我们需要获取更多的数据样本，这里使用到了`重采样`的方式，而重采样也是pandas所支持的。

### ∫∫∫ 转化为每日步数

{% highlight Python %}
## Daily
df_daily = pd.DataFrame()
df_daily['step_count'] = df_hour.steps.resample('D').sum()
df_daily.head()
p = ggplot(df_daily, aes(x='__index__', y='step_count')) + \
    geom_step() + \
    stat_smooth() + \
    scale_x_date(labels="%d/%m/%Y") + \
    ggtitle("Daily Step Count") + \
    xlab("Date") + \
    ylab("Steps")
print p
p.save("daily_step_plot.png")
{% endhighlight%}

![](/assets/daily_step_plot.png)

### ∫∫∫ 转化为每周、每月步数

到此，我们已经可以能够统计到每日的步数数据了，进一步需要得到每周、每月的步数，使用重采样将会非常简单，仅仅需要传入`resample('W')`或`resample('M')`即可。还有如果需要获取平均值或者总的值，仅仅需要调用`mean()`或者`sum()`函数即可。

{% highlight Python %}
## Weekly
df_weekly = pd.DataFrame()
df_weekly['step_count'] = df_daily.step_count.resample('W').sum()
df_weekly['step_mean'] = df_daily.step_count.resample('W').mean()
df_weekly.head()
p = ggplot(df_weekly, aes(x='__index__', y='step_count')) + \
    geom_step() + \
    stat_smooth(method='lm') + \
    scale_x_date(labels="%m/%Y") + \
    ggtitle("Weekly Step Count") + \
    xlab("Date") + \
    ylab("Steps")
print p
p.save("weekly_step_count_plot.png")
{% endhighlight%}


![](/assets/weekly_step_count_plot.png)

{% highlight Python %}
## Weekly Mean
df_weekly = pd.DataFrame()
df_weekly['step_mean'] = df_daily.step_count.resample('W').mean()
df_weekly.head()
p = ggplot(df_weekly, aes(x='__index__', y='step_mean')) + \
    geom_step() + \
    stat_smooth(method='lm') + \
    scale_x_date(labels="%m/%Y") + \
    ggtitle("Weekly Step Mean") + \
    xlab("Date") + \
    ylab("Steps")
print p
p.save("weekly_step_mean_plot.png")
{% endhighlight%}

![](/assets/weekly_step_mean_plot.png)

### ∫∫∫ 进一步分析

在这些数据的基础上，还能够获得什么样的数据？可以想象，某天的数据是否是周末的数据？这样可以分析是否在周末用户的活动量会更大等等？在开始之前，需要判断是否周末的函数来使用：

{% highlight Python %}
def weekendBool(day):
    if day not in ['Saturday', 'Sunday']:
        return False
    else:
        return True

df_daily['weekday'] = df_daily.index.weekday
df_daily['weekday_name'] = df_daily.index.weekday_name
df_daily['weekend'] = df_daily.weekday_name.apply(weekendBool)
df_daily.head()
{% endhighlight%}

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>step_count</th>
      <th>Weekday</th>
      <th>weekday</th>
      <th>weekdady_name</th>
      <th>weekday_name</th>
      <th>weekend</th>
    </tr>
    <tr>
      <th>start_time</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2016-06-19</th>
      <td>959</td>
      <td>6</td>
      <td>6</td>
      <td>Sunday</td>
      <td>Sunday</td>
      <td>True</td>
    </tr>
    <tr>
      <th>2016-06-20</th>
      <td>7947</td>
      <td>0</td>
      <td>0</td>
      <td>Monday</td>
      <td>Monday</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2016-06-21</th>
      <td>7451</td>
      <td>1</td>
      <td>1</td>
      <td>Tuesday</td>
      <td>Tuesday</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2016-06-22</th>
      <td>8477</td>
      <td>2</td>
      <td>2</td>
      <td>Wednesday</td>
      <td>Wednesday</td>
      <td>False</td>
    </tr>
    <tr>
      <th>2016-06-23</th>
      <td>7419</td>
      <td>3</td>
      <td>3</td>
      <td>Thursday</td>
      <td>Thursday</td>
      <td>False</td>
    </tr>
  </tbody>
</table>

得到这些数据之后，进一步可视化。

{% highlight Python %}
df_daily['weekday'] = df_daily.index.weekday
df_daily['weekday_name'] = df_daily.index.weekday_name
# apply the helper on the weekday_name col
df_daily['weekend'] = df_daily.weekday_name.apply(weekendBool)
df_daily.head()

weekend_grouped = df_daily.groupby('weekend')
weekend_grouped.describe()
weekend_grouped.median()

p = ggplot(aes(x='step_count', color='weekend'), data=df_daily) + \
    stat_density() + \
    ggtitle("Comparing Weekend vs. Weekday Daily Step Count") + \
    xlab("Step Count")
print p
p.save("weekday_step_compare_plot.png")
{% endhighlight%}

![](/assets/weekday_step_compare_plot.png)

<table border="0" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th></th>
      <th>step_count</th>
      <th>weekday</th>
    </tr>
    <tr>
      <th>weekend</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th rowspan="8" valign="top">False</th>
      <th>count</th>
      <td>70.000000</td>
      <td>70.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>9864.628571</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4039.553219</td>
      <td>1.424425</td>
    </tr>
    <tr>
      <th>min</th>
      <td>3740.000000</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>7432.250000</td>
      <td>1.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>8340.500000</td>
      <td>2.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>10998.000000</td>
      <td>3.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>26989.000000</td>
      <td>4.000000</td>
    </tr>
    <tr>
      <th rowspan="8" valign="top">True</th>
      <th>count</th>
      <td>28.000000</td>
      <td>28.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>7755.785714</td>
      <td>5.500000</td>
    </tr>
    <tr>
      <th>std</th>
      <td>4810.491130</td>
      <td>0.509175</td>
    </tr>
    <tr>
      <th>min</th>
      <td>271.000000</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>5458.750000</td>
      <td>5.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>7462.000000</td>
      <td>5.500000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>10139.250000</td>
      <td>6.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>18919.000000</td>
      <td>6.000000</td>
    </tr>
  </tbody>
</table>

### ∫∫∫ 结束

基本的数据统计分析以及可视化就是这样了，当然这些仅仅是非常基础性的东西，其中也很有一些可能是不妥当的地方，仅仅作为自己学习的一个记录而已。在这些基础上，还可以做些什么呢？比如用户的活动趋势？用户的活动量？等等都可以进行细化统计。

### ∫∫∫ 源码地址

整个过程中的代码都已经分享到了[Github](https://github.com/RobinChao/python-learning-tips/tree/master/dataframe)，以及使用了IPython进行一些初步的验证等等。欢迎Pull Request。

