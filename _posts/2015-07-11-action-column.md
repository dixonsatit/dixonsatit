---
layout : post
title : การปรับแต่ง Action Column ใน GridView
---

![Action Column](/img/thumbnail/action-column.jpg)

ActionColumn เป็น column ที่ใช้ใน `yii\grid\GridView` เพื่อแสดงปุ่มสำหรับควบคุมการทำงานใน column โดยจะแสดงผลในทุกๆ แถวของข้อมูล โดยมีปุ่ม default อยู่ 3 ปุ่มคือ `view`,`update`,`delete`

เราสามารถจัดการ เพิ่มปุ่มเอง,ซ่อน,แสดงปุ่มที่มีอยู่ได้  หรือปรับแต่งตามที่เราต้องการได้

## Properties พื้นฐานที่ควรรู้
- `Options` เป็นการกำหนดคำสั่ง html attribute เข้าไปใน column เฉพาะส่วน `colgroup` ของ table เช่น class,style หรือ attribute อื่นๆ ของ html ได้ `!ส่วนนี้ไม่เกี่ยวกับ content ที่แสดงผล`
- `contentOptions` เป็นการกำหนดคำสั่ง html attribute เข้าไปใน column ที่แสดงผล เช่น class,style หรือ attribute อื่นๆ ของ html ได้
- `template` กำหนด template การแสดงผลของค่าปุ่ม default หรือปุ่มที่สร้างขึ้นเอง ปกติ default จะเท่ากับ `'{view} {update} {delete}'` จะสลับตำแหน่ง เรียงใหม่,ลบ หรือเลือกแสดงเฉพาะปุ่มที่ต้องการได้ เช่นให้แสดงเฉพาะ `view` ก็ลบอันที่เหลือออก ให้เหลือ `'{view}'` แบบนี้
- `buttons` เป็นส่วนที่กำหนดการแสดงผลของปุ่ม `view`,`update`,`delete` หรือปุ่มอื่นๆ ที่เราสร้างขึ้น
- `buttonOptions` เป็นส่วนกำหนด class,css หรือ attribute ของ html ของ `buttons`
- `header` เป็นส่วนกำหนดข้อความ header ของ column
- `headerOptions` เป็นส่วนกำหนด class,css หรือ attribute ของ html ของ `header`

