---
layout : post
title : การเตรียมข้อมูล Array เพื่อใช้กับ DropdownList,RadioList,CheckBoxList
---

![](/img/thumbnail/item-alias.jpg)

ในการใช้งานฟอร์มส่วนใหญ่ หากมีข้อมูลเพียงเล็กน้อยที่ต้องใช้งานในฟอร์มเพื่อแสดงผล DropdownList,RadioList,CheckBoxList  และข้อมูลส่วนนี้ ไม่ค่อยมีการเปลี่ยนแปลงอะไรมากมาย ผมจะสร้างเป็น function ที่เก็บค่า array  ไว้ เพื่อเรียกใช้งานในตัว Model เลย

> ตัวอย่างนี้เป็นการกำหนดค่า array เข้าไปใน model เลย ซึ่งมันจะเหมาะกับข้อมูลที่ไม่ได้มีการเปลี่ยนแปลงบ่อยๆ ข้อมูลไม่เยอะ ไม่จำเป็นต้องสร้าง table เพิ่มเติม

ผมเองเชื่อว่าหลายๆ คนคงเจอปัญหาในการใช้งาน array เพราะเมื่อจะใช้ที ก็ไปเขียน array ไว้ที่ form หรือที่ Gridviw มันไม่สะดวก เวลาที่ต้องแก้ไข ก็ไปตามที่จุดนั้นๆ วุ่นวาย เห็นหลายๆ คนชอบเขียนแบบนี้จึงอยากจะแนะนำวิธีการสร้าง array เก็บเป็น `itemsAlias()` ฟังก์ชัน เพื่อให้สามารถใช้งานได้สะดวกๆ และเป็นรูปแบบที่มาตรฐาน ไม่ว่าจะเป็นการนำไปใช้งานกับ  DropdownList,RadioList,CheckBoxLis หรือนำเป็นแสดงผลในที่ต่างๆ GridView,DetailView,Listview

รูปแบบที่แนะนำนี้จะดีทั้งกับตัวผู้พัฒนาเอง และยังดีกับผู้ร่วมพัฒนาด้วยเพราะสามารถอ่านโค้ดได้ง่ายและเข้าใจ ไม่ต้องถามกันให้วุ่นวาย ลองดูกันครับ

## เตรียมโครงสร้างข้อมูลกันก่อน

ในตัวอย่างนี้เราจะทำการสร้างฟอร์มกรอกข้อมูล Resume แบบง่ายๆ และใช้ DropdownList,RadioList,CheckBoxList เพื่อทำการเรียกข้อมูล array จาก function ใน Model ที่เราสร้างไว้

สร้างตารางตามนี้

