---
layout: post
title: BlameableBehavior บันทึกข้อมูลรหัสผู้ใช้งานอัตโนมัติ
---

![](/img/bleamable-behavior.png)

ปกติเวลาที่เราสร้างตารางส่วนใหญ่ก็จะมีฟิวด์ `user_id` เพื่อบันทึกข้อมูลผู้ใช้งานที่ `Create` หรือ `Update` ข้อมูล ซึ่งเราต้องกำหนดค่ารหัสผู้ใช้งานทีล็อกอินเอง

yii2 ได้แก้ปัญหานี โดยใช้ `BlameableBehavior` เพื่อใช้งานกับ Model ที่เราต้องการได้ โดย `BlameableBehavior` มีค่าฟิวด์  `default` กำหนดไว้คือ `created_by`,`updated_by`  ถ้าหากเราตั้งชื่อตามนี้สามารถเรียกใช้งานตามโค้ดด้านล่างได้เลย

```php
<?php
use yii\behaviors\BlameableBehavior;

//.....
	public function behaviors()
	{
	    return [
	        BlameableBehavior::className(),
	    ];
	}
//.....
?>
```

แต่ถ้าหากเรากำหนดชื่อฟิดว์เป็นชื่ออื่นๆ ก็สามารถระบุได้เช่นกัน โดย defautl มันสามารถกำหนดได้ 2 ฟิวด์คือ createdByAttribute,updatedByAttribute ตามตัวอย่างผมเก็บแค่ ฟิวด์เดียวก็คือ `user_id` เลยเซ็ตค่าให้ตรงกันทั้ง 2 อัน

```php
<?php
use yii\behaviors\BlameableBehavior;

//.....
	public function behaviors(){
	        return [
	            [
	                'class' => BlameableBehavior::className(),
	                'createdByAttribute'=>'user_id',
	                'updatedByAttribute' => 'user_id',
	            ]
	        ];
	}
//.....
?>
```

หลังจากนั้นข้อมูลในส่วน user_id มันจะทำการเติมให้อัตโนมัติทุกครั้งที่มีการ create หรือ update ข้อมูล
