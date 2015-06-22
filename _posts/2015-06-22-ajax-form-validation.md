---
layout : post
title : การเปิดใช้งาน Ajax Form Validation
---

ปกติ Yii 2 จะมี validation อยู่แล้วแต่มันจะเช็คก็ต่อเมื่อ มีการกดปุ่ม submit form ถึงจะแจ้ง error ออกมา ถ้าเทียบกับ ajax มันก็ถึอว่าช้าอยู่เหมือนกัน

แต่ช้าก่อนเรายังมี ajax validation มาช่วยได้!   ข้อดีของการใช้ ajax validation form คือ ไม่ต้องกดปุ่ม submit form หากมีการกรอกข้อมูล ก็จะทำการเช็คให้ทันที ทำให้ validation ได้เร็วขึ้น

เราสามารถเปิดใช้งาน Ajax Validation Form ได้ เพียงแค่ 2 ขั้นตอน

ทำการเปิดการใช้งาน Ajax Validation ก่อนที่ ActiveForm โดยเปิด `'enableAjaxValidation' => true`

```php
$form = ActiveForm::begin([
    'id' => 'contact-form',
    'enableAjaxValidation' => true,
]);
```


ใน Action ที่เราจะทำการใช้งาน ให้เพิ่มโค้ดสวนนี้เข้าไปหลังบันทัดที่ new ObjectModel แล้ว จริงๆ ก็เหมือนกัน yii1 แหละ แต่ yii2 เค้าไม่ได้ใส่เป็นค่า default ไว้ให้

```php
if (Yii::$app->request->isAjax && $model->load(Yii::$app->request->post())) {
    Yii::$app->response->format = Response::FORMAT_JSON;
    return ActiveForm::validate($model);
}
```
สามารถใช้งานได้ทุก action ทั้ง create,update

> หลักงานทำงานของ ajax validation form มันจะทำการส่ง request มาที่ action  นี้ เป็น ajax แล้วทำการ validate ข้อมูลที่ส่งมา ถ้ามี error ก็ส่งกลับไปเป็น json เมื่อ front ได้ร้ับก็จะแสดงผลต่อเป็น error message ให้เราเห็นอีกที

หลังจากนั้นก็ลองเปิดใช้งานดู กด tabๆ ดูก็ได้ฟิวด์ใหน required ไว้มันจะขึ้นเป็น error สีแดงให้เลย


[อ่านเพิ่มเติมได้ที่นี่](http://www.yiiframework.com/doc-2.0/guide-input-validation.html#ajax-validation)