```sql
CREATE TABLE `resume` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(150) DEFAULT NULL COMMENT 'คำนำหน้า',
  `fistname` varchar(255) DEFAULT NULL COMMENT 'ชื่อ',
  `lastname` varchar(255) DEFAULT NULL COMMENT 'นามสกุล',
  `education_level` int(11) DEFAULT NULL COMMENT 'การศึกษา',
  `marital_status` int(11) DEFAULT NULL COMMENT 'สถานะสมรส',
  `sex` int(11) DEFAULT NULL COMMENT 'เพศ',
  `skill` varchar(10) DEFAULT NULL COMMENT 'ทักษะ',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

ซึ่งแต่ละฟิวด์มีข้อมูล items ที่ใช้เป็นตัวเลือกแตกต่างกันดังนี้

**title**

- นาย
- นาง
- นางสาว

**sex**

- ชาย
- หญิง

**marital_status**

- โสด
- แต่งงานแล้ว
- เป็นหม้าย
- แต่งแล้วแต่ยังไม่มีบุตร
- หย่า

**education_level**

- ต่ำกว่ามัธยมศึกษาตอนต้น
- มัธยมศึกษาตอนต้น
- ปวช
- มัธยมศึกษาตอนปลาย
- ปวส
- อนุปริญญา
- ปริญญาตรี
- ปริญญาโท
- ปริญญาเอก

**skill**

- Git
- PHP
- JavaScript
- CSS
- HTML5
- AngularJs
- ReactJs
- Node.js
- Go
- Ruby on Rails
- Swift
- Android
- MySQL
- MongoDB

## Gii Model Resume และ Gii CRUD Resume

ทำการ gii model และ gii crud ให้เรียบร้อยและทำการทดลองใช้งานว่าทำงานได้

## สร้าง function ใน model เพื่อเตรียมไว้เรียกใช้งานใน Form

เราจะทำการสร้าง function itemsAlias() เพื่อเก็บค่า array ทั้งหมดที่ต้องการใช้งานใน Model Resume.php ดังนี้

ฟิวด์แรกคือ sex จะทำการประกาศเป็น constant เพื่อให้สามารถเรียกใช้งานด้วยชื่อง่ายๆ ไม่ต้องจำว่า 1 คือ ชาย, 2 คือ หญิง เวลาจะเรียกใช้งานก็ `Resume::SEX_MEN` เลย เพิ่มการกำหนดค่าเข้าไปใน Resume ดังนี้

```PHP
class Resume extends \yii\db\ActiveRecord
{

const SEX_MEN = 1;
const SEX_WOMEN = 2;
// ...
```


ข้อดีของการประกาศค่า constant แบบนี้ สมมุติว่าจะมีการเรียกข้อมูลที่ค่า `sex = 1` นั่นก็คือชาย ปกติเราก็จะทำการเขียนแบบนี้

```
$model = Resume::find()->where('sex=:sex',[':sex'=>1])->all();
```
คำสั่งด้านบนคือ ค้นหาข้อมูลที่ sex =  1 หรือ ค้นหาข้อมูลผู้ชายนั้นเอง ซึ่งเมื่อเรามองดูโค้ดแล้ว เราก็คงเข้าใจเพราะเราเขียนเอง และเรารู้ว่าค่า 1 คือผู้ชาย แต่ถ้าหาเป็นคนอื่นละ เค้าก็ต้องค้นหาว่า ค่า 1 เนี้ย มันคือ ชาย หรือ หญิงกันแน่ ซึ่งมันก็จะเสียเวลาในการตรวจสอบโค้ด

แต่ถ้าหาเราใช้ตัวแปร constant แล้วทุกอย่างมันจะดูง่ายขึ้นทันที ตัวอย่าง

```
$model = Resume::find()->where('sex=:sex',[':sex'=>Resume::SEX_MEN])->all();
```

แค่เปลี่ยนจากที่เราระบุค่าเข้าไปตรงๆ ไปเรียก constant แทน มันมีข้อดียังไงมาดูกัน

- เวลาอ่านโค้ดดูง่ายเข้าใจ ไม่ต้องไปเดาว่า 1 มันคืออะไร เค้าไม่ต้องรู้ด้วยซ้ำก็ได้
- หากมีการเปลี่ยนค่าสามารถแก้ได้ที่จุดเดียว ตัวอย่างเช่น หากเรากำหนดค่า 1 เท่ากับ ชาย แล้ววันดีคืนดี เกิดอยากเปลี่ยนเป็น M = ชายแทน ยุ่งละทีนี้ ตรงใหนที่ where เท่ากับ 1 ไว้ต้องไล่ตามเปลี่ยนทุกจุด แต่หากเราใช้ constant แล้วก็สามารถเปลี่ยนได้แค่จุดเดียว

> หากค่าตัวใหนไม่จำเป็นต้องใช้ where หรือเรียกใช้งานอื่นๆ  ก็ไม่จำเป็นต้องประกาศเป็น constant เหมือน `sex` ก็ได้ ให้กำหนดค่าใน array itemsAlias ไปเลย


หลังจากนั้นให้เราทำการสร้าง `static function itemsAlias()` เพื่อเก็บค่า array ทั้งหมดไว้ที่นี่ โดย สามารถเรียกใช้งาน และระบุค่า key ที่ต้องการเข้าไป ก็จะได้ array ของ items ที่เราต้องการ เช่น `Resume::itemsAlias('sex')` จะ return ค่า array sex ออกมา เขียนเป็นฟังก์ชันได้ดังนี้

```PHP
<?php

class Resume extends \yii\db\ActiveRecord
{

const SEX_MEN = 1;
const SEX_WOMEN = 2;

// ...

public static function itemsAlias($key){

  $items = [
    'sex'=>[
      self::SEX_MEN => 'ชาย',
      self::SEX_WOMEN => 'หญิง'
    ],
    'title'=>[
      1 => 'นาย',
      2 => 'นางสาว',
      3 => 'นาง'
    ],
    'marital'=>[
      1 => 'โสด',
      2 => 'สมรส',
      3 => 'เป็นหม้าย',
      4 => 'หย่าร้าง'
    ],
    'education'=>[
      1 => 'ต่ำกว่ามัธยมศึกษาตอนต้น',
      2 => 'มัธยมศึกษาตอนต้น',
      3 => 'ปวช',
      4 => 'มัธยมศึกษาตอนปลาย',
      5 => 'ปวส',
      6 => 'อนุปริญญา',
      7 => 'ปริญญาตรี',
      8 => 'ปริญญาโท',
      9 => 'ปริญญาเอก'
    ],
    'skill'=>[
      'php' => 'PHP',
      'js' => 'JavaScript',
      'css' => 'CSS',
      'html5' => 'Html5',
      'angularjs' => 'AngularJs',
      'node.js' => 'Node.Js',
      'reactjs' => 'ReactJs',
      'go'=>'Go',
      'ruby'=>'ruby on rails',
      'swiff' => 'Swiff',
      'android' => 'Android',
  ]
];
  return ArrayHelper::getValue($items,$key,[]);
  //return array_key_exists($key, $items) ? $items[$key] : [];
}
```

จากนั้นสร้าง function `getItem` เพื่อให้สามารถเรียกใช้งาน array item แต่ละตัวในฟอร์มได้ง่ายๆ เพื่อให้ฟอร์มนำไปแสดงผลเป็น DropdownList,RadioList,CheckBoxList


```PHP
<?php

public function getItemSex()
{
  return self::itemsAlias('sex');
}

public function getItemMarital()
{
  return self::itemsAlias('marital');
}

public function getItemEducation()
{
  return self::itemsAlias('education');
}

public function getItemTitle()
{
  return self::itemsAlias('title');
}

public function getItemSkill()
{
  return self::itemsAlias('skill');
}
```

หลายๆ คนอาจสงสัยเราสร้าง itemsAlias ไว้แล้วทำไมยังต้องสร้าง getItem ขึ้นมาอีก จริงๆ สามารถเรียกใช้ itemsAlias ตรงๆ ได้ และหากดูตามตัวอย่างด้านล่างแล้ว จะเห็นว่ามันต้อง `use app\models\Resume` เพื่อให้สามารถเรียกใช้งาน function แบบ static แบบนี้ `Resume::itemsAlias('title')` ได้

> เรื่องนี้มันเกี่ยวกับการประกาศ function ใน class เป็น public,static ลองไปหาข้อมูลเพิ่มเติมดูนะครับ

```PHP
<?php

use yii\helpers\Html;
use yii\widgets\ActiveForm;
use app\models\Resume;
?>

<div class="resume-form">

