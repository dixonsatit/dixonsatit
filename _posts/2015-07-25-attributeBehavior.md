---
layout : post
title : การใช้งาน AttributeBehavior
---

![](/img/thumbnail/attribute-behavior.jpg)

`AttributeBehavior` คือการดักจับ event ต่างๆ ของ model เช่น EVENT_BEFORE_INSERT,EVENT_BEFORE_UPDATE เพื่อที่เราจะทำงานบางอย่าง กับฟิวด์ที่เราต้องการ ดูตัวอย่าง

ตัวอย่างนี้เราต้องการแปลงข้อมูล array ที่ได้จาก CheckBoxList ให้เป็น string ก่อนบันทึกและแก้ไขข้อมูล ซึงเราจะทำการเรียกใช้ฟังก์ชัน `behaviors()`  ของ model ที่เราใช้งานและเพิ่มฟังกชันนี้เข้าไป


```php
<?php
use yii\behaviors\AttributeBehavior;
use yii\db\ActiveRecord;
// ...

public function behaviors()
{
    return [
        [
            'class' => AttributeBehavior::className(),
            'attributes' => [
                ActiveRecord::EVENT_BEFORE_INSERT => 'skill',
                ActiveRecord::EVENT_BEFORE_UPDATE => 'skill',
            ],
            'value' => function ($event) {
                return implode(',', $this->skill);
            },
        ],
    ];
}
```
จากโค้ดจะเห็นว่าเราทำการเรียกใช้งาน `AttributeBehavior` ในฟังก์ชัน `behaviors()` และกำหนดคุณสมบัติเข้าไปดังนี้

- `class` คือชื่อของ behavior ที่เราต้องการใช้ ในตัวอย่างนี้คือ AttributeBehavior
- `attributes` เป็นการระบุว่าเราจะกระทำข้อมูลในเหตุการณ์อะไร และกับฟิวด์อะไร ซึ่งเราระบุเหตุการณ์ before-insert,before-update และระบุฟิวด์คือ skill
- `value` คือส่วนที่เราจะทำงานบางอย่างเพื่อให้ค่ากับ skill ซึ่งในตัวอย่างเราจะทำการแปลงข้อมูลเดิมที่เป็น array ที่ได้จาก CheckBoxLis และแปลงให้เป็น string โดยใช้ `implode()`

น่าจะพอเข้าใจไม่มากก็น้อย ผมคิดว่ามันคงสามารถนำไปประยุกใช้งานกับงานลักษณะอื่นๆ ได้

@dixonSatit
