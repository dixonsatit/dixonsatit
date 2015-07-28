---
layout : post
title : การใช้งาน CheckBoxColumn สำหรับการลบทีละหลายๆ Record
---

![](/img/thumbnail/checkbox-column.jpg)

`CheckBoxColumn` เป็นตัวเลือก checkbox ที่แสดงเป็นคอลัมน์ภายใน GridView เพื่อให้สามารถเลือกรายการได้ทีละหลายๆ แถว  ซึ่งก็แล้วแต่ว่าเราจะนำไปใช้ทำอะไร วันนี้เราจะลองใช้ `CheckBoxColumn` เพื่อทำการลบแถวที่ต้องการทีละหลายๆ แถว ซึ่งทำได้ง่ายมากๆ  ไปดูตัวอย่างกัน


## เรียกใช้งานใน GridView

เราจะต้องทำการเรียกใช้งานตัว `CheckBoxColumn` ใน column ของ GridView เสียก่อน สามารถทำได้ง่ายมาก เพียงเรียก class `['class' => 'yii\grid\CheckboxColumn']` ใน column

```php
<?= GridView::widget([
      'dataProvider' => $dataProvider,
      'filterModel' => $searchModel,
      'columns' => [
          ['class' => 'yii\grid\SerialColumn'],
          ['class' => 'yii\grid\CheckboxColumn'], //<------
          'title',
          'firstname',
          'lastname',
          ['class' => 'yii\grid\ActionColumn'],
      ],
  ]); ?>
```

ในกรณีที่เราอยากกำหนดค่าต่างๆ เพิ่มเติมก็สามารถทำได้ดังนี้

- `checkboxOptions` กำหนด attributes ต่างๆ ของ html ได้เช่น class,style
- `multiple` กำหนดให้สามารถเลือกได้ทีละหลายๆ ตัว ค่า default จะเท่ากับ `true`
- `name` ชื่อของตัว checkBox ค่า default จะเท่ากับ `selection`

```php
<?= GridView::widget([
      'dataProvider' => $dataProvider,
      'filterModel' => $searchModel,
      'columns' => [
          ['class' => 'yii\grid\SerialColumn'],
          ['class' => 'yii\grid\CheckboxColumn',[
            'checkboxOptions'=>[],
            'multiple'=>true,
            'name'=>'selection'
          ]],
          'title',
          'firstname',
          'lastname',
          ['class' => 'yii\grid\ActionColumn'],
      ],
  ]); ?>
```