    <?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'title')->radioList(Resume::itemsAlias('title')) ?>
```

มันต้องเรียกผ่าน static ในไฟล์ view  `_form.php` อีกที ทั้งๆ ที่มี ตัวแปร $model อยู่แล้วผมจึงแนะนำให้สร้าง getItem ขึ้นมาให้เรียกใช้งานผ่าน $model ได้ เวลาดูโค้ดก็เข้าใจง่าย และเราไม่ต้อง  `use app\models\Resume`  เพิ่มเติม

```PHP
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;
?>

<div class="resume-form">

    <?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'title')->radioList($model->getItemTitle()) ?>
```

เวลาดูโค้ดก็เข้าใจเลย ไม่ต้องเดา ฟังก์ชั่นเหี้ยอะไรของมัน `บ่นๆ เวลาเจอโค้ดเน่าๆ`

รูปแบบนี้สามารถเรียกใช้งานได้หมดเลยทั้ง   DropdownList,RadioList,CheckBoxLis ลองดูตัวอย่างการเรียกใช้งาน จะเห็นว่าง่ายและสามารถดูแล้วเข้าใจ

```php
    <?= $form->field($model, 'title')->inline()->radioList($model->getItemTitle()) ?>

    <?= $form->field($model, 'sex')->radioList($model->getItemSex()) ?>

    <?= $form->field($model, 'education_level')->dropdownList($model->getItemEducation(),['prompt'=>'เลือกการศึกษา..']) ?>

    <?= $form->field($model, 'marital_status')->radioList($model->getItemMarital()) ?>

    <?= $form->field($model, 'skill')->checkBoxList($model->getItemSkill()) ?>
```

จริงๆ เราสร้างเป็นแบบง่ายๆ `getItems()` ก็ได้ แต่ผมไม่ได้ใช้แบบนี้

```php
 public function getItems($type)
 {
   return  self::itemsAlias($type);
 }
```

เพราะผมคิดว่าทำแยกมันดูง่าย อ่านโค้ดเข้าใจง่ายกว่า เลยใช้วิธีนี้เพราะเวลาเขียนไปแล้วนานๆ กลับมาดูรู้สึกว่ามันเข้าใจง่ายดี

จากนั้นปรับแต่ง layout นิดหน่อยพอเป็นพิธี ให้สวยงาม ก็จะได้ฟอร์มแบบนี้

![](/img/item-alias-form.png)

โค้ดส่วน form ทั้งหมด

```php
<?php

