---
layout : post
title : การใช้งาน JsExpression
---

![](/img/thumbnail/js-expression.jpg)

## JsExpression คืออะไร

JsExpression เป็น class ที่เอาไว้ใช้งานร่วมกับพวก widget ต่างๆ ที่จำเป็นจะต้องกำหนดค่าเป็นคำสั่ง javaScript  ให้กับ widget นั้น เพราะปกติพวก options ที่เป็นของ widget จะต้องกำหนดค่าผ่าน properties `clientOptions` และเวลาที่เรากำหนดค่าเข้าไปมันจะมองเป็น string ทั้งหมด ทำให้เราไม่สามารถเรียกใช้งานในลักษณะ function ของ javaScript ได้ เช่นการใช้งาน DatePicker

ถ้าเขียนเรียกใช้งาน DatePicker ด้วย jquery ปกติ รูปแบบที่ถูกต้องก็จะประมาณนี้

```js
jQuery('#group-name').datepicker({
  "defaultDate":"2014-01-01",
  "onSelect" : function(e){ alert(555); },
  "dateFormat":"yy-mm-dd"
});
```
ถ้าดูตามโค้ดด้านบน จะเห็นว่าเรากำหนดค่าเรียกใช้งาน event `onSelect` เพื่อจะดักจับ event ในตอนที่มีการคลิกเลือกวันที่ แล้วเราต้องการให้มันทำอะไรบางอย่าง ในตัวอย่างนี้ ผมให้มัน alert(555) ออกมา

แต่ในส่วนของ Yii 2 นั้นมี widget ให้ เมื่อเราเรียกใช้งานก็จะได้แบบนี้

```php
<?php
use yii\jui\DatePicker;
?>
<?= $form->field($model, 'name')->widget(DatePicker::className(),[
      'language' => 'th',
      'dateFormat' => 'yyyy-MM-dd',
      'clientOptions' => [
          'defaultDate' => '2014-01-01',
          'onSelect'=>'function(e){ alert("555"); }'
]]) ?>
```
เมือเวลาที่ทำงานจริง widget จะแปลงข้อความที่เรา config ไว้ไปเรียกใช้งานเป็นคำสั่ง javaScript ตามแต่ละ widget นั้นกำหนดซึ่ง DatePicker ก็จะเรียกแบบนี้

```js
jQuery('#group-name').datepicker({
  "defaultDate":"2014-01-01",
  "onSelect":"function(e){ alert(555); }",
  "dateFormat":"yy-mm-dd"
});
```

จะสังเกตว่า `"function(e){ alert(555); }"` มันมีเครื่องหมาย "" ครอบมันอยู่มันจึงเป็น string และไม่สามารถทำงานได้ ซึ่งความจริงมันควรเป็นแบบนี้

```js
jQuery('#group-name').datepicker({
  "defaultDate":"2014-01-01",
  "onSelect" : function(e){ alert(555); },
  "dateFormat":"yy-mm-dd"
});
```

จึงเป็นที่มาของ JsExpression เพื่อที่จะระบุว่าข้อความส่วนนี้ไม่ใช่ string แต่เป็น javaScript ซึ่งเราสามารถเรียกใช้งานได้ง่ายๆ แบบนี้  `new JsExpression('function(e){ alert(555); }')` ( อธิบายมาซะยาว! ^ ^ )


```php
<?php
use yii\jui\DatePicker;
use yii\web\JsExpression;
?>

<?= $form->field($model, 'name')->widget(DatePicker::className(),[
   'dateFormat' => 'yyyy-MM-dd',
  'clientOptions' => [
      'defaultDate' => '2014-01-01',
      'onSelect'=>new JsExpression('function(e){ alert(555); }')
]]) ?>
```

> ปกติค่า options ต่างๆ ที่เราตั้งค่ามันจะถูกแปลงค่าด้วย  yii\helpers\Json::encode() หรือ yii\helpers\Json::htmlEncode() เพื่อแปลงค่า options ของเราเป็น string แต่ถ้าส่วนใหนที่เราต้องการให้มันเป็น javaScript ก็แค่เรียกใช้งาน  JsExpression
