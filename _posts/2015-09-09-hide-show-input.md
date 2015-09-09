---
layout : post
title : การซ่อนแสดง Input ตามเงื่อนไข
---

ในกรณีที่เราสร้างฟอร์มแล้วมีตัวเลือกบางตัวที่เราต้องการให้แสดงก็ต่อเมื่อมีการเลือกตัวเลือกนั้นเช่น เรามีฟอร์มกรอกอาชีพ แพทย์, พยาบาล, อื่นๆ ถ้าหากเลือก อื่นๆ เราจะให้แสดงช่องกรอกข้อมูลเพิ่มเติม

ในกรณียังไม่ได้เลือก หรือเปิดขึ้นมาครั้งแรก

![](/img/input-hide.png)

ในกรณีที่เลือกจะแสดงช่องกรอกข้อมูล

![](/img/input-show.png)

หลักการนั้นง่ายมากๆ ครับ ถ้าใครเคยเขียน JQuery มาบ้างน่าจะเข้าใจง่าย เราจะใช้ฟังก์ชัน JQuery hide(),show() เพื่อซ่อนและแสดง input ตามเงื่อนไขของเรา

ให้เราสร้างโค้ดนี้ที่ view ที่ต้องการเช่น \_form.php

```php
<?php
$this->registerJs("
  var input1 = 'input[name=\"Blog[category]\"]';
  setHideInput(3,$(input1).val(),'.field-blog-tag');
  $(input1).click(function(val){
    setHideInput(3,$(this).val(),'.field-blog-tag');
  });


  function setHideInput(set,value,objTarget)
  {
    console.log(set+'='+value);
      if(set==value)
      {
        $(objTarget).show(500);
      }
      else
      {
        $(objTarget).hide(500);
      }
  }
");
 ?>
```

ฟังก์ชัน setHideInput() นี้สร้างขึ้นมาเพื่อรับค่าตามเงื่อนไขแล้วทำการซ่อนหรือแสดง input ที่เราต้องการจะมี 3 parameter ตามลำดับ

- set คือค่าที่เรากำหนดเพื่อนำไปเปรียบเทียบตามเงื่อนไข
- value ค่าที่ได้จาก input  นั้นๆ ที่เราต้องการตรวจสอบ
- objTarget คือ  tag ที่เราต้องการจะซ่อนหรือแสดง

ส่วนแรกเราจะประกาศค่า input เพื่อนำไปใช้ในการตรวจสอบโดยเรียกใช้งานแบบ selector ระบุ input ที่มี attribute `name` = `Blog[category]` ซึ่ง `Blog` คือชื่อ model, `category` คือชื่อฟิวด์ ง่ายๆ คลิกขวา inspect element ใน browser ดูได้ครับ

```js
var input1 = 'input[name=\"Blog[category]\"]';
```

การดึงค่าใช้คำสั่ง val() ของ JQuery

```js
$(input1).val();
```

เราจะต้องทำการกำหนดค่าก่อนว่าเราจะดักจับเมื่อค่าเท่ากับอะไร ในตัวอย่างนี้ ค่าอาชีพ = 3 จะให้แสดงช่องกรอกข้อมูลเพิ่มเติม ซึ่งการเรียกใช้งานมันจะมี 2 เหตการณ์

เหตุการณ์แรกคือ เมื่อเปิดฟอร์มขึ้นมาไม่ว่าจะเป็น create,update

```
  setHideInput(3,$(input1).val(),'.field-blog-tag');
```

เหตุการณ์ที่สองเมื่อมีการคลิกจะใช้ส่วนนี้ดังจับ event click ของ input นั้นๆ

```
$(input1).click(function(val){
  setHideInput(3,$(this).val(),'.field-blog-tag');
});
```

ในส่วน element หรือ tag ที่เราต้องการจะซ่อนนั้นสามารถดูได้ง่ายๆ เพียงคลิก inspect element นั้นที่ต้องการจะซ่อน

![](/img/input-inspect.png)

จากนั้นเลื่อนขึ้นนิดนึงมันจะมี tag ครอบ input อยู่และจะมีชื่อ class ของ css กำกับตามชื่อฟิวด์นั้นๆ เราสามารถนำชื่อนี้มาเป็นตัวอ้างอิงในการซ่อนแสดงได้ ในตัวอย่างนี้คือ `field-blog-tag`

![](/img/input-inspect-tag.png)

เมื่อลองคลิกค่าเมื่อเลือก อื่นๆ จะแสดงช่องกรอกข้อมูลให้ แต่ถ้าหารเลือกอาชีพอื่นๆ จะซ่อนไว้

ในกรณีที่ต้องการใช้หลายๆ ฟิวด์ก็แค่สร้างชึ้นมาอีกชุด แต่ใช้ฟังก์ชันเดียวกัน

```
var input1 = 'input[name=\"Blog[category1]\"]';
  setHideInput(3,$(input1).val(),'.field-blog-tag');
  $(input1).click(function(val){
    setHideInput(3,$(this).val(),'.field-blog-tag');
  });

var input2 = 'input[name=\"Blog[category2]\"]';
  setHideInput(1,$(input2).val(),'.field-blog-other');
  $(input2).click(function(val){
    setHideInput(1,$(this).val(),'.field-blog-other');
  });

```

เป็นตัวอย่างง่ายๆ นะครับ ลองใช้กันดู

> ชื่อฟิวด์ในตัวอย่างอาจจะไม่ตรงกับความเป็นจริงซักเท่าไหร่ผมทำตัวอย่างจากฟอร์มเดิมๆ ครับไม่ต้องสนใจนะ

@dixonsatit