use yii\helpers\Html;
use yii\bootstrap\ActiveForm;

/* @var $this yii\web\View */
/* @var $model app\models\Resume */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="resume-form">

    <?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'title')->inline()->radioList($model->getItemTitle()) ?>
    <div class="row">

      <div class="col-md-6">
        <?= $form->field($model, 'firstname')->textInput(['maxlength' => true]) ?>
      </div>
      <div class="col-md-6">
          <?= $form->field($model, 'lastname')->textInput(['maxlength' => true]) ?>
      </div>
    </div>
    <?= $form->field($model, 'education_level')->dropdownList($model->getItemEducation(),['prompt'=>'เลือกการศึกษา..']) ?>
    <div class="row">
      <div class="col-md-4">
          <?= $form->field($model, 'sex')->radioList($model->getItemSex()) ?>
      </div>
      <div class="col-md-4">
        <?= $form->field($model, 'marital_status')->radioList($model->getItemMarital()) ?>
      </div>
      <div class="col-md-4">
        <?= $form->field($model, 'skill')->checkBoxList($model->getItemSkill()) ?>
      </div>
    </div>

    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? Yii::t('app', 'Create') : Yii::t('app', 'Update'), ['class' =>  $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>

```

## จัดการข้อมูลที่ได้จาก checkBoxList

ในฟอร์มหากเราใช้ CheckBoxLis เราจะเก็บข้อมูลไม่เหมือนตัวอื่นๆ คือเราจะเก็บข้อมูลแล้วคั่นด้วยเครื่องหมาย `,` แบบนี้ครับ เพราะมันเลือกได้หลายอัน ซึ่งมันจะทำให้ลดคอลัมน์ในตารางไปเยอะมากๆ ในตัวอย่างนี้คือฟิวด์ `skill` ลองมาดูว่ามันทำงานยังไง

![](/img/item-alias-flow.png)

- ในขั้นตอนแรก CheckBoxLis จะ render checkbox ตาม array ที่เราได้มาจาก `$model->getItemSkill()`   และนำมาแสดงผลให้เราสามารถเลือก `skill` ได้
- เมือมีการ submit เพื่อบันทึกข้อมูล skill จะมีค่าเป็น array ดังนั้นเราจะต้องทำการแปลงจาก array เป็น string โดยใช้ฟังก์ชัน `implode`  ให้อยู่ในรูปแบบ `jquery,php,js,css` นี้ จากนั้นนำไปบันทึกข้อมูล
- เมื่อจะทำการแก้ไขข้อมูล เราก็จะต้องแปลงค่า  `jquery,php,js,css` ที่เก็บในฟิวด์ skill ให้กลับไปเป็น array เหมือนเดิมเพื่อให้ฟอร์มแสดงผลว่าเราได้เลือกอะไรไปบ้าง โดยใช้ `explode` เพื่อแปลง string กลับเป็น array จะเป็นแบบนี้

![](/img/item-alias-checkbox.png)

มันก็จะวลลูปอยู่แบบนี้ เอาล่ะคงพอเข้าใจการทำงานแล้ว เราลองมาดูในส่วนของโค้ดกัน

**ก่อน Create & Update แปลงข้อมูลจาก array เป็น string**

ก่อนที่จะนำข้อมูลไปบันทึกเราจะแปลงข้อมูลจาก checkBoxList ที่เป็น array ให้เป็น string โดยใช้ฟังก์ชัน `implode()` เพื่อให้เราไม่ต้องเขียนโค้ดวุ่นวายเราจะใช้ความสามารถของ  `behaviors()`  ของ model เพื่อดักจับ event `before-insert`,`before-update` เพื่อให้มันทำงาน ณ ตอนนั้น เพิ่มฟังก์ชันนี้เข้าไปใน Model Resume.php

```php
<?php

namespace app\models;

use Yii;
use yii\helpers\ArrayHelper;
use yii\behaviors\AttributeBehavior;
use \yii\db\ActiveRecord;
/**
 * This is the model class for table "{{%resume}}".
 *
 * @property integer $id
 * @property string $title
 * @property string $firstname
 * @property string $lastname
 * @property integer $education_level
 * @property integer $marital_status
 * @property integer $sex
 * @property string $skill
 */
class Resume extends ActiveRecord
{
  const SEX_MEN = 1;
  const SEX_WOMEN = 2;

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
    // ...
  }
```

เมื่อเรา create & update ข้อมูลก่อนบันทึกมันจะทำการแปลงข้อมูล array ให้เป็น string แล้วนำไปบันทึกที่ฐานข้อมูลอีกที

**ตอน Update แปลงข้อมูลจาก String เป็น Array**

เมื่อเราต้องทำการ update ข้อมูล ก่อนจะแสดงฟอร์ม update เราจะต้องแปลงข้อมูลที่เป็น string ให้กลับมาอยู่ในรูปแบบ array เสียก่อน เพื่อให้ฟอร์มนำไปแสดงผลได้

เพิ่มฟังก์ชัน `skillToArray()` เข้าไปใน Model Resume.php

```php
<?php

namespace app\models;

use Yii;
use yii\helpers\ArrayHelper;
use yii\behaviors\AttributeBehavior;
use \yii\db\ActiveRecord;
/**
 * This is the model class for table "{{%resume}}".
 *
 * @property integer $id
 * @property string $title
 * @property string $firstname
 * @property string $lastname
 * @property integer $education_level
 * @property integer $marital_status
 * @property integer $sex
 * @property string $skill
 */
class Resume extends ActiveRecord
{
  const SEX_MEN = 1;
  const SEX_WOMEN = 2;

  // ....

  public function skillToArray()
  {
    return $this->skill = explode(',', $this->skill);
  }
    // ....

}
```

หลังจากนั้น เรียกใช้งานใน ResumeController.php

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

ลองทดสอบดู เราก็จะสามารถบันทึกข้อมูล,แก้ไขข้อมูลได้ แต่ยังแสดงผลใน พวก gridView หรือ DetailView ได้ยังไม่สมบูรณ์ ต้องปรับแต่งเพิ่มเติมอีก ไปดูกัน


## สร้าง Function เพื่อเรียกแสดงผลโดยใช้ Virtual Attribute

หลังจากที่เราบันทึกข้อมูลไปแล้ว จะต้องนำข้อมูลมาแสดงผลใน gridView,DetailView หรือที่อื่นๆ ยังไงเพราะในฐานข้อมูลมันเป็นแค่ ตัวเลข เดี่ยวเราจะสร้าง function เพื่อแปลงค่าจาก ตัวเลขต่างๆ ที่เราเก็บในฐานข้อมูลให้เป็นข้อความ โดยใช้ความสามารถของ model คือ virtualAttribute

`Virtual Attribute` คือฟิวด์ที่ไม่มีในฐานข้อมูลจริงๆ แต่เราสามารถสร้างมันและเรียกใช้งานได้ เหมือนมันมีอยู่จริงๆ เลย หลักการไม่มีอะไรมาก มายแค่ประกาศ function ใน model ขึ้นต้นด้วยคำว่า get ตัวอย่าง เช่น  `getFullName()`
เวลาเรียกใช้งาน ก็ `$model->fullName`  แบบนี้ได้เลย โอ้วม่าย...แจ่มมาก `จะร้องทำไม!` จะสังเกตได้ว่าเวลาเรียกเราไม่ต้องมีเครื่องหมาย () ตามหลัง เพราะมันเป็นคุณสมบัติ Virtual Attribute

OK ดูตัวอย่างใน Model กันก่อน

เราจะสร้างฟังก์ชั่น getFullName เพื่อรวม คำนำหน้า,ชื่อ,นามสกุล ให้สามารถเรียกได้ใน ฟิวด์เดียว

```PHP
<?php

public function getFullName()
{
   return $this->titleName.$this->firstname.' '.$model->lastname;
}

```

การเรียกใช้งานผ่านตัวแปร $model

```
$model->fullName;
```

เรียกใช้งานผ่าน gridView

```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    'columns' => [
        ['class' => 'yii\grid\SerialColumn'],
        'fullName', // <<-----------
        'education_level',
        'marital_status',
        'sex',
        'skill',
        ['class' => 'yii\grid\ActionColumn'],
    ],
]); ?>
```

เรียกใช้งานผ่าน DetailView

```php
<?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'fullName', // <<-----------
        'education_level',
        'marital_status',
        'sex',
        'skill',
    ],
]) ?>
```

จะเห็นว่ามันมองเป็นเหมือนเป็นฟิวด์จริงๆ ที่สามารถเรียกใช้งานได้เลย เอาล่ะคิดว่าน่าจะพอเข้าใจ concept ของการสร้าง Virtual Attribute ทีนี้เราลองมาสร้าง function สำหร้บที่เราจะใช้งานกัน

เราจะสร้าง function เพื่อแสดงผลว่าเราได้กรอกข้อมูลอะไรเข้าไป โดย function จะทำการเรียก itemsAlias() ที่เป็น static function เพื่อให้ได้ค่า array ของประเภทนั้นๆ ออกมา จากนั้นใช้ `ArrayHelper::getValue` เพื่อดึงค่า label ออกมาแสดงเป็นข้อความให้เรา

```php
<?php
public function getSexName(){
    return ArrayHelper::getValue($this->getItemSex(),$this->sex);
}

