---
title: 每日天气
date: 2021/3/6 13:39:17
---


<style>
    /*--预设--*/ 
    body { padding:0px;margin: 0px; } 
    #lyrow, #lyrow input, #lyrow textarea { font-size:12px;font-family: 'Microsoft YaHei', '微软雅黑', MicrosoftJhengHei, '华文细黑', STHeiti, MingLiu; } 
    #lyrow { height:100vh;width: 100vw; } 
    #lyrow div { min-height: 18px;  } 
    #lyrow input, #lyrow textarea { border:rgb(235, 235, 235) 1px solid;border-radius: 3px;padding: 5px 8px;outline: 0; } 
    #lyrow input:hover, #lyrow textarea:hover { border: 1px solid #6bc1f2; } 
    /*--编辑--*/ 
    #lyrow .weather { width:900px !important;height:450px !important;margin:30px 30px 0 30px;box-shadow:0 29px 20px 2px rgba(191, 191, 191, 0.84);justify-content:center;display:flex !important;align-items:center;border-radius:20px 20px 20px 20px;background-size:cover;background-image:url(/daily_weather/weather_bg.png); } 
    #lyrow .daily { width:120px !important;height:400px !important;border-left:rgba(98, 214, 69, 1) solid 3px !important;background-color:rgba(0, 191, 255, 0.6); } 
    #lyrow .today { background-color:rgba(34, 136, 191, 0.8);width:120px !important;height:410px !important; } 
    #lyrow .data { width:120px !important;height:55px !important;background-color:rgba(224, 224, 224, 0.31);font-size:20px;font-weight:bold;text-align:center;color:rgba(255, 255, 255, 1);line-height:25px; } 
    #lyrow .halfday { width:120px !important;height:170px !important;flex-direction:column; } 
    #lyrow .temperature { background-color:rgba(148, 232, 65, 0.7);width:120px !important;height:35px !important;position:relative;font-size:28px;font-weight:lighter;text-align:center;color:rgba(255, 255, 255, 1);top:7px; } 
    #lyrow .temperaturedown { background-color:rgba(12, 230, 190, 0.7);position:relative;height:35px !important;font-size:28px;font-weight:lighter;text-align:center; } 
    #lyrow .img { width:50px !important;height:50px !important;position:relative;top:5px;left:34px; } 
    #lyrow .info { width:120px !important;height:78px !important;font-size:20px;font-weight:lighter;text-align:center; } 
</style>
<h1>武汉七日天气</h1>
<div id="lyrow"><div class="weather">
<div class="today">
<div class="data">03/06<br>周六</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/7.png" class="img">
<div class="info">小雨<br>北风<br>4~5级</div>
<div class="temperature">10℃</div></div>
<div class="halfday">
<div class="temperaturedown">4℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/2.png" class="img">
<div class="info">阴<br>北风<br>4~5级</div> </div> </div>

<div class="daily">
<div class="data">03/07<br>周日</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/0.png" class="img">
<div class="info">晴<br>北风<br>3~4级</div>
<div class="temperature">11℃</div></div>
<div class="halfday">
<div class="temperaturedown">4℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/7.png" class="img">
<div class="info">小雨<br>东风<br>微风</div> </div> </div>

<div class="daily">
<div class="data">03/08<br>周一</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/7.png" class="img">
<div class="info">小雨<br>东北风<br>微风</div>
<div class="temperature">9℃</div></div>
<div class="halfday">
<div class="temperaturedown">4℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/1.png" class="img">
<div class="info">多云<br>北风<br>微风</div> </div> </div>

<div class="daily">
<div class="data">03/09<br>周二</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/2.png" class="img">
<div class="info">阴<br>北风<br>微风</div>
<div class="temperature">15℃</div></div>
<div class="halfday">
<div class="temperaturedown">7℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/1.png" class="img">
<div class="info">多云<br>东北风<br>3~4级</div> </div> </div>

<div class="daily">
<div class="data">03/10<br>周三</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/7.png" class="img">
<div class="info">小雨<br>东风<br>3~4级</div>
<div class="temperature">16℃</div></div>
<div class="halfday">
<div class="temperaturedown">9℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/7.png" class="img">
<div class="info">小雨<br>东北风<br>微风</div> </div> </div>

<div class="daily">
<div class="data">03/11<br>周四</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/1.png" class="img">
<div class="info">多云<br>北风<br>微风</div>
<div class="temperature">18℃</div></div>
<div class="halfday">
<div class="temperaturedown">10℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/0.png" class="img">
<div class="info">晴<br>西北风<br>微风</div> </div> </div>

<div class="daily">
<div class="data">03/12<br>周五</div>
<div class="halfday">
<img src="http://image.nmc.cn/assets/img/w/40x40/4/1.png" class="img">
<div class="info">多云<br>无持续风向<br>微风</div>
<div class="temperature">21℃</div></div>
<div class="halfday">
<div class="temperaturedown">8℃</div>
<img src = "http://image.nmc.cn/assets/img/w/40x40/4/1.png" class="img">
<div class="info">多云<br>无持续风向<br>微风</div> </div> </div>

</div></div>
<h1>更新时间:2021年3月6日13:39:17</h1>
