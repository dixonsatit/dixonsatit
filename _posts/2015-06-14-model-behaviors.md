---
layout: post
title: การใช้งาน TimestempBehavior ใน model เพื่ออัพเดทข้อมูลวันที่
---

ส่วนใหญ่ในการสร้างตารางต่างๆ เราก็มักจะมีฟิวด์ที่เก็บข้อมูลคือ 

- create_date บอกว่าเราสร้าง record นี้เมื่อไหร่
- update_date บอกว่ามีการแก้ไขข้อมูลเมื่อไหร่

ในการใช้การจริงๆ เราจะทำการใส่ข้อมูล create_date ในตอน create record และในตอน update เราก็ใส่ข้อมุล update_date ด้วย หรือไม่ก็เปลี่ยนประเภทข้อมูลให้เป็น timestamp แล้วก็ให้มันอัพเดทข้อมูลทุกครั้งที่มีการเปลี่ยนแปลงข้อมูล

แต่เดี๋ยวก่อน Yii2 มีนวัตกรรมใหม่ที่จะทำให้คุณพ่อบ้านสบายขึ้น (เดี่ยวๆ ก่อน..) มันดียังไง?

ตามเอกสารเค้าบอกว่ามันสามารถใส่ข้อมูล timestamp ให้อัตโนมัติได้เพียงแค่เราต้งชื่อฟิวด์ create_at,update_at และกำหนด type ใน mysql เป็น Integer หลังจากนั้นเพิ่มการเรียกใช้งาน TimestampBehavior ใน model ดังนี้

> มันเก็บข้อมูลเป็น time() นะครับ

```php
<?php
use yii\behaviors\TimestampBehavior;
public function behaviors()
{
    return [
        TimestampBehavior::className(),
    ];
}
?>
```

หลังจากเรียกใช้งานแล้วหากมีการ create,update record `TimestampBehavior` จะเป็นตัวที่ใส่ข้อมูลให้เองอัตโนมัติ สะดวกมากๆ ครับ อย่าลืม!.. วิธีนี้ต้องกำหนดชื่อฟิวด์เป็น create_at,update_at เท่านั้นครับ

แต่หากใครไม่ได้กำหนดชื่อฟิวด์ตามนี้ก็สามารถกำหนดให้ TimestampBehavior ให้มันได้รู้จักฟิวด์ create,update ได้ ดูตัวอย่างด้านล่าง

```php
<?php
use yii\db\Expression;
use yii\behaviors\TimestampBehavior;
public function behaviors()
{
    return [
        [
            'class' => TimestampBehavior::className(),
            'createdAtAttribute' => 'create_time',
            'updatedAtAttribute' => 'update_time',
            'value' => new Expression('NOW()'),
        ],
    ];
}
?>
```

เราสามารถสั่งให้มันเรียกข้อมูล timestame ให้ฟิวด์นั้นๆ ได้โดยใช้คำสั่งนี้

```
$model->touch('creation_time');
```

