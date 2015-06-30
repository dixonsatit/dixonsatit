---
layout : post
title : การใช้งาน Flash Message
excerpt : การใช้งาน Flash Message , yii2, yii Framework
---

FlashMessage เป็นตัวแจ้งสถานะต่างๆ ตามที่ต้องการ เช่น แจ้งบันทึกข้อมูลหรือแก้ไขข้อมูล โดยหลักการก็ใช้ข้อมูลเก็บไว้ใน session การทำงานจะแบ่งเป็น 2 ส่วนคือ `กำหนดค่า`,`แสดงผล`

## กำหนดค่า

เราสามารถกำหนดค่าในส่วนที่ต้องการได้ เช่น เมื่อมีการบันทึกข้อมูลเสร็จเราต้องการแจ้งเตือนไปยังผู้ใช้ว่า "บันทึกข้อมูลเสร็จเรียบร้อย" ก็ทำการกำหนดค่าลงไปดังนี้

```php
<?php
Yii::$app->getSession()->setFlash('alert',[
    'body'=>'ลงทะเบียนงานวิจัยเสร็จเรียบร้อย! เจ้าหน้าที่จะติดต่อกลับไปเร็วที่สุด..',
    'options'=>['class'=>'alert-success']
]);
?>
```
> หากต้องการแจ้งเตือนแบบต่างๆ สามารถกำหนดได้ ดังนี้ `alert-success`,`alert-info`,`alert-warning`,`alert-danger`


ตัวอย่างการใช้งานกับ `actionCreate`

```php
<?php
    public function actionCreate()
    {
        $model = new Research();
        if ($model->load(Yii::$app->request->post()) && $model->save()) {

            Yii::$app->getSession()->setFlash('alert',[
                'body'=>'ลงทะเบียนงานวิจัยเสร็จเรียบร้อย! เจ้าหน้าที่จะติดต่อกลับไปเร็วที่สุด..',
                'options'=>['class'=>'alert-success']
            ]);

            return $this->redirect(['view', 'id' => $model->id]);
        } else {
            return $this->render('create', [
                'model' => $model,
            ]);
        }
    }
?>
```

## การแสดงผล

เราจะใช้ `\yii\bootstrap\Alert` เป็น widget ของ Bootstrap ซึ่งมีมากับ yii2 อยู่แล้ว เราสามารถนำโค้ดนี้ไปไว้ที่ใหนก็ได้ที่เราต้องการแสดงผลของ alert ขึ้นมา

```php
<?php
use yii\helpers\ArrayHelper;
?>

<?php if(Yii::$app->session->hasFlash('alert')):?>
	<?= \yii\bootstrap\Alert::widget([
	'body'=>ArrayHelper::getValue(Yii::$app->session->getFlash('alert'), 'body'),
	'options'=>ArrayHelper::getValue(Yii::$app->session->getFlash('alert'), 'options'),
	])?>
<?php endif; ?>
```

## แสดงผลทีละหลายๆ ค่า

ในการกำหนดค่าเราสามารถกำหนดกี่ตัวก็ได้ แต่อย่าให้ key ซ้ำกัน ตามตัวอย่างผมตั้ง alert1,alert2

```php
<?php
     Yii::$app->getSession()->setFlash('alert1',[
        'body'=>'ลงทะเบียนงานวิจัยเสร็จเรียบร้อย! เจ้าหน้าที่จะติดต่อกลับไปเร็วที่สุด..',
        'options'=>['class'=>'alert-success']
     ]);

     Yii::$app->getSession()->setFlash('alert2',[
        'body'=>'ลงทะเบียนงานวิจัยเสร็จเรียบร้อย! เจ้าหน้าที่จะติดต่อกลับไปเร็วที่สุด..',
        'options'=>['class'=>'alert-info']
     ]);
?>
```

## ส่วนแสดงผล

ส่วนนี้มันจะมี funciton `getAllFlashes()` เพื่อดึงข้อมูล flashMessage ทั้งหมด จากนั้นเราก็ทำการวนลูปโดยใช้ foreach เพื่อมาแสดงผล

```php
<?php foreach (Yii::$app->session->getAllFlashes() as $message):; ?>
    <?= \yii\bootstrap\Alert::widget([
        'body'=>ArrayHelper::getValue($message, 'body'),
        'options'=>ArrayHelper::getValue($message, 'options'),
    ])?>
<?php endforeach; ?>
```

![](/img/flash-message.png)