public function getMaritalName(){
    return ArrayHelper::getValue($this->getItemSex(),$this->marital_status);
}

public function getEducationName(){
    return ArrayHelper::getValue($this->getItemEducation(),$this->education_level);
}

public function getTitleName(){
    return ArrayHelper::getValue($this->getItemTitle(),$this->title);
}

```

ตอนเรียกใช้งานก็ $model->sexName,$model->titleName,$model->skillName ได้เลย ส่วนใน gridView,DetailView ก็เหมือนตัวอย่าง fullName ด้านบน

แต่ในส่วนของ skill เราต้องปรับแต่งเป็น พิเศษเพื่อให้แสดงข้อมูลได้หลายๆ รายการเพราะมันเลือกได้มากกว่า 1 รายการ เพิ่มฟังก์ชั่นนี้ใน Model Resume.php เพิ่มฟังก์ชัน `getSkillName()` เข้าไป

```php
<?php

// ...

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
        'skillName',

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
        'skillName'
    ],
]) ?>
```

![](/img/item-alias-detailview-1.png)

เราจะสังเกตว่าตรง label คำอธิบายฟิวด์ต่างๆ ยังเป็นภาษาอังกฤษเราจะต้องปรับปรุง AttributeLabel() เพิ่มเติมใน Model Resume.php

```php
<?php

