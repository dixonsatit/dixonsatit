---
layout : post
title : การติดตั้งและใช้งาน Highchart 
---

heighchart เป็น chart ที่สามารถใช้งานได้ฟรีๆ ถ้าไม่ทำเป็นการค้านะครับ ซึ่งตัวมันเองมี chart มากมายหลายแบบ ใช้กันไม่หมดครับ เยอะจริงๆ

## ติดตั้ง

มีคนทำ extension ไว้ให้แระ [https://github.com/miloschuman/yii2-highcharts](https://github.com/miloschuman/yii2-highcharts)

```
composer require --prefer-dist miloschuman/yii2-highcharts-widget "dev-master"
```
หรือเพิ่มใน composer.json ในส่วน require

```
"miloschuman/yii2-highcharts-widget": "dev-master"
```
รันคำสั่ง composer update



## การใช้งาน

เท่าที่ผมสังเกต มีคนถามผมบ่อยๆ ว่าจะสร้าง array ในรูปแบบนีได้ยังไง query ออกมา จาก mysql ไม่ได้ตามแบบนี้

ขอแนะนำง่ายๆ ครับ เราต้องมองข้อมูลออกเป็นส่วนๆ query ออกมาทีละอันแล้วค่อยจับมารวมกันอีกที 

> อีกปัญหานึงที่เจอบ่อยๆ คือ  เมือเราสร้าง array ตามรูปแบบนี้ได้แล้วแต่ข้อมูลไม่ออกซักที ปัญหาส่วนใหญ่คือ type ของค่าต่างๆ ไม่ตรงเช่น มันรับเป็น init แต่ค่าที่อยู่ใน array เป็น string มันก็จะไม่แสดงนะครับ เพราะฉะนั้น var_dump เพื่อดูค่าของ array ว่ามี type ตรงกันหรือปล่าวครับ

```php
<?php

use miloschuman\highcharts\Highcharts;

echo Highcharts::widget([
   'options' => [
      'title' => ['text' => 'Fruit Consumption'],
      'xAxis' => [
         'categories' => ['Apples', 'Bananas', 'Oranges']
      ],
      'yAxis' => [
         'title' => ['text' => 'Fruit eaten']
      ],
      'series' => [
         ['name' => 'Jane', 'data' => [1, 0, 4]],
         ['name' => 'John', 'data' => [5, 7, 3]]
      ]
   ]
]);

```
