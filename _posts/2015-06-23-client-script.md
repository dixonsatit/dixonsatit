---
layout : post
title : การ Register css,js ด้วย Client Script
---

Yii 2 มีตัวช่วยในการจัดการพวก script file เช่น include ไฟล์พวก css,js หรือแทรกโค้ด css,js บางส่วนเข้าไปใช้เฉพาะ action ได้ ซึ่งโดยปกติ ถ้าเป็นไฟล์ที่ใช้ในทุกๆ หน้า เราจะใช้ Asset เป็นตัวจัดการให้อยู่แล้ว

แต่ถ้าหากเราต้องการแทรกไฟล์หรือโค้ดบางส่วนก็สามารถทำได้ ซึ่งคุณสมบัติเหล่านี้มีอยู่ใน `yii\web\View ` อยู่แล้ว นั่นแสดงว่าคุณสามารถเรียกใช้ที่ view ใหนๆ ก็ได้ ลองใช้งานกัน

## Register JavaScript

เป็นการเขียนโค้ด javascript เข้าไปโดยตรงใน view ในนั้น หลักการก็เหมือนเราเขียน javascript ในแท็ก script เพียงแค่เราไม่ต้องประกาศเอง

```php
<?php
$options = ['success'=>true,'data'=>[1,2,3,4,5]];
$this->registerJs("var options = ".json_encode($options).";", View::POS_END, 'my-options');

$this->registerJs(" $('body').click(function(){
	alert(555);
}); ", View::POS_END, 'my-options');

```

`$this->registerJs($js, $position = self::POS_READY, $key = null)` จะรับค่า parameter ดังนี้

- `$js` คือคำสั่ง js ที่เราต้องการแทรกเข้าไปใน view
- `$position` คือตำแหน่งที่ใช้ให้แทรก javascript ไว้จุดใหนของ html
- `$key` ถ้าเรามี $this->registerJs() หลายๆจุดใหน หน้าเดียวกันแต่ไม่อย่างให้มันอยู่ใน script เดียวกันก็ตั้งเป็นคนละชื่อได้ ค่า defaul  ของมันคือ null

ตามตัวอย่างนี้ parameter แรกเป็นการประกาศตัวแปร​ javascript ชื่อ options เก็บข้อมูล json ที่แปลงจาก array php และตัวอย่างที่ 2 เป็นการเขียนดักจับ event ถ้ามีการคลิกภายใต้ tag `body` ก็จะ alert "555" ออกมา

 `position` ตัวที่กำหนดตำแหน่งว่าจะให้เอา script ที่ต้องการ register ไปไว้ส่วนใหนของ html มีตัวเลือกดังนี้

- `View::POS_HEAD` ในส่วน `<header>`
- `View::POS_BEGIN` ใต้แท็ก  `<body>` ตัวแรก
- `View::POS_END`  ก่อนแท็กปิด `</body>`
- `View::POS_READY` เป็นการประกาศเข้าไปในแท็ก ready ของ jquery เลย 
- `View::POS_LOAD` สำหรับเมื่อมีการรันโค้ดถึงจะทำการ register script ให้.


เมื่อ view page source ดูก็จะเห็นโค้ด Javascript ถูกแทกเข้าไปใน view ให้เอง

```html
<script type="text/javascript">var options = {"success":true,"data":[1,2,3,4,5]};</script>

<script type="text/javascript">jQuery(document).ready(function () {
 $('body').click(function(){
	alert(5);
}); 
});</script>

```

## Register Javascript File

เป็นการ include  ไฟล์เข้ามาใช้งานเลย และยังสามารถเรียกใช้งาน asset อื่นๆ ได้อีกด้วย ผ่าน `depends` เราก็ระบุชื่อ asset ที่ต้องการเรียกใช้งานเข้าไป ซึ่งตามตัวอย่าง ก็จะทำการ include main.js และทำการเรียกใช้งาน `JqueryAsset` เพื่อ include ไฟล์ jquery ด้วย

```php
<?php
$this->registerJsFile('http://example.com/js/main.js', ['depends' => [\yii\web\JqueryAsset::className()]]);

```

## Registering CSS

เราสามารถที่จะแทรกสไตล์ใน view ได้เลยผ่าน `registerCss` สามารถใส่คำสั่ง css เข้าไปได้เลยตามตัวอย่าง

```
$this->registerCss("body { background: #f00; }");
```

เมื่อ view page source จะเห็นเป็นแบบนี้

```html
<style>
body { background: #f00; }
</style>
```

## Registering CSS File

เป็นการ register ไฟล์ css เข้ามาใช้งาน ใน view ของเราชื่อ black-and-white.css พร้อมทั้งเรียกใช้งาน BootstrapAsset ซึ่งเป็น css ของ bootstrap มาใช้งานด้วย และระบุ css media เท่ากับ print ด้วย 

> css media คือกำรหนดการแสดงผลหน้าตาเว็บเพจ บนสื่อประเภทต่างๆ เราสามารถกำหนด style ให้แตกต่างกันได้ เช่น ตอนที่เราเห็นบนหน้าจอ (screen) กับตอนที่สั่งพิมพ์ออกกระดาษ (print) ให้แสดงผลไม่เหมือนกัน  [อ่าน css media ได้ที่นี](http://www.enjoyday.net/webtutorial/css/css_chapter22.html)

```php
<?php
$this->registerCssFile("http://example.com/css/themes/black-and-white.css", [
    'depends' => [BootstrapAsset::className()],
    'media' => 'print',
], 'css-print-theme');

$this->registerCssFile("@web/css/style.css");
```

หาก view page source ก็จะพบการเรียกใช้งาน css 

```html
<link href="/yii2/yii2-Leanning-Source/web/css/style.css" rel="stylesheet">
<link href="/yii2/yii2-Leanning-Source/web/assets/99615aa3/css/bootstrap.css" rel="stylesheet">
<link href="http://example.com/css/themes/black-and-white.css" rel="stylesheet" media="print">
```

[อ่านเพิ่มเติมได้ที่นี่](http://www.yiiframework.com/doc-2.0/guide-output-client-scripts.html)
