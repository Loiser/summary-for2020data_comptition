# 数据分析初探——以2020百度&西安交大大数据竞赛：传染病感染人数预测为例

&emsp;emm第一次参加这种比赛试水，虽然初赛的具体情况没有公布，但是感觉已经翻车了ORZ（端午节最后一天冲榜的都太猛了嘤）。已经有将近一年没怎么写程序了，感觉在重新学习python,作为一只菜狗还在ctrlC ctrlZ之间游走，虽然如此本菜在大佬的帮助下还是学习到了很多的知识。想做一个总结&对学到的ML，数据分析，数据挖掘知识进行梳理&练习markdown的书写&整理之前收藏的乱糟糟的链接（这次的文本是用jupter notebook写的，jupyter属实好用:3 !)比赛相关程序是用python写的，但是下面的内容不仅限于此次比赛与python,但可能举例多举此次比赛的例子。python和juyter相关下载配置部署详见后面有关部分。

## 比赛的大致情况

&emsp; 这次比赛比赛流程和一般的数据分析比赛的流程差不多，可能多了一个时间有点长（将近2个月）的初赛，复赛是一个月时间，决赛取复赛前一定名次做答辩。每天限制提交2次预测结果，计算误差进行排名，取排名靠前者。报名已经截至了，具体的情况见[官网](https://aistudio.baidu.com/aistudio/competition/detail/36)    
&emsp; 比赛的目标是用给出前45天5个城市的相关数据（包括感染人数，人流量密度，城市间人口流动量，城市内人口流动量，温度，湿度，风力，风向等）来预测往后30天5个城市，总计392个地区每日的的新增感染人数。   


## 环境配置与相关包的配置

本来这个比赛的目的之一是推广百度自己的paddlepaddle的框架，但是这个不太常用，而且很多API不知道是啥，然后就没用这个。   
### anaconda和pytorch（顺带tensorflow）的配置    
Anaconda是Python的一个开源发行版本，pytorch/tensorflow是神经网络常用的框架，[这个上面有安装配置方式](https://tangshusen.me/Dive-into-DL-PyTorch/#/chapter02_prerequisite/2.1_install) 我在安装时没按照这个来，但是一般来说anaconda的安装都是很简单的而且一般下载完后会自动配置环境变量，上网查到的方法也一般都能用（不管安装什么，都比较建议在CSDN等网站找一篇比较detailed，赞比较多，评论大多数都是感谢贴主的安装指南照着安装），安装完anaconda之后在anaconda navigator里或者prompt里安装pytorch,tensorflow,sklearn,numpy,pandas包都很容易。  
   
需要注意:   
1.如果你在使用SSR或者VPN科学地上网，可能会出现占用jupyter notebook打开的端口，把SSR关掉就能打开jupyter notebook了。   
2.在安装pytorch/tensorflow的时候建议不要在prompt里面使用conda install下载，这样下载的是pytorch和tensorflow的CPU版本，对应的GPU版本下载在后续进行说明。  
### GPU的配置
配这个是真的麻烦啊啊啊啊，当时搞了好多天。一般电脑里用NVIDIA的显卡可以按照下面的步骤，（原来把详细是步骤放在了收藏夹，后来发现在收藏夹里蒸发了？？？但是大致步骤如下，大家可以在网上找比较靠谱的教程（有的官方文档提供的安装教程不太好用）然后考虑下面的tips安装）   
1.cuda和cudnn的配置（参考网上的一些教程，官网上可以直接下载，但是要选择对版本，而且要在nvidia官网里注册）      
2.gpu版本pytorch和tensorflow的下载（参考网上的教程/在navigator里可以下gpu版本）     
3.pycuda可以试着用gpu跑普通的程序（还没有尝试，似乎效率不太高）
   
tips:   
1.安装中最容易出问题的是版本不兼容，需要注意python的版本，cuda的版本，显卡的版本，和下载的pytorch/tensorflow的版本，一般来说都是不同版本向下兼容，向上不兼容的，具体的情况可以参考网上的安装说明/另一种方法是安装后对版本升级（升级python &&升级显卡 比如我的电脑用的GeForce的显卡，就安装了GeForce Experience自动升级）    
2.tensorflow安装后使用时如果有no module named "tensorflow"的报错可能是因为安装路径不对，改这个很麻烦，建议卸载重下。     
3.能使用tensorflow/pytorch之后可能会有报错“ailed to get convolution algorithm. This is probably because cuDNN failed to...”类似的报错，可能是因为显存不够，建议关闭之前的运行的网络/增加显存/简化网络...   
### 写这个Notebook的配置   
这个文档书写用到的是jupyter notebook，编辑目录等需要下载扩展文件nbextension[具体下载方法](https://www.jianshu.com/p/f2adb4f64051),目录[怎么搞](https://www.jianshu.com/p/f314e9868cae)

## 数据处理

### 一些数据处理软件的选择

&emsp;数据处理方法和相关软件有很多种，并不是越复杂越高端越好，需要根据实际情况进行选择。Excel一般情况下都是超好用的!    
&emsp;在先前发的文章中有提到过SPSS，stata（虽然只是吐槽一下）[在这个的最后面](https://mp.weixin.qq.com/s/FJ0D67-ty7B4aNcRwx2c8Q)，还有SAS啥的对于一般的数据处理，简单的数据分析（计量上的面板数据多元logit,probit回归）都可以handle，而且用stata,spss之类有数据显示页面，能直接看着整个数据操作，而且很好学，而且很多都和excel一样能直接按button操作，python和matlab需要写和csv，excel的接口，暂时不能看到完整的数据状态，如果写相关程序不熟练会耗费更多的精力或者感觉很烦躁。    
&emsp;R没学过，但是听说相关的时序处理的包比较多但是处理的数据量不如python,而且比python慢。And,SQL,去年暑假计划好要学SQL的，但是最后竟然下了好多次没有下下来就鸽了（可能现在科学上个网在官网下会比较快，或者找国内的镜像）。    
&emsp;然后就是这次比赛主要使用的python的pandas包，开始还觉得pandas不好用。。。后来我就真香了！pandas真香！


### 这次比赛的数据处理

#### 比赛提供的数据格式

&emsp;把比赛用的数据包下载之后整理了一下各个包里的数据格式，提供的数据没有做train-data和test_data的划分，数据的内容主要是下面几种：
    
density:  数据完整   时间（按小时） 地点（经纬度）                人口流动量   
infection:数据完整   时间(按天)    地点（按区域（两个变量：城市&区域）） 新增感染   
weather:  有缺失     时间（按小时） 地点（按城市）                温度、湿度（有百分比）、风向：九个方向（加上无风）   
                 &emsp;&emsp;&emsp;&emsp;                     风速（<12km/h，16-24km/h,24-34km/h,三档&空白（包括无风和缺失值））   
							&emsp;&emsp;&emsp;&emsp;                          风力(0,1,2,3,4,<3,无缺失值)    
		&emsp;&emsp;&emsp;&emsp;                       天气（cloudy moderaterain rain overcast sunny fog lightrain)  &emsp;&emsp;&emsp;&emsp; 
				                          p.s.风力和天气无缺失值   
migration:数据完整   时间（按天）    地点：出发——到达（城市）           城市间迁徙指数   
transfer  :数据完整   时间（日均小时）地点 ：（城市内:出发经纬度，到达经纬度）  城市中迁移强度   
grid_attr: 数据完整              地点：经纬度                    属于该城市的哪个区域     			
							

#### 数据处理的主要步骤

下面主要是在这次比赛中用到的，更系统一点的方法[看这里](https://mp.weixin.qq.com/s/NKKk8nRd0qn5XhxXgYWknw)    
1.观察数据格式类型，了解数据内涵。   
2.可视化。         
&emsp;(1)看基本走势，（这个很重要，我们第一次提交就是直接根据infection的走势拟合了一个函数直接预测，有人拟合的好就直接进复赛辽）    
&emsp;(2)数据之间的关系    
&emsp;(3)数据情况（描述性统计）（噪声大的考虑分箱等方法降噪，噪声特别大的舍弃这个特征）。       
3.缺失值处理。       
&emsp;(1)直接删除[一些具体的做法](https://blog.csdn.net/sinat_25873421/article/details/80795499?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.nonecase&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-4.nonecase)    
&emsp;(2)pandas fillina函数的使用[具体看这里](http://www.cppcns.com/jiaoben/python/263449.html)    
&emsp;(3)用KNN或者决策树等方法填充    
5.异常值处理。   
&emsp;这次没怎么用...主要数据实在（全是噪声ORZ）   
4.数据格式转换。   
&emsp;(1)数据的编码与转换，详细见后面的部分。    
&emsp;(2)数据格式转换，主要做CNN等神经网络的时候的输入格式的调整。   

#### stata，SPSS数据处理的一些方法    
这些功能都能在pandas中实现

&emsp;混着用软件做数据处理属实弟弟行为，但是当时太菜了就没想太多ORZ，还是应该学好一样来做处理（首推pandas!）   
&emsp;因为开始的时候还不太会用pandas就想用其他的软件。开始使用SPSS，但是SPSS没有批量处理多个文件的能力，且无法提取符号和数字混合的变量（例如带%￥）
于是又用了stata，stata在查看数据窗口可以完成字符型变量与数值变量的转换，最妙的是 
即使你不知道对应程序怎么写，在窗口上点按钮操作后会自动出现相应的代码，另外stata
支持正则表达式的提取，[详见](https://stata-club.github.io/stata_article/2016-09-10.html)  
   
1.数据类型的转换     
声明数值变量的方法:gen variable=.   
字符到数值的转换:  
    &emsp;&emsp;   gen v5_change=.   
      &emsp;&emsp; replace v5_change=0 if v5=="Quiet"   
    &emsp;&emsp;   replace v5_change=1if v5=="East"     
在例如logit回归（附一个[logistic回归结果回归系数&OR值解读](http://m.iikx.com/e/action/ShowInfo.php?classid=17&id=6062)and [SPSS二项logistic回归的方法和解释](https://www.zhihu.com/question/34502688/answer/329779658)）中需要将分类变量转化为哑变量,避免将分类元的数值作为倍数关系处理。（例如在分类变量：性别中，将男变为0，女变为1）。在python 的pandas库中有更好的处理方法在后面会提到。   
   
   
2.缺失值处理   
&emsp;关于缺失值，实际上SPSS提供了取总体平均值，中位数，以及缺失值前后非缺失值平均数，中位数，线性估计。但是如果变量是int型，SPSS处理后会变成float。然后有人告诉我了一个神奇偏方（PS:此方法仅供娱乐）:先按照上述方法补全缺失值再取整，SPSS中取整数参考（SPSS中：转换->计算变量->函数组->字符串->Char.substr()）(取整方法:或者调整SPSS总体数据格式)


#### python pandas包的使用   
&emsp;一般来说pandas包和numpy包是配合使用的。[官方文档](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.fillna.html)关于pandas包的学习强列推荐:[这个](https://github.com/datawhalechina/joyful-pandas)里面有教程和习题，同样也可以当成工具书使用。    
&emsp;下面是一些写程序过程中常用的points,写在这里方便以后copy:

(1)读取csv文件加表头(不要学我这样的菜狗路径带中文):


```python
import pandas as pd
name=['date','hour','grid','infect','density','w_temp','w_humid','w_toward','w_speed','w_force','w_w','trans_1','trans_2']
data=pd.read_csv(r"C:\Users\10539\Desktop\数据竞赛\数据整理\except_migration.csv",header=None,names=name)#none表示原文件没有表头,header=i表示从第i+1行开始
data.head()#看前5排（加表头）
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_humid</th>
      <th>w_toward</th>
      <th>w_speed</th>
      <th>w_force</th>
      <th>w_w</th>
      <th>trans_1</th>
      <th>trans_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>16</td>
      <td>76</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>8.2</td>
      <td>7.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0.0</td>
      <td>987.2</td>
      <td>16</td>
      <td>76</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>17.9</td>
      <td>14.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0.0</td>
      <td>665.5</td>
      <td>16</td>
      <td>76</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>8.7</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0.0</td>
      <td>818.0</td>
      <td>16</td>
      <td>76</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>5.7</td>
      <td>7.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0.0</td>
      <td>1797.9</td>
      <td>16</td>
      <td>76</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>12.6</td>
      <td>15.6</td>
    </tr>
  </tbody>
</table>
</div>



(2)选几排:


```python
a=data[['date','hour']]
a.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



(3)索引:


```python
#方法1：loc 得到series或者dataframe
b=data.loc[3,['date']]
b
```




    date    0.0
    Name: 3, dtype: float64




```python
#方法2：[]  两种方法最后得到的值的形式不同
data['hour'][400]#先列后行
```




    1



(4)取值（数据类型不发生改变）:


```python
a=data[['date','hour']].values
a
```




    array([[ 0,  0],
           [ 0,  0],
           [ 0,  0],
           ...,
           [44, 23],
           [44, 23],
           [44, 23]], dtype=int64)




```python
a=data.loc[:,['date','hour']].values
a
```




    array([[ 0,  0],
           [ 0,  0],
           [ 0,  0],
           ...,
           [44, 23],
           [44, 23],
           [44, 23]], dtype=int64)




```python
a=data.loc[:,'date'].values#注意pandas中[]作为索引和表示列表的双重作用
a
```




    array([ 0,  0,  0, ..., 44, 44, 44], dtype=int64)



(5)变量变换，以标准化为例(注意其中lambda函数的使用):


```python
data[['density','w_temp','w_humid','trans_1','trans_2']]=data[['density','w_temp','w_humid','trans_1','trans_2']].transform(lambda x:(x-x.mean())/x.std())
```

(6)去掉行/列


```python
data=data.drop(['w_humid','w_toward','w_force'],axis=1)
data.tail()#看后5排
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_speed</th>
      <th>w_w</th>
      <th>trans_1</th>
      <th>trans_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>423355</th>
      <td>44</td>
      <td>23</td>
      <td>387</td>
      <td>6.375000</td>
      <td>143.4</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>0.5</td>
      <td>0.8</td>
    </tr>
    <tr>
      <th>423356</th>
      <td>44</td>
      <td>23</td>
      <td>388</td>
      <td>41.083333</td>
      <td>99.0</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>0.8</td>
      <td>0.9</td>
    </tr>
    <tr>
      <th>423357</th>
      <td>44</td>
      <td>23</td>
      <td>389</td>
      <td>25.375000</td>
      <td>0.2</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>423358</th>
      <td>44</td>
      <td>23</td>
      <td>390</td>
      <td>29.000000</td>
      <td>121.3</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>1.0</td>
      <td>1.8</td>
    </tr>
    <tr>
      <th>423359</th>
      <td>44</td>
      <td>23</td>
      <td>391</td>
      <td>4.125000</td>
      <td>157.7</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>1.1</td>
      <td>1.7</td>
    </tr>
  </tbody>
</table>
</div>



(7)增加列:


```python
trans_1=list(range(423360))
data['trans_1']=trans_1#trans
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_speed</th>
      <th>w_w</th>
      <th>trans_1</th>
      <th>trans_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>7.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0.0</td>
      <td>987.2</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>14.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0.0</td>
      <td>665.5</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0.0</td>
      <td>818.0</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>7.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0.0</td>
      <td>1797.9</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>15.6</td>
    </tr>
  </tbody>
</table>
</div>



(8)改变行/列名:


```python
#用字典的形式，不改变存储
data=data.rename(columns={'trans_1':1})
data.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_speed</th>
      <th>w_w</th>
      <th>1</th>
      <th>trans_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>7.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0.0</td>
      <td>987.2</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>1</td>
      <td>14.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0.0</td>
      <td>665.5</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>2</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0.0</td>
      <td>818.0</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>3</td>
      <td>7.9</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0.0</td>
      <td>1797.9</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>15.6</td>
    </tr>
  </tbody>
</table>
</div>



(9)按条件索引:


```python
wa=data.loc[lambda x :x['grid']==0]
wa.head()#注意新的dataframe每行的index保持原状
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_speed</th>
      <th>w_w</th>
      <th>1</th>
      <th>trans_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>7.8</td>
    </tr>
    <tr>
      <th>392</th>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>15</td>
      <td>1</td>
      <td>3</td>
      <td>392</td>
      <td>8.4</td>
    </tr>
    <tr>
      <th>784</th>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>14</td>
      <td>1</td>
      <td>2</td>
      <td>784</td>
      <td>7.7</td>
    </tr>
    <tr>
      <th>1176</th>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>14</td>
      <td>1</td>
      <td>2</td>
      <td>1176</td>
      <td>5.5</td>
    </tr>
    <tr>
      <th>1568</th>
      <td>0</td>
      <td>4</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>14</td>
      <td>0</td>
      <td>3</td>
      <td>1568</td>
      <td>12.2</td>
    </tr>
  </tbody>
</table>
</div>



(10)按行、列遍历:[详见](https://www.jb51.net/article/172623.htm)

(11)分类函数groupby


```python
#这个函数是真的好用
grid_pre=data.groupby('grid')
grid_pre#把data按照相同的grid进行划分
```




    <pandas.core.groupby.generic.DataFrameGroupBy object at 0x0000020A6F9DC608>




```python
x=grid_pre.get_group(200)#取grid=200的分组
x
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_humid</th>
      <th>w_toward</th>
      <th>w_speed</th>
      <th>w_force</th>
      <th>w_w</th>
      <th>trans_1</th>
      <th>trans_2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>200</th>
      <td>0</td>
      <td>0</td>
      <td>200</td>
      <td>0.000000</td>
      <td>252.1</td>
      <td>15</td>
      <td>96</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>4.3</td>
      <td>4.3</td>
    </tr>
    <tr>
      <th>592</th>
      <td>0</td>
      <td>1</td>
      <td>200</td>
      <td>0.000000</td>
      <td>252.1</td>
      <td>13</td>
      <td>96</td>
      <td>7</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>4.4</td>
      <td>4.8</td>
    </tr>
    <tr>
      <th>984</th>
      <td>0</td>
      <td>2</td>
      <td>200</td>
      <td>0.000000</td>
      <td>252.1</td>
      <td>14</td>
      <td>97</td>
      <td>7</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>2.5</td>
      <td>2.4</td>
    </tr>
    <tr>
      <th>1376</th>
      <td>0</td>
      <td>3</td>
      <td>200</td>
      <td>0.000000</td>
      <td>252.1</td>
      <td>15</td>
      <td>92</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>3.3</td>
      <td>3.0</td>
    </tr>
    <tr>
      <th>1768</th>
      <td>0</td>
      <td>4</td>
      <td>200</td>
      <td>0.000000</td>
      <td>252.1</td>
      <td>16</td>
      <td>90</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>3</td>
      <td>4.3</td>
      <td>5.3</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>421600</th>
      <td>44</td>
      <td>19</td>
      <td>200</td>
      <td>496.583333</td>
      <td>375.7</td>
      <td>16</td>
      <td>71</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>26.5</td>
      <td>24.6</td>
    </tr>
    <tr>
      <th>421992</th>
      <td>44</td>
      <td>20</td>
      <td>200</td>
      <td>495.666667</td>
      <td>375.7</td>
      <td>13</td>
      <td>71</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>17.0</td>
      <td>16.9</td>
    </tr>
    <tr>
      <th>422384</th>
      <td>44</td>
      <td>21</td>
      <td>200</td>
      <td>494.750000</td>
      <td>375.7</td>
      <td>13</td>
      <td>71</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>12.1</td>
      <td>17.2</td>
    </tr>
    <tr>
      <th>422776</th>
      <td>44</td>
      <td>22</td>
      <td>200</td>
      <td>493.833333</td>
      <td>375.7</td>
      <td>11</td>
      <td>71</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>11.7</td>
      <td>11.3</td>
    </tr>
    <tr>
      <th>423168</th>
      <td>44</td>
      <td>23</td>
      <td>200</td>
      <td>492.916667</td>
      <td>375.7</td>
      <td>13</td>
      <td>71</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>5.9</td>
      <td>5.4</td>
    </tr>
  </tbody>
</table>
<p>1080 rows × 13 columns</p>
</div>



需要注意的是groupby后的get_group得到的新dataframe里面数据的Index还是原来的index（200，592，984.....）没有重新排

#### numpy 包操作array

(1)写入读出:


```python
import numpy as np
trans=np.load(r'XXXX');
np.save(r'XXXX',trans)
```

(2)看形状:


```python
trans.shape
```

(3)list,array相互转化: 


```python
a=[[[1,5],[2,6]],[[3,7],[4,8]]]
a=np.array(a) #np.array（a）不改变a的内存
a.shape
```




    (2, 2, 2)




```python
b=list(a)
```

(4)索引:


```python
a[:,:,0]#抽出的保持原格式
```




    array([[1, 2],
           [3, 4]])




```python
a[:][1][0][1]=9#冒号和上面的冒号意思不同,可以将[:]看作不存在
a[1][0][1]
```




    9




```python
a[:][1][1]
```




    array([4, 8])



(5)和list对比    
（因为两者之间有区别在进行数据处理的时候，特别是多维度可能出现list里面套array的时候需要格外注意，尽量避免这种情况）


```python
b=[[[1,5],[2,6]],[[3,7],[4,8]]]
#b[:,:,0]#报错，无这种索引方式
b[:][1][0]#和array一样
#list没有shape,array不能append
```




    [3, 7]




```python
b*2#list外面的乘法表示copy
```




    [[[1, 5], [2, 6]], [[3, 7], [4, 8]], [[1, 5], [2, 6]], [[3, 7], [4, 8]]]




```python
a*2#array表示数乘
```




    array([[[ 2, 10],
            [ 4, 12]],
    
           [[ 6, 18],
            [ 8, 16]]])



(6)reshape&flatten函数(不改变内存)    


```python
a.reshape(2,4)#高维到低维是顺着排的，各维度之间的转换需要有整除关系
```




    array([[1, 5, 2, 6],
           [3, 9, 4, 8]])




```python
a.flatten()#flatten可以排成一维
```




    array([1, 5, 2, 6, 3, 9, 4, 8])



(7)合并:


```python
a=[[1,2],[3,4]]
b=[[5,6],[7,8]]
x=np.concatenate((a,b),axis=0)#按维度
x
```




    array([[1, 2],
           [3, 4],
           [5, 6],
           [7, 8]])




```python
x=np.concatenate((a,b),axis=1)
x
```




    array([[1, 2, 5, 6],
           [3, 4, 7, 8]])




```python
y=np.c_[a,b]#按列
y
```




    array([[1, 2, 5, 6],
           [3, 4, 7, 8]])



(7)初始化


```python
migration_data = np.zeros((6，6，6), dtype='float')
```

(8)不同维度互换（eg:（2，3，4）矩阵换成（4，3，2））


```python
a=np.array([[[1,2,3,4],[5,6,7,8],[9,10,11,12]],[[13,14,15,16],[17,18,19,20],[21,22,23,24]]])
a.shape
```




    (2, 3, 4)




```python
b= np.transpose(a,(2,1,0))#后面的（2，1，0）表示一个置换，即原来的shape:（2,3,4）中0号位置（'2'）现在在2号位置，1号位置('3')在1号位置
b.shape
```




    (4, 3, 2)



#### csv文件读写有关问题


```python
#写入list
import csv
with open(r'路径','a+',newline='') as f:#a+表示接着在文件后面写入
    writer=csv.writer(f)
    writer.writerows(data)
```

下面是一个读取csv存入list的例子，在读取后成行放入列表。    
需要注意的是假如没有for line in csv_reader:后面的temp=float(x) for x in line,直接存入points时得到的是包含str的List,且其中的逗号分隔符（，）也算在str里面


```python
import csv
points=[]
with open(r"C:\Users\10539\Desktop\数据竞赛\train_data\density_filled.csv")as f:
    csv_reader=csv.reader(f)
    for line in csv_reader:
        temp=[float(x) for x in line]
        points.append(temp)#直接读都是str
```

#### python 程序编写中发现的问题和tricks

（1）关于利用list&dict写循环和函数(这段代码没头没尾的仅提供形式)


```python
Cities = ["city_A", "city_B", "city_C", "city_D", "city_E"]
CitiesIndex ={'A':0,'B':1,'C':2,'D':3,'E':4}#索引方式: dict[key]

def find_place(direction:int) -> int:
    start = CitiesIndex[direction[0][-1]]#direction是list里面嵌套一个list 
    end = CitiesIndex[direction[1][-1]]
    return start * 4 + (end if start > end else end-1)

count = 0
for city in Cities:
    with open("train_data/"+city+"/migration.csv") as f:#利用list批量录入csv文件
        csv_reader = csv.reader(f,delimiter=',')
        for line in csv_reader:
            date = calcu_days(int(line[0]))
            index = find_place(line[1:3])
            for i in range(24):
                for j in range(grid_num[count]):
                    migration_data[date * 24 + i, sum(grid_num[:count]) + j, index] += float(line[-1])
    print(sum(grid_num[:count]))
    count+=1
```

（2）attention:关于Python中循环嵌套的问题:   
在多层循环中建议简单的循环放在里面，复杂的循环放在外面。例如:读取csv文件的循环放在外面，for i in range(5):这样的循环放在里面。  
因为python 的for 循环的内层循环是不支持循环读取文件等较复杂循环的。如果     


```python
for i in range(5):    
    for city in Cities:
        csv.DictReaderXXXX
    ......
#这样外面的for i 循环只会运行一次，但是可能不会报错
```

(3)list的enumerate遍历


```python
point=[13,56,12,44]
for index,row in enumerate(point):
    print(index,row)
```

    0 13
    1 56
    2 12
    3 44
    

(4)zip的用法    
zip的用法多样，在上面的enumerate不方便用的时候还可以利用range(len(list_a))和zip构成字典进行循环  
在组成字典和元组中都很方便[更多zip方法](https://www.cnblogs.com/xuchunlin/p/6676304.html):


```python
#(a)构成元组
a=[1,2,3,4]
b=[5,6,7,8]
c=zip(a,b)#c:[(1,5),(2,6),(3,7),(4,8)]
for (x,y) in c:print(x+y)
```

    6
    8
    10
    12
    


```python
#(b)构成字典
d=dict(zip(a,b))
d
```




    {1: 5, 2: 6, 3: 7, 4: 8}



(5)lambda 用法


```python
y=lambda x:2*x+1
y(2)
```




    5



#### 简单可视化


```python
import matplotlib.pyplot as plt#借助 matplotlib包
```

（1）画dataframe里的总图


```python
data.plot() #一般来说都没有标准化画出来的总图乱糟糟作用不大
plt.show()
```


![png](output_90_0.png)


(2)分开画


```python
data['infect'].plot()#默认x轴是index
```




    <matplotlib.axes._subplots.AxesSubplot at 0x18c05fb3908>




![png](output_92_1.png)



```python
plt.plot(data['grid'].values,data['infect'].values)#plot(x,y)
```




    [<matplotlib.lines.Line2D at 0x2eb218a88c8>]




![png](output_93_1.png)


(3)散点图:


```python
data.plot.scatter('density','trans_1')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x18c05593f48>




![png](output_95_1.png)


(4)箱形图:   
对箱型图的理解[看这里](https://blog.csdn.net/symoriaty/article/details/93978817)


```python
data[['density','trans_1']].boxplot()#数据情况属实不好
```




    <matplotlib.axes._subplots.AxesSubplot at 0x20a70841cc8>




![png](output_97_1.png)


#### 数据编码与转换      
#####  （1）热编码
one-hot热编码:将分类变量变成多个0-1变量       
关于连续函数离散化，分类指标哑元化[详见](https://blog.csdn.net/weixin_30596023/article/details/95459059)

pandas的get_dummies函数


```python
dataa=pd.get_dummies(data,columns=['w_w'])
dataa.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>hour</th>
      <th>grid</th>
      <th>infect</th>
      <th>density</th>
      <th>w_temp</th>
      <th>w_speed</th>
      <th>1</th>
      <th>trans_2</th>
      <th>w_w_0</th>
      <th>w_w_1</th>
      <th>w_w_2</th>
      <th>w_w_3</th>
      <th>w_w_4</th>
      <th>w_w_5</th>
      <th>w_w_6</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0.0</td>
      <td>951.8</td>
      <td>16</td>
      <td>1</td>
      <td>0</td>
      <td>7.8</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0.0</td>
      <td>987.2</td>
      <td>16</td>
      <td>1</td>
      <td>1</td>
      <td>14.1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0.0</td>
      <td>665.5</td>
      <td>16</td>
      <td>1</td>
      <td>2</td>
      <td>8.9</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0.0</td>
      <td>818.0</td>
      <td>16</td>
      <td>1</td>
      <td>3</td>
      <td>7.9</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>0.0</td>
      <td>1797.9</td>
      <td>16</td>
      <td>1</td>
      <td>4</td>
      <td>15.6</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



##### （2）时间序列处理
pandas里有[看这里的后面几个cap](https://github.com/datawhalechina/joyful-pandas)，在脉冲神经网络中会对时间有其他的编码方式。

##### （3）空间编码    
因为这次数据中有很多point to point 的数据，但是我们在操作的过程中处理的不太好（本来想到过用连接矩阵，但是与其它的数据格式不太兼容）。  
具体的一些空间编码的方向可以参考[这里](https://wenku.baidu.com/view/01b4bc84710abb68a98271fe910ef12d2bf9a97e.html)

## 机器学习和神经网络的初步了解    
&emsp;开始的时候用了sklearn做天气这些外源变量的回归，还尝试使用了XGBoost做了感染人数的回归，但是总体的回归效果不太好，就没搞sklearn了。     
&emsp;后来用了LSTM长短期记忆做回归，但是效果也不好，最后用了CNN卷积神经网络效果才好一点。


### sklearn包     
&emsp;sklearn包可以直接在anaconda prompt里直接下载，里面有线性回归到随机森林的一系列可以直接调用的相关函数。一般单核单线程，关于这个包的学习首推[中文文档](http://www.scikitlearn.com.cn/)   



#### KNN和SVM   
&emsp;KNN和SVM原本是两种分类算法。SVM通过超平面和核做分类划分，KNN则直接计算距离，根据距离远近做划分，具体的内容看上面的文档，网上的大多数博客也写的很清楚。  
    
&emsp;还可以借助KNN对分类变量做回归:


```python
from sklearn.neighbors import KNeighborsClassifier
X=data[['date','hour']]
X=X.values
y=data['w_speed']
y=y.values
#y=y.astype(np.int16) 
neigh =KNeighborsClassifier(n_neighbors=2)
neigh.fit(X,y)
predict_y= neigh.predict(predict_X)
```

&emsp;因为KNN是根据距离做的划分，但是不同维度的距离可能内涵不同，或者单位不统一，从而容易导致分类不准确，可以通过改变各维度的比重进行优化[具体看这里](https://blog.csdn.net/acceptedday/article/details/99681262)

&emsp;SVM里面的SVR可以用来做回归


```python
from sklearn.svm import SVR
X=data[['date','hour']]
X=X.values
y=data['w_temp']
y=y.values
model = SVR()
model.fit(X,y)
predict_y = model.predict(predict_X)
```

#### 训练集划分和参数训练      
主要是下面两个函数，具体看上面的sklearn文档，下面两个函数还可以用在XGBoost上具体的看下面[做房价预测的完整代码例子](https://cloud.tencent.com/developer/article/1331828)    



```python
from sklearn.model_selection import GridSearchCV     
from sklearn.model_selection import ShuffleSplit
cv_split = ShuffleSplit(n_splits=6, train_size=0.7, test_size=0.2)
#参数集写成字典形式
grid_params = dict(
    max_depth = [4, 5, 6, 7],
    learning_rate = np.linspace(0.03, 0.3, 10),
    n_estimators = [100, 200]
)
grid = GridSearchCV(model, grid_params, cv=cv_split, scoring='neg_mean_squared_error')
grid.fit(X, y)
#看结果
print(model.best_params_)
print('rmse:', (-grid_model.best_score_) ** 0.5)
```

### XGBoost

&emsp;XGBoost在一般的分类中超好用,主要原理是多个决策树的拼接和权重分配。具体原理[看这里](https://blog.csdn.net/huacha__/article/details/81029680)
还有[做房价预测的完整代码例子](https://cloud.tencent.com/developer/article/1331828)

&emsp;在回归中使用XGBoost中的XGBregressor函数[参数含义看这里](https://blog.csdn.net/qq_33799246/article/details/88930721):


```python
import xgboost

model=xgboost.XGBRegressor(objective ='reg:squarederror')
X=data[['date','hour']]
X=X.values
y=data['w_temp']
y=y.values
model=model.fit(X,y)    
pre=model.predict(predict_X)
```

&emsp;XGBoost还能做特征选取:


```python
from xgboost import plot_importance
print(model.feature_importances_)#选择得到的比重大的
```

除了searchCV外的[调参策略](https://zhuanlan.zhihu.com/p/106432182)

### LSTM    
关于LSTM和CNN之类的强烈建议看[这个](https://zhuanlan.zhihu.com/p/106432182)   
网络是用keras搭的，强烈推荐[keras文档](https://keras.io/zh/layers/convolutional/#conv3d)


```python
#一个效果不佳但是能运行的LSTM网络的例子
from keras.models import Sequential
from keras.layers import Dense
from keras.layers import LSTM
from keras.layers import Flatten
from keras.layers import Dropout

model = Sequential()
model.add(LSTM(64, input_shape=(80,8),return_sequences=True))
model.add(Dropout(0.5))
model.add(LSTM(64,return_sequences=True))#这个参数很关键保证了网络output的格式
model.add(LSTM(32,return_sequences=False))
model.add(Dense(80))
model.compile(loss='mae', optimizer='adam')
model.fit(train_x0, train_y0, epochs=10, batch_size=200)#validation_data=(test_x0, test_y0), verbose=2, shuffle=False)
predict_y0=model.predict(predict_x0)
```

LSTM同样能做多步多变量预测，和CNN做多步多变量预测相似

### CNN


```python
#先放个网络
import tensorflow as tf
input_shape = (48, 392, 7)
model = tf.keras.models.Sequential([
    tf.keras.layers.Conv2D(64, (24,1),activation='relu', input_shape=input_shape),#
    tf.keras.layers.Conv2D(64, (2,2), padding='same',activation='relu'),
    tf.keras.layers.Conv2D(64, (3,1),strides=(2,1), activation='relu'),
    tf.keras.layers.Dropout(.5),
    tf.keras.layers.Conv2D(32, (3,3),padding='same', activation='relu'),
    tf.keras.layers.Dropout(.5),
    tf.keras.layers.Conv2D(32, (12,1), activation='relu'),
    tf.keras.layers.Reshape((392,32)),
    tf.keras.layers.Dropout(.1),
    tf.keras.layers.Dense(21),
    tf.keras.layers.Dropout(.1),
    tf.keras.layers.Dense(14),
    tf.keras.layers.Dropout(.1),
    tf.keras.layers.Dense(7)
    #tf.keras.layers.Reshape((392,7))
])
model.compile(optimizer='adam',
              loss='mean_squared_error',
              metrics=[tf.keras.metrics.RootMeanSquaredError()])
model.summary()
```

    Model: "sequential_1"
    _________________________________________________________________
    Layer (type)                 Output Shape              Param #   
    =================================================================
    conv2d_5 (Conv2D)            (None, 25, 392, 64)       10816     
    _________________________________________________________________
    conv2d_6 (Conv2D)            (None, 25, 392, 64)       16448     
    _________________________________________________________________
    conv2d_7 (Conv2D)            (None, 12, 392, 64)       12352     
    _________________________________________________________________
    dropout_5 (Dropout)          (None, 12, 392, 64)       0         
    _________________________________________________________________
    conv2d_8 (Conv2D)            (None, 12, 392, 32)       18464     
    _________________________________________________________________
    dropout_6 (Dropout)          (None, 12, 392, 32)       0         
    _________________________________________________________________
    conv2d_9 (Conv2D)            (None, 1, 392, 32)        12320     
    _________________________________________________________________
    reshape_1 (Reshape)          (None, 392, 32)           0         
    _________________________________________________________________
    dropout_7 (Dropout)          (None, 392, 32)           0         
    _________________________________________________________________
    dense_3 (Dense)              (None, 392, 21)           693       
    _________________________________________________________________
    dropout_8 (Dropout)          (None, 392, 21)           0         
    _________________________________________________________________
    dense_4 (Dense)              (None, 392, 14)           308       
    _________________________________________________________________
    dropout_9 (Dropout)          (None, 392, 14)           0         
    _________________________________________________________________
    dense_5 (Dense)              (None, 392, 7)            105       
    =================================================================
    Total params: 71,506
    Trainable params: 71,506
    Non-trainable params: 0
    _________________________________________________________________
    


```python
model.fit(X,Y,batch_size=64,epochs=135)
test = model.predict([temp])
```
