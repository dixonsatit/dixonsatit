---
layout : post
title : รวมคำสั่งที่ใช้งานบ่อยๆ
---

รวบรวมคำสั่งที่จำเป็นและใช้งานบ่อยๆ


# getRoute()

```
Yii::$app->controller->getRoute();
```

ดึงค่าข้อมูล route ปัจจุบันที่ใช้งานอยู่ อย่างเช่น ตอนนี้ใช้งานอยู่ที่ BlogController action index ก็จะได้ค่าออกมา `blog/index`

# การรับค่า GET,POST

การเรียกค่า parameter จาก $\_GET, $\_POST ปกติ php จะใช้ $\_GET['parameterName'], $\_POST['parameterName'] ใน yii2 นั้นมีวิธีเรียกใช้งานดังนี้

การเรียกใช้งาน

```php
Yii::$app->request->get('id');
Yii::$app->request->post('id');
```
สามารถกำหนดค่า default ได้ด้วยในกรณีที่รับแล้วไม่มีค่า

```php
Yii::$app->request->get('id',0);
Yii::$app->request->post('id',9);
```
> หากไม่ได้หนดค่า default แล้วค่าที่ได้ในกรณีที่รับแล้วไม่มีค่า จะเป็น null