> ดูรายละเอียดเพิ่มเติมได้ที่นี่ [http://www.yiiframework.com/doc-2.0/yii-grid-actioncolumn.html](http://www.yiiframework.com/doc-2.0/yii-grid-actioncolumn.html)


หลักๆ ในการจัดการ ActionColumn จะมีแค่ 2 Properties ที่จำเป็นต้องใช้งานคือ `template`,`buttons`

### 1.template  

เป็น Properties ที่กำหนดว่าใน column นี้เราจะให้ปุ่มอะไรแสดงบ้าง ซึ่งปกติมันจะมี `view`,`update`,`delete` หากเราไม่ต้องการแสงดปุ่มใหนก็สามารถลบ ปุ่มที่ไม่ต้องการออกไปได้เลย เช่น หากคุณต้องการแสดงเฉพาะ `view` ก็สามารถตั้งค่าได้แบบนี้

```php
[
    'class' => 'yii\grid\ActionColumn',
    'header'=>'Action',
    'template'=>'{view}'
]
```
ก่อน

![](/img/all-button-action-column.png)

หลัง

![](/img/view-action-column.png)

> หากใช้ทั้ง 3 ปุ่มอยู่แล้ว ไม่ต้องกำหนดอะไรเพิ่ม ยกเว้นเราต้องการเพิ่มอะไรบางอย่างเข้าไป

เราสามารถเพิ่ม tag html เข้าไปใน template ได้ ยกตัวอย่างการเพิ่ม style Button groups เข้าไป
ดูรายละเอียด Button Groups ได้[ที่นี่](http://getbootstrap.com/components/#btn-groups) ซึ่งโครงสร้าง html จะเป็นแบบนี้

```html
<div class="btn-group" role="group" aria-label="...">
  <button type="button" class="btn btn-default">Left</button>
  <button type="button" class="btn btn-default">Middle</button>
  <button type="button" class="btn btn-default">Right</button>
</div>
```
![](/img/button-groups.png)

จะสังเกตได้ว่า Button Groups จะมี div ครอบอยู่ และใช้งาน class `btn-group` และในแต่ละปุ่มจะมี class = `btn btn-default` เราจึงสามารถปรับแต่งให้ ActionColumn ให้แสดงผลปุ่มแบบ Button Groups ได้แบบนี้

```php
[
  'class' => 'yii\grid\ActionColumn',
  'buttonOptions'=>['class'=>'btn btn-default'],
  'template'=>'<div class="btn-group btn-group-sm text-center" role="group"> {view} {update} {delete} </div>'
],
```
ก่อน

![](/img/all-button-action-column.png)

หลัง

![](/img/button-groups-action-column.png)



## 2. buttons

เป็น Properties ที่เอาไว้กำหนดค่าต่างๆ ให้ปุ่มที่เรากำหนด อาจจะใส่เงื่อนไขต่างๆ ซ่อนแสดงปุ่ม,แสดงข้อความ เป็นต้น

### การเพิ่มปุ่ม
ตัวอย่างนี้ เป็นการสร้างปุ่ม copy เพิ่มเข้าไปใน ActionColumn โดยเราจะต้องประกาศค่าปุ่มที่ template โดยเพิ่ม `{copy}`  เข้าไปเพื่อให้มันรู้ว่ามีปุ่มนี้อยู่

```
'template'=>'{copy} {view} {update} {delete}',
```

จากนั้นไปกำหนดค่าต่างๆ ของปุ่มที่ Properties `buttons` โดยใช้ค่าที่เราตั้งไว้ใน template คือ `copy` และกำหนดค่าต่างๆ ได้เองเลย โดย buttons จะมีค่า params มาให้ใช้ 3 ตัวคือ

- `$url` ระบบมันจะสร้างให้ตามชื่อปุ่มที่เราตั้งไว้ หาไม่ใช้ก็สามารถกำหนดเองได้
- `$model` ข้อมูลทั้งหมดของแถวนั้นๆ สามารถเรียกใช้งานได้ทุกฟิวด์ ที่มีในตาราง `$model->fieldName`
- `$key` คือค่า pk ของตารางสามารถนำไปใช้ได้เลย

```php
[
  'class' => 'yii\grid\ActionColumn',
  'template'=>'{copy} {view} {update} {delete}',
  'buttons'=>[
    'copy' => function($url,$model,$key){
        return Html::a('<i class="glyphicon glyphicon-duplicate"></i>',$url);
      }
    ]
],
```
![](/img/action-column-copy.png)

> ในส่วนของ url ระบบจะสร้างมาให้เองอัตโนมัติ โดยยึดจากชื่อ button ที่เราสร้าง เช่น ชื่อ button คือ copy ดังนั้นใน url จะเท่ากับ `r=employee/copy&id=1` โดยจะมองชือ button เป็นชื่อ action ให้เลยและแนบค่า pk ของตารางที่ใช้ไปให้เลย ในกรณีที่เราไม่ต้องการใช้งาน default Url ก็ใส่เป็นค่า array เข้าไปได้เลย  เช่น `
Html::a('<i class="glyphicon glyphicon-duplicate"></i>',['controller/action','id'=>$model->id])
`

## คอลัมน์แคบ
เมื่อดูตามรูปภาพด้านบนจะเห็นว่า ปุ่มยังไม่เรียงเป็นแนวนอนยังตกบรรทัดอยู่ สามารถเพิ่ม `contentOptions` และใส่ค่า `  'noWrap' => true` ได้ดังนี้

```html
[
  'class' => 'yii\grid\ActionColumn',
  'template'=>'{copy} {view} {update} {delete}',
  'contentOptions'=>[
    'noWrap' => true
  ],
  'buttons'=>[
    'copy' => function($url,$model,$key){
        return Html::a('<i class="glyphicon glyphicon-duplicate"></i>',$url);
      }
    ]
],
```

![](/img/action-column-nowarp.png)

> หากลองใช้วิธีนี้แล้วยังไม่ได้อีกให้ลอง เพิ่มความกว้างให้ column เข้าไป ดังนี้ `'options'=> [ 'style'=>'width:100px;'] ` ตัวนี้ได้ชัวๆ

```html
[
  'class' => 'yii\grid\ActionColumn',
  'template'=>'{copy} {view} {update} {delete}',
  'options'=> ['style'=>'width:100px;'],
  'contentOptions'=>[
    'noWrap' => true
  ],
  'buttons'=>[
    'copy' => function($url,$model,$key){
        return Html::a('<i class="glyphicon glyphicon-duplicate"></i>',$url);
      }
    ]
],
```

### การเพิ่มเงื่อนไขเพื่อซ่อนแสดงปุ่ม

เราสามารถเพิ่มเงื่อนไขเพื่อตรวจสอบค่าบางอย่าง แล้วซ่อนหรือแสดงได้ หรืออาจจะเป็นเงื่อนไขอื่นๆ ก็ได้
เช่นตัวอย่าง จะเช็คจากค่า status = 1 ให้แสดง, ถ้าไม่ใช้ให้ซ่อน

```php
[
  'class' => 'yii\grid\ActionColumn',
  'template'=>'{copy} {view} {update} {delete}',
  'buttons'=>[
    'copy' => function($url,$model,$key){
        return $model->status == 1 ? Html::a('<i class="glyphicon glyphicon-duplicate"></i>',$url) : null;
      }
    ]
],
```
ถ้า status = 1 คือแสดงปุ่ม

![](/img/action-column-nowarp.png)

ถ้า status ไม่เท่ากับ (!=) 1 คือซ่อนปุ่ม

![](/img/all-button-action-column.png)


### โค้ดการเพิ่ม style Button Groups ให้กับปุ่มเดิมๆ ของเรา

เราสามารถเพิ่ม Button Groups เข้าไปได้ และจะทำให้ปุ่มของเราดูชัดเจนขึ้น, มองง่ายขึ้น, ดูสวยขึ้น

```html
[
  'class' => 'yii\grid\ActionColumn',
  'buttonOptions'=>['class'=>'btn btn-default'],
  'template'=>'<div class="btn-group btn-group-sm text-center" role="group">{copy} {view} {update} {delete} </div>',
  'options'=> ['style'=>'width:150px;'],
  'buttons'=>[
    'copy' => function($url,$model,$key){
        return Html::a('<i class="glyphicon glyphicon-duplicate"></i>',$url,['class'=>'btn btn-default']);
      }
    ]
],
```

![](/img/button-groups-copy.png)

## หากใช้ Pjax
ถ้าหากใช้ pjax ด้วยตรง link ต้องใส่ options  `'data-pjax' => '0'` ด้วยเพื่อให้ pjax ไม่ทำงานกับ link นี้ให้เป็น link url ปกติ

```php
   Html::a('<i class="glyphicon glyphicon-duplicate"></i>',['/search/index'],['data-pjax' => '0']);
```

## สรุป ActionColumn

 Yii 2 ค่อนข้างเปิดกว้างให้เราได้ปรับแต่งส่วนต่างๆ ได้เองทังหมด ในส่วน ActionColumn ก็เหมือนกัน ลองใช้งานและปรับแต่งดูครับ หากติดปัญหาก็คอมเม้นด้านล่างได้เลยครับ ^ ^
