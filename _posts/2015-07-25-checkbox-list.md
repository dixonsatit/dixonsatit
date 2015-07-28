---
layout : post
title : การใช้งาน CheckBoxList และการบันทึกข้อมูล
---

![](/img/item-alias-flow.png)

CheckBoxList เป็นตัวเลือกที่เลือกได้มากกว่า 1 รายการ การเก็บข้อมูลจะแตกต่างจากฟิวด์ปกติ ที่เก็บเป็นค่าๆ เดียว ดังนั้นการจะใช้ CheckBoxList จะต้องมีขั้นตอนในการบันทึกที่ไม่เหมือนตัวอื่น

มันจะมีลำดับดังนี้

> บทความนี้ผมแยกมาจาก [การเตรียมข้อมูล Array เพื่อใช้กับ DropdownList,RadioList,CheckBoxList](/2015/07/21/item-alias.html) ตัดออกมาเพราะเนื้อหาค่อยข้างเยอะ

- ในขั้นตอนแรก CheckBoxLis จะ render checkbox ตาม array ที่เราได้มาจาก `$model->getItemSkill()`   และนำมาแสดงผลให้เราสามารถเลือก `skill` ได้
- เมือมีการ submit เพื่อบันทึกข้อมูล skill จะมีค่าเป็น array ดังนั้นเราจะต้องทำการแปลงจาก array เป็น string โดยใช้ฟังก์ชัน `implode`  ให้อยู่ในรูปแบบ `jquery,php,js,css` นี้ จากนั้นนำไปบันทึกข้อมูล
- เมื่อจะทำการแก้ไขข้อมูล เราก็จะต้องแปลงค่า  `jquery,php,js,css` ที่เก็บในฟิวด์ skill ให้กลับไปเป็น array เหมือนเดิมเพื่อให้ฟอร์มแสดงผลว่าเราได้เลือกอะไรไปบ้าง โดยใช้ `explode` เพื่อแปลง string กลับเป็น array จะเป็นแบบนี้

![](/img/item-alias-checkbox.png)

มันก็จะวลลูปอยู่แบบนี้ เอาล่ะคงพอเข้าใจการทำงานแล้ว เราลองมาดูในส่วนของโค้ดกัน


## Form
ในส่วนฟอร์ม ก็จะเรียกใช้งาน CheckBoxList

ถ้าหากเป็น array ที่กำหนดค่าเอง ก็ระบุเข้าไปตรงๆได้ แต่ไม่อยากแนะนำให้ใช้แบบนี้

```
  <?= $form->field($model, 'skill')->checkBoxList(['php'=>'PHP','jquery'=>'JQuery']) ?>
```

อยากแนะนำให้สร้างเป็น itemAlias() ใน model ไว้แล้วค่อยเรียกใช้งานจะดีกว่า [ดูตัวอย่างการสร้าง itemAlias ได้ที่นี่](/2015/07/21/item-alias.html)

```php
<?= $form->field($model, 'skill')->checkBoxList($model->getItemSkill()) ?>
```

แต่ถ้าหากเรียกจากข้อมูลจาก model ก็สามารถเรียกได้แบบนี้ ในกรณีที่เราเก็บข้อมูลไว้ที่ตาราง เราจะใช้ `ArrayHelper::map()`  เพื่อทำการระบุฟิวด์ที่เป็นโค้ดและฟิวด์ที่ต้องการแสดงเป็นข้อความ ให้มันเลือกข้อมูลออกมาให้อยู่ในรูปแบบ  array 1 มิติ ที่มี key กับ value

```php
[
  'php'=>'PHP',
  'jquery'=>'JQuery'
]
```

เมือเรียกใช้งานร่วมกับ checkBoxList ก็จะได้แบบนี้

```php
<?php
use yii\helpers\ArrayHelper;
use app\models\Skill;
?>
<?= $form->field($model, 'skill')->checkBoxList(ArrayHelper::map(Skill::find()->all(),'id','name')) ?>
```
จากนั้นเราก็จะสามารถแสดง ฟอร์มเป็น checkBoxList ได้แล้ว

## Model


ส่วนนี้เป็นส่วนที่เราจะดักจับ event และทำการแปลงค่าให้ได้ตามที่เราต้องการ โดยเพิ่มเข้าไปที่ไฟล์ model ของเรา และทำการระบุ Validation ใน `roles()` ให้กับฟิวด์ที่จะใช้งานเป็น `safe` ด้วย

```php
<?php
//...

public function rules()
    {
        return [
            // .....
            [['skill'], 'safe'],
        ];
    }

public function behaviors()
{
    return [
        [
            'class' => AttributeBehavior::className(),
            'attributes' => [
                ActiveRecord::EVENT_BEFORE_INSERT => 'skill',
                ActiveRecord::EVENT_BEFORE_UPDATE => 'skill',
            ],
            'value' => function ($event) {
                return implode(',', $this->skill);
            },
        ],
    ];
}
```

`AttributeBehavior` คือการดักจับ event ต่างๆ ของ model เช่น EVENT_BEFORE_INSERT,EVENT_BEFORE_UPDATE เพื่อที่เราจะทำงานบางอย่าง กับฟิวด์ที่เราต้องการ ดูตัวอย่าง

