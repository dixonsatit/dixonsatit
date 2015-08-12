---
layout : post
title : การเปลี่ยนหน้า Default Error ให้สวยงาม
---

![](/img/thumbnail/default-error.jpg)

เวลาที่มี error ที่ไม่ใด้เกิดจาก syntax error มันจะ redirect ไปที่หน้า default error ปกติจะอยู่ที่ SiteController ใน action `error` ซึ่งมันเป็น class action ที่มีไว้ให้เรียกใช้งานอยู่แล้ว เมื่อเข้าไปดูใน controller จะเป็นดังนี้

```php
public function actions()
{
    return [
        'error' => [
            'class' => 'yii\web\ErrorAction',
        ],
    ];
}
```

เราสามารถเปลี่ยน view ในส่วนที่แสดงข้อความ error ได้ โดยสามารถปรับเปลี่ยนเองได้หมดซึ่ง ErrorAction จะส่งตัวแปรมาให้ใช้ 2 ตัวคือ

- `$name` ตัวแปรนี้จะแสดงชื่อ error ต่างๆ เช่น `Not Found (#404)`,`Forbidden (#403)`
- `$message` เป็นข้อความแสดง error ต่างๆ ให้ผู้ใช้งานได้ทราบ

ซึ่งหากเราต้องการเปลี่ยนแปลงหน้าตาส่วน error ก็สามารถทำได้โดยแก้ไขที่ไฟล์ view ชื่อ error ใน views/site/error.php ก็ปรับแต่งตามความต้องการได้ ผมทำตัวอย่างง่ายๆ มาให้ดูกัน

```php
<?php

/* @var $this yii\web\View */
/* @var $name string */
/* @var $message string */
/* @var $exception Exception */

use yii\helpers\Html;
use yii\helpers\Url;

$this->title = $name;
?>
<div class="jumbotron">
      <h1><?=Html::img(Url::base().'/images/Minion_icon.png')?><?= Html::encode($this->title) ?></h1>
      <p class="text-danger"><?= nl2br(Html::encode($message)) ?></p>
      <p><a class="btn btn-default btn-lg" href="http://dixonsatit.github.io/" role="button">Learn more</a></p>
</div>
```

![](/img/error.png)

จะได้หน้าตาประมาณนี้ ก็ดูดีกว่าของเก่าเยอะ ^ ^ ทีนี้ความสวยงามก็ขึ้นอยู่กับการ design ของเราแล้วครับ

> ตัว Minion โหลดได้ที่นี่ครับ https://www.iconfinder.com/icons/174006/minion_icon#size=128