// ...

public function attributeLabels()
{
    return [
        'id' => Yii::t('app', 'ID'),
        'title' => Yii::t('app', 'คำนำหน้า'),
        'firstname' => Yii::t('app', 'ชื่อ'),
        'lastname' => Yii::t('app', 'นามสกุล'),
        'education_level' => Yii::t('app', 'การศึกษา'),
        'marital_status' => Yii::t('app', 'สถานะสมรส'),
        'sex' => Yii::t('app', 'เพศ'),
        'skill' => Yii::t('app', 'ทักษะ'),
        // เพิ่มเติม
        'fullName'=> Yii::t('app', 'ชื่อ - นามสกุล'),
        'educationName'=>Yii::t('app', 'การศึกษา'),
        'maritalName'=> Yii::t('app', 'สถานะสมรส'),
        'sexName' => Yii::t('app', 'เพศ'),
        'skillName'=> Yii::t('app', 'ทักษะ')
    ];
}
```

หัวคอลัมน์ก็จะเป็นภาษาไทยแล้ว

![](/img/item-alias-gridview-2.png)
![](/img/item-alias-detailview-2.png)

## สรุปโค้ดทั้งหมดที่เพิ่มใน model

```php
<?php

namespace app\models;

use Yii;
use yii\helpers\ArrayHelper;
use yii\behaviors\AttributeBehavior;
use \yii\db\ActiveRecord;
/**
 * This is the model class for table "{{%resume}}".
 *
 * @property integer $id
 * @property string $title
 * @property string $firstname
 * @property string $lastname
 * @property integer $education_level
 * @property integer $marital_status
 * @property integer $sex
 * @property string $skill
 */
class Resume extends ActiveRecord
{
  const SEX_MEN = 1;
  const SEX_WOMEN = 2;

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
    /**
     * @inheritdoc
     */
    public static function tableName()
    {
        return '{{%resume}}';
    }

    /**
     * @inheritdoc
     */
    public function rules()
    {
        return [
            [['title','firstname', 'lastname','education_level', 'marital_status', 'sex'], 'required'],
            [['education_level', 'marital_status', 'sex'], 'integer'],
            [['title'], 'string', 'max' => 150],
            [['firstname', 'lastname'], 'string', 'max' => 255],
            [['skill'], 'safe'],
        ];
    }

    /**
     * @inheritdoc
     */
    public function attributeLabels()
    {
        return [
            'id' => Yii::t('app', 'ID'),
            'title' => Yii::t('app', 'คำนำหน้า'),
            'firstname' => Yii::t('app', 'ชื่อ'),
            'lastname' => Yii::t('app', 'นามสกุล'),
            'education_level' => Yii::t('app', 'การศึกษา'),
            'marital_status' => Yii::t('app', 'สถานะสมรส'),
            'sex' => Yii::t('app', 'เพศ'),
            'skill' => Yii::t('app', 'ทักษะ'),

            'fullName'=> Yii::t('app', 'ชื่อ - นามสกุล'),
            'educationName'=>Yii::t('app', 'การศึกษา'),
            'maritalName'=> Yii::t('app', 'สถานะสมรส'),
            'sexName' => Yii::t('app', 'เพศ'),
            'skillName'=> Yii::t('app', 'ทักษะ')
        ];
    }

