---
layout : post
topic : รวม Tip & Trick การใช้งานต่างๆ
---
![](/img/thumbnail/tip-and-trick.jpg)

ผมจะรวบรวม tip & trick ในการใช้งาน Yii Framework 2.0 ง่ายๆ รวมไว้ที่นี่ หรือใครมี tip & trick ก็แนะนำมาผ่าน comment มาได้นะครับเดี่ยวผมจะนำมาเขียนไว้ให้ได้อ่านกัน

## การเพิ่ม link ใน GridView ที่ใช้ Pjax

ปกติเราสามารถเพิ่ม link ใน columns ได้อยู่แล้ว แต่ถ้าหากเราใช้งาน pjax แล้วละก็ link อะไรก็ตามที่อยู่ภายใต้ pjax จะถูกเรียกแบบ ajax ทั้งหมดอย่างเช่น เราอยากให้คลิก ที่แถวใน GridView แล้วให้เปิด link อีกแทบนึงไปเลยก็จะไม่สามารถทำได้จนกว่าเราจะสั่งให้ pjax รู้ว่า link นี้ไม่ต้องทำแบบ pjax นะ โดยกำหนดค่า properties `data-pjax = "0"` ใน tag a

```php
<?php echo GridView::widget([
      'dataProvider' => $dataProvider,
      'filterModel' => $searchModel,
      'columns' => [
        //....
            [
              'label'=>'เปิดแท็บใหม่',
              'format'=>'raw',
              'value'=>function($model){
                return Html::a('DEMO',['/site/index'],['target'=>'_blank',"data-pjax"=>"0"]);
              }
            ],
        //....
      ]
]); ?>
````

แค่นี้เราก็จะสามารถเปิด link ได้ตามปกติแล้วครับ
