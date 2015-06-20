---
layout : post
title : การสร้างและการใช้งาน AssetBundle
---

Asset Bundles คือตัวช่วยในการเรียกใช้งานกลุ่มไฟล์ต่างๆ ให้เราสามารถเรียกใช้งานได้ง่ายๆ ไม่ว่าจะเป็น css หรือ  js ยกตัวอย่าง เช่น 

- `yii\bootstrap\BootstrapAsset` เป็น assets ที่เอาไว้เรียกไฟล์ css ของ Bootstrap 
- `yii\bootstrap\BootstrapPluginAsset` เป็น assets ที่เอาไว้เรียกไฟล์ js ของ Bootstrap 
- `yii\web\YiiAsset` เป็น assets ที่เรียกใช้งาน yii.js 
- `yii\web\JqueryAsset` เป็น assets ที่เรียกใช้งาน JQuery
- `yii\jui\JuiAsset` เป็น assets ที่เรียกใช้งาน css,js jquery-ui
- `app\assets\AppAsset` เป็น asset ดีฟอลต์ที่เรียกใช้งาน site.css และ asset ตัวนี้ ยังได้เรียก `YiiAsset`,`BootstrapAsset` ด้วย

> สังเกตว่าแต่ละ asset จะเป็น asset เฉพาะเรื่องนั้นๆ เพื่อให้สามารถเลือกใช้แต่ละตัวได้ง่ายๆ


## การเรียกใช้งาน

เราสามารถเรียก  assets ที่ view ใหนก็ได้ หลังจากนั้นจะทำการ include css,js ตามที่ได้สร้าง asset ไว้

```php
use app\assets\AppAsset;
AppAsset::register($this);
```

## การสร้าง Asset

ตัวอย่างนี้เราจะสร้าง assets เพื่อใช้เรียก style.css,main.js มาใช้งาน โดยที่ไฟล์ css,js ของเราอยู่ที่โฟลเดอร์ web

สร้างโฟลเดอร์ชื่อ assets ที่ root โปรเจคเพื่อเก็บไฟล์ asset จากนั้นสร้างไฟล์ php ชือ MyAsstes.php

- `$basePath` พาทที่เก็บไฟล์จริง
- `$baseUrl` url ที่เรียกใช้งาน
- `$css` ระบุชื่อ css ที่เรียกใช้งาน สามารถระบุได้ทีละหลายๆ ตัว เป็น array
- `$js` ระบุชื่อ js ที่เรียกใช้งาน สามารถระบุได้ทีละหลายๆ ตัว เป็น array
- `$depends` เป็นการเรียกใช้ asset อื่นๆ เข้ามาใช้งานได้

```php
<?php
namespace app\assets;

use yii\web\AssetBundle;

class MyAsset extends AssetBundle
{
    public $basePath = '@webroot';
    public $baseUrl = '@web';
    public $css = [
        'css/style.css',
    ];
    public $js = [
    	'js/main.js'
    ];
    public $depends = [
        'yii\web\YiiAsset',
        'yii\bootstrap\BootstrapAsset',
    ];
}
```

ที่เรียกใช้งานที่ view ที่ต้องการ

```php
use app\assets\MyAsset;
MyAsset::register($this);
```
จากนั้นเมื้อเรา view page source ก็จะพบไฟล์ style.css,main.js ถูกเรียกใช้งาน และเรียกใช้งาน asset อื่นที่ $depends ได้ระบุไว้

## การสร้าง Asset จากเพ็คเก็จ Bower & NPM

หากใครเคยเขียน angularJs,Node.js เชื่อว่าคงคุ้นเคยกับ Bower และ NPM เป็นอย่างดีเพราะเป็นตัวจัดการไลบราลี่ให้ สามารถเรียกใช้งาน Web Library ต่างๆ เช่น jquery,bootstrap โดยที่เราไม่ต้องเข้าไปดาวน์โหลดไฟล์ หลังจากได้ไฟล์ก็ต้องทำการเรียกใช้งานเอง หากใช้งาน Bower มันสามารถ include  ไฟล์เข้ามาใช้งานได้เลยหรือยกเลิกการใช้งานไลบราลี่ให้เอง ซึ่งทำได้แค่ไม่กี่คำสั่ง ซึ่งสะดวกมากๆ (ในฝั่งของ javascript นะ)

ในส่วนของ Composer เป็นระบบจัดการไลบลาลี่ ของฝั่ง php บางครั้งเราก็จำเป็นต้องเรียกไลบลาลี่จาก bower เพราะ composer ไม่มี ดังนั้น composer จึงเปิดให้เราเรียกใช้งาน Bower ผ่านตัว composer ได้ด้วย

เราสามารถเรียกใช้งาน Bower และ NPM โดยใช้ Composer ทำได้ง่ายมากๆ เพียงแค่ระบุใน require `bower-asset/PackageName` สำหรับ bower และ `npm-asset/PackageName`  หากเป็น NPM ในไฟล์ composer.json

ตัวอย่างการเรียก fontawesome จาก Bower แพคเกจ ในไฟล์ composer.json

```json
 "require": {
    "php": ">=5.4.0",
    "yiisoft/yii2": "2.0.*",
    "bower-asset/fontawesome": "4.3.*" <<-----
  }

```

เมื่อรันคำสั่ง composer update แล้ว ไลบราลี่ในส่วนของ Bower จะถูกดาวน์โหลดมาไว้ที่ `vendor/bower/fontawesome` เราก็สามารถเขียน asset เพื่อเรียกใช้งาน css ตัวนี้ได้ โดยไม่จำเป็นต้อง copy ไฟล์ css มาไว้ที่ web/css เพียงแค่ระบุใน asset ที่เราสร้างว่า sorucePath ของเราอยู่ที่ @bower/fontawesome 

ต่อจากนั้น composer จะ copy ไฟล์ไปไว้ที่ web/assets/xxxx ซึ่ง xxxx จะเป็นชื่อโฟล์เดอร์ที่เก็บไฟล์ของแต่ละ asset และไฟล์ที่ใช้งานจริงๆ จะถูเรียกใช้งานจากที่นี่

สังเกต หากทำการ view page source ดูจะพบ url ในตอนเรียกใช้งานจะเป็นแบบนี้ `http://127.0.0.1/yii2/yii2-Leanning-Source/web/assets/9a53425f/css/font-awesome.min.css`

สร้างไฟล์ asset ชื่อ `FontAwesomeAsset` และทำการเรียกใช้งไฟล์จาก vendor\bower\fontawesome โดยระบุไว้ที่ $sourcePath = @bower/fontawesome

```php
<?php
namespace app\assets;

use yii\web\AssetBundle;

class FontAwesomeAsset extends AssetBundle 
{
    public $sourcePath = '@bower/fontawesome'; 
    public $css = [ 
        'css/font-awesome.min.css', 
    ];
    public $publishOptions = [
        'only' => [
            'fonts/',
            'css/',
        ]
    ];
}
```

ในส่วนของ $publishOptions เป็นการระบุว่าให้ copy ไปเฉพาะโฟลเดอร์ fonts,css เพราะปกติเมื่อเราระบุ `$sourcePath = '@bower/fontawesome'; ` และไม่ได้ระบุใน `$publishOptions` ในโฟลเดอร์ fontawesome มีอะไรมันจะทำการ copy ไปให้หมดเลย