    /**
     * @inheritdoc
     * @return ResumeQuery the active query used by this AR class.
     */
    public static function find()
    {
        return new ResumeQuery(get_called_class());
    }

    public static function itemsAlias($key){

      $items = [
        'sex'=>[
          self::SEX_MEN => 'ชาย',
          self::SEX_WOMEN => 'หญิง'
        ],
        'title'=>[
          1 => 'นาย',
          2 => 'นางสาว',
          3 => 'นาง'
        ],
        'marital'=>[
          1 => 'โสด',
          2 => 'สมรส',
          3 => 'เป็นหม้าย',
          4 => 'หย่าร้าง'
        ],
        'education'=>[
          1 => 'ต่ำกว่ามัธยมศึกษาตอนต้น',
          2 => 'มัธยมศึกษาตอนต้น',
          3 => 'ปวช',
          4 => 'มัธยมศึกษาตอนปลาย',
          5 => 'ปวส',
          6 => 'อนุปริญญา',
          7 => 'ปริญญาตรี',
          8 => 'ปริญญาโท',
          9 => 'ปริญญาเอก'
        ],
        'skill'=>[
          'php' => 'PHP',
          'js' => 'JavaScript',
          'css' => 'CSS',
          'html5' => 'Html5',
          'angularjs' => 'AngularJs',
          'node.js' => 'Node.Js',
          'reactjs' => 'ReactJs',
          'go'=>'Go',
          'ruby'=>'ruby on rails',
          'swiff' => 'Swiff',
          'android' => 'Android',
      ]
    ];
      return ArrayHelper::getValue($items,$key,[]);
      //return array_key_exists($key, $items) ? $items[$key] : [];
    }

    public function getItemSex()
    {
      return self::itemsAlias('sex');
    }

    public function getItemMarital()
    {
      return self::itemsAlias('marital');
    }

    public function getItemEducation()
    {
      return self::itemsAlias('education');
    }

    public function getItemTitle()
    {
      return self::itemsAlias('title');
    }

    public function getItemSkill()
    {
      return self::itemsAlias('skill');
    }

    public function getSexName(){
        return ArrayHelper::getValue($this->getItemSex(),$this->sex);
    }

    public function getMaritalName(){
        return ArrayHelper::getValue($this->getItemMarital(),$this->marital_status);
    }

    public function getEducationName(){
        return ArrayHelper::getValue($this->getItemEducation(),$this->education_level);
    }

    public function getTitleName(){
        return ArrayHelper::getValue($this->getItemTitle(),$this->title);
    }

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

    public function getFullName()
    {
       return $this->titleName.$this->firstname.' '.$this->lastname;
    }

    public function skillToArray()
    {
      return $this->skill = explode(',', $this->skill);
    }

}

```

## เพิ่มการ Filter & Sorting

ตอนนี้มันยังใช้งาน gridview ได้ยังไม่สมบูรณ์คือ ยังไม่สามารถกดตรงหัวคอลัมน์ให้มันจัดเรียงข้อมูลได้และยังไม่สามารถค้นหาได้ เราจะมาปรับแต่ง gridview กัน

ตัวแรกจะเป็น fullName ตัวนี้มันไม่ใช่ฟิวด์ที่มีอยู่จริงๆ ใน table resume เพราะฉะนั้นเราจะต้องกำหนดค่าให้มันสามารถ Sorting และ Search ได้ โดยเข้าไปปรับแต่งที่ Model ResumeSearch.php

ให้เราประกาศตัวแปร `$fullName` และแก้ไขฟังก์ชัน rules() เพิ่มฟิวด์ fullName เข้าไป ในตัว validator `safe` เพื่อให้ gridview รู้ว่ามันสามารถค้นได้

```php
<?php

//...

class ResumeSearch extends Resume
{
  public $fullName;
    /**
     * @inheritdoc
     */
    public function rules()
    {
        return [
            [['id', 'education_level', 'marital_status', 'sex'], 'integer'],
            [['title', 'firstname', 'lastname', 'skill','fullName'], 'safe'],
        ];
    }
}
```

จากนั้นให้ทำการแก้ไขฟังก์ชัน Search() เพื่อทำให้มัน Sorting และ นำตัวแปร $fullName ไปค้นหาจริงๆ

```php
<?php

//...


