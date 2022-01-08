---
title: python爬取天气
date: 2021-02-22 15:09:44
tags:
	- python
	- 爬虫
---

### 准备工作

首先导入一些需要用到的库

```python
from bs4 import BeautifulSoup #网页解析
import re	#正则表达式
import urllib.request,urllib.error #制定URL，获取网页数据
```

若没有所需的库命令行输入pip install 需要下载的库命，下载即可

<!-- more -->

### 代码分析

1. **爬取网页**

将需要爬取的网页网址保存起来，以银川为例

```python
baseUrl="http://www.nmc.cn/publish/forecast/ANX/yinchuan.html"
```

将网址传入到下面的函数中，获取网页源码，返回值为字符串

```python
#获取网页源码
def askURL(url):
    # 模拟浏览器头部信息，向服务器发送消息
    head={"User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.100 Safari/537.36"}
    # 用户代理，表示告诉豆瓣服务器，我们是什么类型的机器、浏览器（本质上是告诉浏览器，我们可以接收什么水平的文件内容）
    request=urllib.request.Request(url,headers=head)
    html=""
    try:
        response = urllib.request.urlopen(request)
        #设置utf-8编码，防止乱码
        html = response.read().decode("utf-8")
    except urllib.error.URLError as e:
        if hasattr(e, "code"):
            print(e.code)
        if hasattr(e, "reason"):
            print(e.reason)
    return html
```

此处的User-Agent获取方法为，打开浏览器进入想要爬取的页面，按下F12点击NetWork，然后F5刷新界面，会出现以下内容

![image-20210222152256383](python%E7%88%AC%E5%8F%96%E5%A4%A9%E6%B0%94/image-20210222152256383.png)

按顺序点击，找到User-Agent，复制到代码中

![image-20210222152510259](python%E7%88%AC%E5%8F%96%E5%A4%A9%E6%B0%94/image-20210222152510259.png)

如果不写这一段代码模拟浏览器访问网页，很多时候会被认出是爬虫然后被拒绝访问

2.**解析数据**

我们想要爬取的天气数据集中在这一块

![image-20210222153236986](python%E7%88%AC%E5%8F%96%E5%A4%A9%E6%B0%94/image-20210222153236986.png)

对应HTML的标签为div class="weatherWrap"，于是在解析时先定位到这个标签下的字符串，利用之前导入的BeautifulSoup库

```python
soup = BeautifulSoup(html,"html.parser")
for item in soup.find_all('div',class_="weatherWrap"):  #查找符合要求的字符串，形成列表
```

接下来是对需要取得的详细信息的解析，这里一般用正则表达式来抓取

```python
findDate=re.compile(r'<div class="date"> (.*?) <br/>')	#日期
findWeek=re.compile(r'<br/>(.*?) </div><div class="weathericon">')	#星期
findImage=re.compile(r'<div class="weathericon"><img src="(.*?)"/></div>')	#图片
findweather=re.compile(r'<div class="desc"> (.*?) </div><div class="windd">')	#天气
findWindDir=re.compile(r'<div class="windd"> (.*?) </div><div class="winds">')	#风向
findWind=re.compile(r'<div class="windd"> .*? </div><div class="winds"> (.*?) </div>')	#风
findTemperature=re.compile(r'<div class="tmp tmp_lte_.*?"> (.*?) </div>')	#温度
```

爬取后我们将数据保存在列表里返回

```python
def getData(baseurl):
    datalist=[]
    html=askURL(baseurl)
    #解析数据
    soup = BeautifulSoup(html,"html.parser")
    for item in soup.find_all('div',class_="weatherWrap"):  #查找符合要求的字符串，形成列表
        data=[]  #保存天气的所有信息
        item=str(item)
        date=re.findall(findDate,item)[0]
        data.append(date)

        week=re.findall(findWeek,item)[0]            
        data.append(week)

        image=re.findall(findImage,item)
        data.append(image)    

        weather=re.findall(findweather,item)
        data.append(weather)

        windDir=re.findall(findWindDir,item)
        data.append(windDir)

        wind=re.findall(findWind,item)
        data.append(wind)

        temperature=re.findall(findTemperature,item)
        data.append(temperature)

        datalist.append(data)
    return datalist
```

至此我们想要的数据就爬取完成了，输出到文件里验证是否正确

![image-20210222154120685](python%E7%88%AC%E5%8F%96%E5%A4%A9%E6%B0%94/image-20210222154120685.png)

![image-20210222154136724](python%E7%88%AC%E5%8F%96%E5%A4%A9%E6%B0%94/image-20210222154136724.png)