ตัวอย่างนี้เราต้องการแปลงข้อมูล array ที่ได้จาก CheckBoxList ให้เป็น string ก่อนบันทึกและแก้ไขข้อมูล ซึงเราจะทำการเรียกใช้ฟังก์ชัน `behaviors()`  ของ model ที่เราใช้งานและเพิ่มฟังกชันนี้เข้าไป

จากโค้ดจะเห็นว่าเราทำการเรียกใช้งาน `AttributeBehavior` ในฟังก์ชัน `behaviors()` และกำหนดคุณสมบัติเข้าไปดังนี้

- `class` คือชื่อของ behavior ที่เราต้องการใช้ ในตัวอย่างนี้คือ AttributeBehavior
- `attributes` เป็นการระบุว่าเราจะกระทำข้อมูลในเหตุการณ์อะไร และกับฟิวด์อะไร ซึ่งเราระบุเหตุการณ์ before-insert,before-update และระบุฟิวด์คือ skill
- `value` คือส่วนที่เราจะทำงานบางอย่างเพื่อให้ค่ากับ skill ซึ่งในตัวอย่างเราจะทำการแปลงข้อมูลเดิมที่เป็น array ให้เป็น string โดยใช้ `implode()`


หลังจากนั้น เมื่อเราลองบันทึกข้อมูล ก็จะพบข้อมูลในลักษณะนี้ `php,jquery,`  เก็บใน table

## เพิ่มฟังก์ชันแปลง string ให้เป็น array

ในตอนที่เราจะแก้ไขข้อมูล ข้อมูลที่มาจาก table จะเป็น string แบบนี้ `php,jquery,` เมื่อนำมาใช้กับ checkBoxList มันจะไม่สามารถใช้งานได้ เราต้องทำการแปลงค่าจาก string เป็น array ซะก่อน

เราจะทำการสร้างฟังชันเพื่อแปลง string ในฟิวด์ skill เป็น array เพิ่มเข้าไปที่ model ที่เราใช้งาน

```php
public function skillToArray()
{
  return $this->skill = explode(',', $this->skill);
}
```

หลังจากนั้น เรียกใช้งานที่ฟังก์ชัน update() ใน ResumeController.php

```php
<?php

// ...

public function actionUpdate($id)
{
    $model = $this->findModel($id);
    $model->skillToArray(); //<<------------------

    if ($model->load(Yii::$app->request->post()) && $model->save()) {
        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('update', [
            'model' => $model,
        ]);
    }
}
```

จากนั้นฟอร์มก็จะสามารถแสดงข้อมูลที่เราเคยบันทึกไปแล้วโดยจะทำการเลือกในส่วนที่เราเคยบันทึกไว้

## การแสดงผลใน Griview หรือ DetailView

เพิ่มฟังก์ชั่นนี้เข้าไปใน model ของเรา ในกรณีที่สร้างเป็น itemAlias() เป็น array เก็บไว้เองใน model

```php
public function getSkillName(){
    $skills = $this->getItemSkill();
    $skillSelected = explode(',', $this->skill);
    $skillSelectedName = [];
    foreach ($skills as $key => $skillName) {
      foreach ($skillSelected as $skillKey) {
        if($key === $skillKey){
          $skillSelectedName[] = $skillName;
        }
      }
    }

    return implode(', ', $skillSelectedName);
}
```

กรณีถ้าใช้จาก Modlel ให้ใช้แบบนี้

```php
<?php

use yii\helpers\ArrayHelper;
use app\models\Skill;

// ...

public function getSkillName(){
    $skills = ArrayHelper::map(Skill::find()->all(),'id','name');
    $skillSelected = explode(',', $this->skill);
    $skillSelectedName = [];
    foreach ($skills as $key => $skillName) {
      foreach ($skillSelected as $skillKey) {

        if($key === (int)$skillKey){
          $skillSelectedName[] = $skillName;
        }
      }
    }

    return implode(', ', $skillSelectedName);
}
```

การเรียกใช้งานใน GridView

```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    'columns' => [
        ['class' => 'yii\grid\SerialColumn'],
        // 'id',
        // 'title',
        // 'firstname',
        // 'lastname',
        // 'education_level',
        // 'marital_status',
        // 'sex',
        // 'skill',

        // virtual attribute
        'fullName',
        'educationName',
        'maritalName',
        'sexName',
        'skillName',//<----

        ['class' => 'yii\grid\ActionColumn'],
    ],
]); ?>
```

![](/img/item-alias-gridview-1.png)

เรียกใช้งานใน DetailView

```php
<?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'id',
        'title',
        'firstname',
        'lastname',
        'education_level',
        'marital_status',
        'sex',
        'skill',
        // virtual attribute
        'fullName',
        'educationName',
        'maritalName',
        'sexName',
        'skillName' //<----
    ],
]) ?>
```

![](/img/item-alias-detailview-1.png)


จากนั้นเราก็จะสามารถ เพิ่มลบแก้ไขและแสดงผลข้อมูลจาก CheckBoxLis ได้ บทความนี้ผมแยกมาจาก [การเตรียมข้อมูล Array เพื่อใช้กับ DropdownList,RadioList,CheckBoxList](/2015/07/21/item-alias.html) เพื่อให้ดูง่ายขึ้น

@dixonSatit