public function search($params)
{
    $query = Resume::find();

    $dataProvider = new ActiveDataProvider([
        'query' => $query,
    ]);

    // Sorting by fullName
    $dataProvider->sort->attributes['fullName'] = [
       'asc' => ['firstname' => SORT_ASC, 'lastname' => SORT_ASC],
       'desc' => ['firstname' => SORT_DESC, 'lastname' => SORT_DESC],
       'default' => SORT_ASC
    ];

    $this->load($params);

    if (!$this->validate()) {
        // uncomment the following line if you do not want to return any records when validation fails
        // $query->where('0=1');
        return $dataProvider;
    }

    $query->andFilterWhere([
        'id' => $this->id,
        'education_level' => $this->education_level,
        'marital_status' => $this->marital_status,
        'sex' => $this->sex,
    ]);

    // filter by person full name
     $query->andWhere('firstname LIKE "%' . $this->fullName . '%" ' .
         'OR lastname LIKE "%' . $this->fullName . '%"'
     );

    $query->andFilterWhere(['like', 'title', $this->title])
        ->andFilterWhere(['like', 'firstname', $this->firstname])
        ->andFilterWhere(['like', 'lastname', $this->lastname])
        ->andFilterWhere(['like', 'skill', $this->skill]);

    return $dataProvider;
}
```

จากนั้น fullname จะสามารถค้นหาข้อมูลได้

ส่วน column อื่นๆ ปรับแต่งใน Gridviw โดยใช้การกำหนด property ของ Column และยังเพิ่ม filter ให้เป็น dropdownList โดยการระบบค่า array items ที่เราได้สร้างไว้เข้าไป จากนั้นก็เรียกใช้งาน virtualAttribute ที่เราได้สร้างไว้ return ออกไปแสดงผลได้เลยเพราะเราได้สร้างเตรียมไว้หมดแล้ว

```php
<?php
use app\models\Resume;
 ?>
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
        'fullName',
        [
          'attribute'=>'education_level',
          'filter'=>Resume::itemsAlias('education'),
          'value'=>function($model){
            return $model->educationName;
          }
        ],
        [
          'attribute'=>'marital_status',
          'filter'=>Resume::itemsAlias('marital'),
          'value'=>function($model){
            return $model->maritalName;
          }
        ],
        [
          'attribute'=>'sex',
          'filter'=>Resume::itemsAlias('sex'),
          'value'=>function($model){
            return $model->sexName;
          }
        ],
        [
          'attribute'=>'skill',
          'filter'=>Resume::itemsAlias('sex'),
          'value'=>function($model){
            return $model->skillName;
          }
        ],
        // 'educationName',
        // 'maritalName',
        // 'sexName',
        // 'skillName',

        ['class' => 'yii\grid\ActionColumn'],
    ],
]); ?>
```

![](/img/item-alias-gridview-success.png)

## สรุป

คิดว่าคงพอเป็นตัวอย่างที่ดีในการพัฒนาได้ บางคร้ัง เราอาจเขียนโปรแกรมให้มันใช้ใช้งานได้ แต่อย่าลืมว่าเมื่อเราต้องมีการแก้ไปหรือปรับปรุงระบบ มันต้องง่ายและยืดหยุ่น และที่สำคัญต้องให้คนอื่นอ่านโค้ดเราเข้าใจได้ด้วย ไม่งั้นพอได้คนใหม่มาหรือต้องทำงานรว่มกันเป็นทีม  ก็ต้องมารือทำระบบใหม่อีกที เป็นวนเวียนอยู่แบบนี้ ถ้าเขียนแล้วมันใช้ได้นานๆ ยืดยุ่น มันก็น่าคิดนะครับ ว่าคุณแค่อยากเขียนโค้ดแค่ให้พอรันได้ ให้มันเสร็จๆไป หรือคุณอยากเขียนโค้ดให้อ่านได้ รันได้ และยืดยุ่นกว่า ส่วนใหญ่ bug มันก็เกิดตั้งแต่ส่วนเล็กๆ แบบนี้แหละครับ ทำจุดเล็กๆ แต่ละจุดให้มันดี พอรวมกันแล้วมันจะแข็งแกร่งอย่างไม่น่าเชื่อ..

สำหรับผมแล้วสิ่งเหล่านี้คนอื่นอาจไม่เห็น คนใช้อาจไม่รู้ แต่มันเป็นความสุขที่สัมผัสได้ เป็นความภูมิใจเล็กๆ น้อยๆ ^ ^

ดาวโหลด Source Code [ได้ที่นี่](https://github.com/dixonsatit/yii2-workshop-items-alias)

@DixonSatit