ดู properties ที่สามารถใช้งานได้[ที่นี่](http://www.yiiframework.com/doc-2.0/yii-grid-checkboxcolumn.html)

เมื่อตั้งค่าเสร็จจะเห็น CheckboxColumn ขึ้นมาแบบนี้

![](/img/check-box-column.png)

## การเรียกใช้งานค่าที่ Selection

เมื่อเราทำการเลือกรายการที่ต้องการแล้ว การที่จะดึงรายการที่เราเลือกนั้น สามารถทำได้โดยผ่าน JQuery และระบุ id ของ GridView ของเราเข้าไป จะได้ค่ากลับมาเป็น array pk ที่เราเลือก เช่น `[2,3,1]`

```
var keys = $('#grid').yiiGridView('getSelectedRows');
```
ซึ่งก็แล้วแต่ว่าเราจะนำไปทำอะไรต่อ


## ใช้ CheckBoxColumn เพื่อลบข้อมูลทีละหลายๆ แถว

- ขั้นตอนแรกทำการเรียกใช้งาน `CheckBoxColumn` ใน GridView ที่เราต้องการ

```php
// ...

'columns' => [
    //....
    ['class' => 'yii\grid\CheckboxColumn'],
  // ....
],

// ...
```

- เพิ่ม ปุ่มเพื่อทำการคลิกลบข้อมูล ด้านบน GridView ตั้งค่า id = 'btn-delete'

```php
<p>
    <?= Html::a(Yii::t('app', 'Create Resume'), ['create'], ['class' => 'btn btn-success']) ?>
    <?= Html::button(Yii::t('app', 'Delete'), ['class' => 'btn btn-warning pull-right','id'=>'btn-delete']) ?>
</p>
```

![](/img/check-box-column-delete.png)

- จากนั้นเราจะทำการใช้ jQuery `yiiGridView` เพื่อดึงรายการที่เราเลือกไว้ส่งไปให้ action `delete-all` ใน controller เพื่อทำการลบข้อมูล เพิ่มไว้ด้านล่าง GridView ที่เราใช้งาน

```php
<?php
use yii\helpers\Url;
// .....
$this->registerJs('
  jQuery("#btn-delete").click(function(){
    var keys = $("#w0").yiiGridView("getSelectedRows");
    //console.log(keys);
    if(keys.length>0){
      jQuery.post("'.Url::to(['delete-all']).'",{ids:keys.join()},function(){

      });
    }
  });
');
 ?>
```

การทำงาน

- `var keys = $("#w0").yiiGridView("getSelectedRows");` เราจะเรียกค่าที่ selection มาเก็บไว้ที่ตัวแปร keys โดยเรียกไปที่ GridView ที่ id = `w0` ซึ่งก็แล้วแต่เราจะตั้งอะไร
- ทำการ if เพื่อตรวจสอบค่าว่า มีการเลือกรายการหรือไม่ ถ้ามีให้ส่งค่าไปลบ
- เมื่อมีรายการที่เลือก ก็จะนำ array ที่ได้ ใช้ function `join()` ของ javaScript เพื่อแปลง array ให้เป็น string ในรูปแบบ `1,4,3`
- จากนั้นใช้ JQuery.post() เพื่อทำการส่งค่า ids ที่เป็นค่า selection ที่แปลงเป็น string แล้ว ส่งค่าไปลบข้อมูล

ในส่วนของ front ก็เพียงแค่ส่งค่าตัวแปรไปให้ จากนั้นเราจะทำการสร้าง action เพื่อรองรับการลบข้อมูลทีละหลายๆ แถว

## สร้าง ActionDeleteAll() เพื่อทำการลบข้อมูลทีละหลายๆ แถว

ใน controller เราจะทำการเพิ่ม function ActionDeleteAll() เข้าไป ดังนี้

```php
<?php

// ...

public function behaviors()
{
    return [
        'verbs' => [
            'class' => VerbFilter::className(),
            'actions' => [
                'delete' => ['post'],
                'delete-all' => ['post'],
            ],
        ],
    ];
}

public function actionDeleteAll(){
    $delete_ids = explode(',', Yii::$app->request->post('ids'));
    Resume::deleteAll(['in','id',$delete_ids]);
    return $this->redirect(['index']);
}
```

ในส่วน behaviors() เราต้องควบคุม actionDeleteAll() ให้สามารถส่งข้อมูลมาลบด้วย method POST เท่านั้นเพื่อความปลอดภัย

ส่วน actionDeleteAll() เราจะทำการสร้างขึ้นมาใหม่เพราะของเดิมสามารถลบได้เพียงครั้งละ 1 Record

- เราจะทำการรับค่าตัวแปรที่ส่งมาเป็น POST ชื่อว่า `ids` ด้วย `Yii::$app->request->post('ids')` - -
- จากนั้นทำการแปลง string ให้เป็น array เก็บไว้ที่ตัวแปร `$delete_ids`
- และใช้ function deleteAll และใส่เงื่อนไขเป็น in ตัวนี้จะมีค่าเท่ากับ `DELETE FROM `resume` WHERE `id` IN (1, 2, 3) ที่ใช้ in เพราะจะได้ไม่ต้องไปวลลูปเพื่อลบ ใช้ in ทีเดียวจบเลย
- จากนั้น redirect ไปที่ action index เหมือนเดิม

จากนั้นทดลองทำการลบข้อมูลดู  หวังว่าคงจะเป็นประโยชน์ไม่มากก็น้อยครับ ติดปัญหาอะไร comment ไว้ด้านล่างเลยครับ

@dixonSatit
