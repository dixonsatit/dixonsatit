---
layout : post
title : การใช้งาน DetailView
---

![](/img/detail-view.png)

 DetailView คือ Widget ที่เอาไว้แสดงผลข้อมูลได้ทีละแถว ดังนั้น ข้อมูลที่นำมาใช้งานกับ DetailView ต้องเป็น array 1 มิติ DetailView มี property ที่ต้องใช้บ่อยๆ คือ

 - `model` ค่า array ที่จะนำไปแสดงผล รับค่าเป็น array 1 มิติ
 - `attributes` เป็นตัวกำหนดชื่อคอลัมน์หรือชื่อคีย์ array ที่จะแสดงผล
 - `template` เป็นตัวที่กำหนด html ในส่วนแสดงผลในแต่ละคอลัมน์ ซึ่งเราสามารถปรับแต่งหรือเพิ่มเติม html เข้าไปได้
 - `options` เป็นตัวกำหนดค่า attributes ของ DetailView เช่นแท็ก class

## model & attributes

 ตัวอย่างเช่น เรียกข้อมูลขึ้นมา 1 แถว ซึงมันจะได้ array มา 1 มิติ เราถึงจะนำไปใช้งานกับ DetailView ได้

```php
 $model = Resume::findOne($id);
```
ที่ view

 ```php
use yii\widgets\DetailView;
 <?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'fullname',
        'education',
        'marital',
        'sex',
        'skill'
    ],
]) ?>
 ```

![](/img/detail-view-1.png)

 หรือเราจะกำหนด Array 1 มิติ เข้าไปตรงๆ เลยก็ได้

 ```php
 <?php
$items = [
  'id'=>'1',
  'title'=>'นาย',
  'firstname'=>'สาธิต',
  'lastname'=>'สีถาพล'
];
echo DetailView::widget([
    'model' => $items,
    'attributes' => [
      'id',
      'title',
      'firstname',
      'lastname',
    ],
]);
?>
 ```

 ![](/img/detail-view-2.png)

 > หากเราไม่กำหนด property `attributes` เข้าไปมันจะแสดงทุกคอลัมน์ที่มีใน array

## template

 เป็น property ที่เราสามารถปรับแต่งส่วนของการแสดงผลในแต่ละคอลัมน์ ซึ่งปกติค่า default ของ template จะเท่ากับ

  `<tr><th>{label}</th><td>{value}</td></tr>`

สิ่งที่อยู่ในเครื่องหมายปีกกาคือค่าที่จะนำมาแสดงผล `{label}` เป็นส่วนแสดงคำอธิบาย, `{value}` เป็นส่วนที่แสดงค่าของข้อมูล ซึ่งเราสามารถปรับแต่ง html เองได้หมด

เราจะทำการปรับปรุง template ของเดิม ด้วยการเพิ่ม icon เข้าไป

```php
<?= DetailView::widget([
    'model' => $model,
    'template'=>'<tr><th>{label}</th><td><i class="glyphicon glyphicon-info-sign"></i></i> {value}</td></tr>',
    'attributes' => [
      'fullname',
      'education',
      'marital',
      'sex',
      'skill'
    ],
]) ?>
```

 ![](/img/detail-view-icon.png)

## Shortcut Format

DetailView นั้นยังสามารถกำหนด format ได้เหมือนกับ gridview เลย
`shortcut format` รูปแบบจะเป็นดังนี้ `"attribute:format:label"` ปกติเราจะใส่แต่ชื่อคอลัมน์เข้าไปแต่ `shortcut format`เราสามารถกำหนดการแสดงผลได้ ว่าจะให้ออกมา format อะไร และยังกำหนด label ในตรงนี้ได้อีกด้วย แค่คั้นด้วยเครื่องหมาย `:`

- `attribute` คือชื่อคอลัมน์ที่ต้องการแสดงผล
- `format` คือข้อมูลที่เราต้องการให้แสดงผลออกมาในรูปแบบที่ต้องการ เช่น เราเก็บข้อมูลเป็นวันที่ `2015-06-01` เราสามารถกำหนด format เป็น `date` ได้โดยผลลัพจะออกมาเป็น `Jun 24, 2015` ได้ซึ่งมีหลาย format มาก เช่น `date`,`dateTime`,`email`,`time`,`text`,`html` ไปเลือกใช้งานกันเองได้[ที่นี่](http://www.yiiframework.com/doc-2.0/yii-i18n-formatter.html)
- `label` เป็นข้อความของคำอธิบายฟิวด์ที่เราต้องการแสดงผลให้เปลี่ยนจากของเดิมที่กำหนดไว้ใน `attributesLabel()` ใน Model และตรงนี้เป็น option จะระบุหรือไม่ก็ได้

```php
<?= DetailView::widget([
   'model' => $model,
   'attributes' => [
       'created_at:date:วันที่สร้าง', // แสดงผลวันที่
       'updated_at:dateTime', // แสดงผลวันที่และเวลา
       'email:email', // แสดงผลเป็น link MailTo
       'website:url',  // แสดงผล link url
       'is_admin:boolean', // แสดงเป็น Yes,No
       'payroll:decimal', // แสดงผลเป็นทศนิยม
       'content:html',// แสดงผลเป็น Html
       'photo:image',// แสดงผลเป็นรูปภาพ
       'age:integer',// แสดงผลเป็นจำนวนเต็ม
       'vat:percent',// แสดงผลเป็น %
       'content:text',// แสดงผลข้อความ text
       'created_at:time',// แสดงผลเป็นเวลา
       'file_size:size',// แสดงผลเป็นหน่วยของขนาดเช่น 12 kilobytes ค่าข้อมูลต้องเป็น bytes
       'file_size:shortSize',// แสดงผลเป็นหน่วยของขนาดเช่น 12 KB ค่าข้อมูลต้องเป็น bytes
       'created_at:relativeTime'// แสดงผลเป็นเวลาที่ผ่านมาแล้ว
   ],
]) ?>

```
## การปรับแต่งค่าที่จะแสดงผล

เราสามารถกำหนดค่าเข้าไปตรงๆ ได้ โดยไม่ต้องประกาศฟังก์ชัน Anonymous functions เหมือน gridview เพรามันเป็น array 1 มิติ สามารถกำหนดค่าเข้าไปตรงๆ ได้เลย

```php
<?= DetailView::widget([
   'model' => $model,
   'attributes' => [
    [  
        'label' => 'fullname',
        'value' => $model->title.$model->firstname.' '.$model->lastname,
    ],
    [
        'format'=>'html',
        'label' => 'color',
        'value' => '<span style="color:green;">'.$model->educationName.'</span>'
    ],
     'fullname',
     'education',
     'marital',
     'sex',
     'skill'
   ],
]) ?>

```

DetailView เป็น widget ที่ง่ายๆ ไม่ซับซ้อนอะไร ลองใช้กันดูครับ

@dixonSatit
