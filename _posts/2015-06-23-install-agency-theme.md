---
layout : post
title : การติดตั้งและใช้งาน agency theme
---
![agency theme](/img/agency-theme-top.png)
ใหนๆ ก็สอน[วิธีการสร้าง theme](/2015/06/20/create-theme-yii2.html) แล้ว ก็เลยเอา theme ที่สร้างมาทำเป็น extension ให้ติดตั้งใช้งานกันได้ง่ายๆ ผ่าน composer ซักหน่อย

theme นี้ฟรี และสามารถเอามาทำเป็น portfolio ของเราได้ หน้าตาก็ ok

## การติดตั้ง

```
 composer require dixonsatit/yii2-agency-theme
```
หรือ เพิ่มที่ composer.json ในส่วนของ require

```
"dixonsatit/yii2-agency-theme": "*"
```
จากนั้นรันคำสั่ง `composer update`

## การใช้งาน

หลังจากติดตั้งเสร็จ ให้ทำการตั้งค่าที่ config/main.php

```php
        'view'=>[
            'theme'=>[
                'pathMap'=>[
                    '@app/views'=>'@dixonsatit/agencyTheme/views'
                ]
            ]
        ],
```

ในการใช้งานจริงเราไม่ควรแก้ไขไฟล์ที่ `vendor/dixonsatit/yii2-agency-theme/views` เราควร copy ไฟล์ view มาไว้ที่ application ของเรา

- สร้างโฟลเดอร์ /themes/agency ที่ application ของคุณ
- copy ไฟล์ทั้งหมดที่อยู่ใน view จาก `vendor/dixonsatit/yii2-agency-theme/views` ไปไว้ที่ `/themes/agency` ใน application ของคุณ
- ตั้งค่า view ใหม่ เป็น

```php
 'view'=>[
            'theme'=>[
                'pathMap'=>[
                    '@app/views'=>'@app/themes/agency'
                ]
            ]
        ],
```

จากนั้นคุณสามารถปรับเปลี่ยน view ต่างๆ ได้ตามต้องการ ^^

![Agency Theme](https://github.com/dixonsatit/yii2-agency-theme/raw/master/dist/img/screencapture-yii2-agency-theme.png)
