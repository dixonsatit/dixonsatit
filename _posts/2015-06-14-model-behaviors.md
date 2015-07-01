---
layout: post
title: การใช้งาน TimestampBehavior ใน model เพื่ออัพเดทข้อมูลวันที่
---

![timezone](/img/timezone.png)

การบันทึกเวลาเป็น init นั้นมีข้อดีคือ เราสามารถที่จะเปลี่ยนค่า Time  ที่เราบันทีกเป็น TimeZone ที่ใหนก็ได้ ทำให้เรารู้ว่า ณ ช่วงเวลาที่เราบันทึกข้อมูลตอนนี้ เป็นเวลากี่โมง ณ อีก timezone นึง

ตัวอย่างเช่นเรามีระบบ application ที่มีคนใช้อยู่คนละ timeZone เช่น `America/Los_Angeles`  กับ `Asia/Bangkok`

Yii 2 เราสามารถตั้งค่า `default`  TimeZone  ได้ที่ config/main.php ในส่วนของ `timeZone` ได้

```
[
    'timeZone' => 'America/Los_Angeles',
    // ...
]
```

> หากไม่รู้ว่า default ของเราเป็น timezone อะไร สามารถใช้ function ` date_default_timezone_set()` เพื่อ echo ตรวจสอบข้อมูลได้ เพราะการแสดงผลจะอิงตาม TimeZone


ส่วนใหญ่ในการสร้างตารางต่างๆ เราก็มักจะมีฟิวด์ที่เก็บข้อมูลคือ

- create_date บอกว่าเราสร้าง record นี้เมื่อไหร่
- update_date บอกว่ามีการแก้ไขข้อมูลเมื่อไหร่

ในการใช้การจริงๆ เราจะทำการใส่ข้อมูล create_date ในตอน create record และในตอน update เราก็ใส่ข้อมุล update_date ด้วย หรือไม่ก็เปลี่ยนประเภทข้อมูลให้เป็น timestamp แล้วก็ให้มันอัพเดทข้อมูลทุกครั้ง ที่มีการเปลี่ยนแปลงข้อมูล

แต่เดี๋ยวก่อน Yii2 มีนวัตกรรมใหม่ที่จะทำให้คุณพ่อบ้านสบายขึ้น (เดี่ยวๆ ก่อน..) มันดียังไง?

ตามเอกสารเค้าบอกว่ามันสามารถใส่ข้อมูล timestamp ให้เองอัตโนมัติ เพียงแค่เราตั้งชื่อฟิวด์ `created_at`,`updated_at` และกำหนด type ใน mysql เป็น Integer หลังจากนั้น เพิ่มการเรียกใช้งาน TimestampBehavior ใน Model ของคุณ โดยเพิ่ม `function behaviors()` เข้าไปใน Model ที่คุณต้องการใช้งาน

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

หลังจากกำหนดค่าแล้วหากมีการ create,update record `TimestampBehavior` จะใส่ข้อมูลให้เองอัตโนมัติ สะดวกมากๆ ครับ อย่าลืม!.. วิธีนี้ต้องกำหนดชื่อฟิวด์เป็น `created_at`,`updated_at` เท่านั้นครับ

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
            'updatedAtAttribute' => 'update_time'
        ],
    ];
}
?>
```

เมื่อดูข้อมูใน db จะได้ข้อมูลเป็นเลข `1435245364` ประมาณนี้

แต่ถ้าคุณไม่อยากเก็บเป็น timestamp อยากเก็บเป็น format ธรรมดา แบบนี้ `2015-03-20 10:32:48` ก็สามารถทำได้โดยเพิ่ม `'value' => new Expression('NOW()')` เข้าไป

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
จากนั้นเมื่อดูใน db ก็จะพบข้อมูล ประมาณนี้ `2015-03-20 10:32:48`

เราสามารถสั่งให้มันเรียกข้อมูล timestame ให้ฟิวด์นั้นๆ โดยตรง ได้โดยใช้คำสั่ง `touch` เพื่อใช้ในกรณีที่ เราต้องการเซ็ตค่าที่ไม่ต้องรอเวลาตาม event

```
$model->touch('creation_time');
```


## การใช้งานกับ GridView

```php
 <?= GridView::widget([
        'dataProvider' => $dataProvider,
        'filterModel' => $searchModel,
        'columns' => [
            'created_at:dateTime', // แสดงเฉพาวันที่ แสดงวันที่เวลา
            'updated_at:date', // แสดงเฉพาวันที่
            // กำหนดเอง อ่านเพิ่มเติม link ด้านล่าง
            ['attribute'=>'created_at','format'=>'html','value'=>function($model, $key, $index, $column){
                return Yii::$app->formatter->asDate($model->created_at,'medium'); //short,medium,long,full
                //return Yii::$app->formatter->asDateTime($model->created_at,'medium');
            }],

        ]
])
?>
```
## การใช้งานกับ DetailView

```php
   <?= DetailView::widget([
        'model' => $model,
        'attributes' => [
            'created_at:date', // แสดงเฉพาวันที่
            'updated_at:dateTime',// แสดงเฉพาวันที่ แสดงวันที่เวลา
             // กำหนดเอง อ่านเพิ่มเติม link ด้านล่าง
            [
                'attribute' => 'updated_at',
                'format'=>'html',
                'value' => Yii::$app->formatter->asDate($model->created_at,'medium')
            ]
        ],
    ]) ?>
```



อ่านการใช้งาน formatter  เพิ่มเติมได้[ที่นี่](/2015/06/23/date-formatter.html)
