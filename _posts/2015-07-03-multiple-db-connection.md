---
layout : post
title : การเปิดใช้งาน Multiple Database Connection
---
![](/img/multiple-db.jpg)

Yii 2 สามารถตั้งค่า connection db ได้หลายๆ ฐานข้อมูลหรือ หลายๆ server แม้จะไม่ได้อยู่ใน ip เดียวกัน เช่น เรามีฐานข้อมูล อยู่ 2 ที่คนละ ip กัน ตัวอย่าง

-  MySql 192.168.0.101
-  MySql 192.168.0.102

> **Tip** : ในกรณีที่ฐานข้อมูลอยู่ที่ ip เดียว เราไม่จำเป็นต้องตั้งค่าแยก สามารถระบุชื่อ ฐานข้อมูลแล้วตามด้วยชื่อ table ในส่วน model ใน function tableName()ได้ เช่น `intranet.employee`  

## สำหรับ Basic

เราสามารถตั้งค่า db ได้ที่ `config/web.php` มองหา `db` จะอยู่ด้านล่าง

```
  'db' => require(__DIR__ . '/db.php')
```
ทำการ copy บรรทัดนี้แล้ววางตั้งชื่อใหม่เป็นฐานที่เราต้องการ และตั้งชื่อไฟล์ config db ใหมเป็น db2

```
  'db' => require(__DIR__ . '/db.php'),
  'db2' => require(__DIR__ . '/db2.php')
```
> 'db' => require(__DIR__ . '/db.php') ตัว default Connection ห้ามเปลี่ยนะครับ ต้องมีอย่างน้อย 1 ตัวที่ชื่อ ว่า db แนะนำเป็นฐานข้อมูลหลักที่เราใช้งาน

และถ้าหาเราเปิดไฟล์ db.php จะพบการตั่งค่า db ดังนี้ ให้เราตั้งค่าตามต้องการ และ copy ไฟล์นี้ตั้งชื่อใหม่เป็น db2.php

**db.php**

```php
<?php
return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=192.168.0.101;dbname=yii2-workshop1',
    'username' => 'user',
    'password' => '1234',
    'charset' => 'utf8',
];

```

**db2.php**

```php
<?php
return [
    'class' => 'yii\db\Connection',
    'dsn' => 'mysql:host=192.168.0.102;dbname=yii2-workshop2',
    'username' => 'user',
    'password' => '54321',
    'charset' => 'utf8',
];

```

จากนั้นเราสามารถเรียกใช้งานโดยใช้ชื่อ connection ใหม่ ได้ คือ db2

หรือเพิ่มง่ายๆ ในไฟล์ `config/web.php` แบบนี้ก็ได้

```php
'db' => require(__DIR__ . '/db.php'),
'db2'=>[
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=192.168.0.102;dbname=yii2-workshop2',
            'username' => 'root',
            'password' => 'root',
            'charset' => 'utf8',
        ]
```

## สำหรับ Advance

สำหรับ advance จะอยู่ที่ common/config/main-local.php

```php
<?php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=127.0.0.1;dbname=yii2-workshop-day4',
            'username' => 'root',
            'password' => 'root',
            'charset' => 'utf8',
        ],
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
            'viewPath' => '@common/mail',
            // send all mails to a file by default. You have to set
            // 'useFileTransport' to false and configure a transport
            // for the mailer to send real emails.
            'useFileTransport' => true,
        ],
    ],
];

```

เราสามารถเพิ่ม connection ได้ดังนี้

```php
<?php
return [
    'components' => [
        'db' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=192.168.0.101;dbname=yii2-workshop1',
            'username' => 'root',
            'password' => 'root',
            'charset' => 'utf8',
        ],
        'db2' => [
            'class' => 'yii\db\Connection',
            'dsn' => 'mysql:host=192.168.0.102;dbname=yii2-workshop2',
            'username' => 'root',
            'password' => 'root',
            'charset' => 'utf8',
        ],
        'mailer' => [
            'class' => 'yii\swiftmailer\Mailer',
            'viewPath' => '@common/mail',
            // send all mails to a file by default. You have to set
            // 'useFileTransport' to false and configure a transport
            // for the mailer to send real emails.
            'useFileTransport' => true,
        ],
    ],
];

```


## การ Gii Model โดยใช้ connection ใหม่ที่เราสร้าง

เมื่อเราทำการ Gii Model จะมีส่วนของการตั้งค่า `Database Connection ID`  อยู่ซึ่งปกติค่า default จะชื่อ  `db`

![](/img/connection-db.png)

เราสามารถตั้งค่าเป็น db2 ดังนี้

![](/img/connection-db2.png)


## การ เรียกใช้ใน Model
หาก Gii ตามด้านบนเมื่อดูในไฟล์ model ที่เราสร้างขึ้นมาจะพบว่ามันเพิ่ม `function  getDb()` ขึ้นมา เพื่อให้ไปเรียกใช้ db2 ซึ่งปกติดหากไม่ได้ตั้งค่าใหม่ มันจะใช้ค่า default คือ `db`

ถ้าหาไม่ได้ gii ก็สามารถเพิ่ม function นี้เข้าไปเองได้ แล้วทำการเปลี่ยนเป็นชื่อ connection ที่ต้องการเรียกใช้งาน

```php
  public static function getDb()
  {
      return Yii::$app->get('db2');
  }
```

## การ เรียกใช้ใน SQL Queries

```
$posts = Yii::$app->db2->createCommand('SELECT * FROM post')->queryAll();

Yii::$app->db2->createCommand((new \yii\db\Query)->select('*')->from('tbl_name'))->queryAll()
```
