---
layout : post
title :  การสร้างฟอร์มอัพโหลดแบบง่ายๆ
---

การสร้างฟอร์มอัพโหลดไฟล์ ปกติเป็นเรื่องที่ยุ่งยากพอสมควรหากเขียนด้วย php ธรรมดา เพราะต้องจัดการหลายๆ ส่วนมากๆ แต่เราโชคดี! ^ ^ เพราะใช้ Yii Framework เราสามารถสร้างฟอร์ม เพื่อแนบไฟล์ เช่นรูปภาพหรือไฟล์ต่างๆ ได้โดยง่ายโดยใช้ [yii\web\UploadedFile](http://www.yiiframework.com/doc-2.0/yii-web-uploadedfile.html) ซึ่งจะต้องใช้งานร่วมกันกับ [Activeform](http://www.yiiframework.com/doc-2.0/yii-widgets-activeform.html) และ [Model](http://www.yiiframework.com/doc-2.0/guide-structure-models.html) โดยผมมีตัวอย่างการสร้างฟอร์มอัพโหลดทั้ง 2 แบบคือ

- แบบทีละไฟล์ (Uploading single File)
- แบบทีละหลายๆ ไฟล์ (Uploading multiple files)

จุดประสงค์หลักๆ คืออยากให้เราเข้าใจหลักการทำงานของการอัพโหลดไฟล์แบบพื้นฐานที่มากับ Yii Framework เมื่อเข้าใจหลักการทำงานดีแล้ว เราจะไปใช้พวก extension อื่นๆ ก็สามารถทำได้สบายๆ และรู้ว่าจะต้องไปปรับแต่งอะไรตรงใหนเพื่อให้ใช้กับงานของเราได้

# ส่วนประกอบในการสร้างฟอร์มอัพโหลด

ในการสร้างฟอร์มอัพโหลดจะต้องประกอบไปด้วย 3 ส่วนดังต่อไปนี้

- [Activeform](http://www.yiiframework.com/doc-2.0/yii-widgets-activeform.html) เป็นตัวรับข้อมูลจากผู้ใช้งาน ว่าทำการเลือกไฟล์อะไร หรือข้อมูลอื่นๆ
- [UploadedFile](http://www.yiiframework.com/doc-2.0/yii-web-uploadedfile.html) เป็นพระเอกของเรา มีหน้าที่นำไฟล์ที่เราเลือกไปอัพโหลดเก็บไว้บน Server ของเรา
- [Model](http://www.yiiframework.com/doc-2.0/guide-structure-models.html) เป็นผู้ช่วยที่ทำให้เราสามารถ validate ข้อมูลต่างๆ ได้เช่น ให้เลือกได้เฉพาะไฟล์รูปภาพเท่านั้น, การจำกัดขนาดไฟล์ที่อัพโหลด, การระบุจำนวนไฟล์ที่ให้อัพโหลดเป็นต้น

เมื่อเตรียมส่วนประกอบต่างๆ ครบแล้วก็ไปลุยสร้างของจริงกันเลยครับ

# หลักการทำงานของการอัพโหลดไฟล์

![](/img/basic-upload.jpg)

ขั้นตอนหลักๆ จะเริ่มจากผู้ใช้งาน

- กรอกข้อมูลต่างๆ เลือกไฟล์ที่ต้องการอัพโหลด
- ตัว Activeform จะทำงานร่วมกับ model เพื่อ validate ข้อมูลที่กรอกและเลือกเข้ามาโดยใช้ข้อมูลจาก model ที่เราได้กำหนดในส่วน rule() เพื่อ validate ข้อมูล
- เมื่อตรวจสอบข้อมูลถูกต้องตามกฎที่เราได้กำหนดไว้ใน model ข้อมูลจะถูกส่งไปที่ UploadedFile เพื่อนำไฟล์ไปทำการอัพโหลดบันทึกเก็บไว้ที่ Server
- ในส่วนของตารางข้อมูลจะเก็บไว้เพียงชื่อไฟล์เท่านั้น

# สร้างฟอร์มอัพโหลด

ตัวอย่างนี้ผมจะสร้างตารางขึ้นมาง่ายๆ เพื่อเก็บข้อมูล ชื่อ,นามสกุล,รูปภาพ เพื่อรับข้อมูลและสร้างฟอร์มอัพโหลดไฟล์รูปภาพ โครงสร้างมีดังนี้

```sql
CREATE TABLE `easy_upload` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(100) DEFAULT NULL,
  `surname` varchar(100) DEFAULT NULL,
  `photo` varchar(255) DEFAULT NULL,
  `photos` text,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

- สร้างตาราง `easy_upload` ตามโครงสร้างด้านบน
- gii model จากตาราง `easy_upload` และ gii crud
- ทดลองเข้าใช้งาน

![](/img/easy-upload-index.png)

## ตั้งค่า Model Validation

ในส่วนของ model เราจะต้องกำหนดค่า validate ของฟิวด์ `photo` ให้เป็น file เพื่อทำการตรวจสอบข้อมูลก่อนอัพโหลด เช่น เป็นรูปภาพ,เป็นไฟล์

เปิดไฟล์ model `EasyUpload.php` ขึ้นมา เพิ่ม validate `file` ให้กับฟิวด์ `photo` ในฟังก์ชัน rules() และมีการระบุ `extensions` นามสกุลไฟล์ที่สามารถอัพโหลดได้คือ png,jpg ส่วน `skipOnEmpty` คือหากไม่มีการอัพโหลดก็ให้มาข้ามในส่วนของ upload ไป

```php
<?php

namespace backend\models;

use Yii;
use \yii\web\UploadedFile;

class EasyUpload extends \yii\db\ActiveRecord
{
    public $upload_foler ='uploads';

// ....

public function rules()
{
    return [
        [['name', 'surname'], 'required'],
        [['name', 'surname'], 'string', 'max' => 100],
        [['photo'], 'file',
          'skipOnEmpty' => true,
          'extensions' => 'png,jpg'
        ]
    ];
}

public function upload($model,$attribute)
{
    $photo  = UploadedFile::getInstance($model, $attribute);
      $path = $this->getUploadPath();
    if ($this->validate() && $photo !== null) {

        $fileName = md5($photo->baseName.time()) . '.' . $photo->extension;
        //$fileName = $photo->baseName . '.' . $photo->extension;
        if($photo->saveAs($path.$fileName)){
          return $fileName;
        }
    }
    return $model->isNewRecord ? false : $model->getOldAttribute($attribute);
}

public function getUploadPath(){
  return Yii::getAlias('@webroot').'/'.$this->upload_foler.'/';
}

public function getUploadUrl(){
  return Yii::getAlias('@web').'/'.$this->upload_foler.'/';
}

public function getPhotoViewer(){
  return empty($this->photo) ? Yii::getAlias('@web').'/img/none.png' : $this->getUploadUrl().$this->photo;
}
// ...

}
```

> สร้างโฟลเดอร์ uploads ที่ `web/uploads` เพื่อเอาไว้เก็บไฟล์ที่อัพโหลดขึ้นไป

### ลำดับการทำงานของฟังก์ชัน upload()

- เรียกข้อมูลรูปภาพที่แนบมากับฟอร์มโดยใช้  `UploadedFile::getInstance()`
- ทำการตรวจสอบข้อมูลที่ส่งมาด้วย $this->validate() จริงๆ ตรงนี้ไม่ต้องมีก็ได้ แต่เรียกไว้เผื่อเรานำไปใช้ในส่วนที่ไม่ได้มีการ save ข้อมูลในตาราง
- เปลี่ยนชื่อไฟล์
- บันทึกไฟล์ลง server ด้วยคำสั่ง saveAs()
- ถ้าบันทึกสำเร็จจะส่งชื่อไฟล์กลับไป ถ้าไม่สำเร็จจะส่งค่า false ไปแทน
- ถ้าหากเป็นการอัพเดทฟอร์ม หากอัพโหลดภาพใหม่สำเร็จจะใช้ชื่อไฟล์ใหม่ แต่ถ้าไม่สำเร็จจะใช้ค่าเดิมที่มาจาก db


## ตั้งค่า Activeform ให้สามารถอัพโหลดไฟล์ได้

ให้ไปที่ไฟล์ view `easy-upload/_form.php`  ทำการเพิ่ม enctype เพื่อให้สามารถแนบไฟล์ไปกับฟอร์มได้

> หลายคนตกม้าตายเพราะลืมตั้งค่าส่วนนี้ T_T' ซึ่งผมก็เป็นหนึ่งในนั้น

```php
<?php $form = ActiveForm::begin([
    'options' => ['enctype' => 'multipart/form-data']
  ]); ?>

```

## เปิดใช้ fileInput

เรียกใช้งาน fileInput เพื่อให้สามารถเลือกไฟล์เพื่ออัพโหลดข้อมูลได้ และทำการจัดแต่ง layout

```php
  <?= $form->field($model, 'photo')->fileInput() ?>
```

โค้ด \_form.php ทั้งหมด

```php
<?php
use yii\helpers\Html;
use yii\widgets\ActiveForm;
?>
<div class="easy-upload-form">
  <?php $form = ActiveForm::begin([
      'options' => ['enctype' => 'multipart/form-data']
    ]); ?>
    <?= $form->field($model, 'name')->textInput(['maxlength' => true]) ?>
    <?= $form->field($model, 'surname')->textInput(['maxlength' => true]) ?>
    <div class="row">
      <div class="col-md-2">
        <div class="well text-center">
          <?= Html::img($model->getPhotoViewer(),['style'=>'width:100px;','class'=>'img-rounded']); ?>
        </div>
      </div>
      <div class="col-md-10">
            <?= $form->field($model, 'photo')->fileInput() ?>
      </div>
    </div>
    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? Yii::t('app', 'Create') : Yii::t('app', 'Update'), ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>
```


เมื่อลองรันหน้าฟอร์มจะได้แบบนี้

![](/img/easy-upload-form.png)

> รูปภาพ none.png [download](/img/none.png) ได้ที่นี่ นำไปไว้ที่ ` web/img/none.png`

## แก้ไข EasyUploadController.php

ให้เปิดไฟล์ controllers/EasyUploadController.php ของเราขึ้นมาแล้วทำการแก้ไข actionCreate & actionUpdate ดังนี้

```php
<?php

// ....

public function actionCreate()
{
    $model = new EasyUpload();

    if ($model->load(Yii::$app->request->post()) && $model->validate()) {
        $model->photo = $model->upload($model,'photo');
        $model->save();
        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('create', [
            'model' => $model,
        ]);
    }
}


public function actionUpdate($id)
{
    $model = $this->findModel($id);

    if ($model->load(Yii::$app->request->post()) && $model->validate()) {
        $model->photo = $model->upload($model,'photo');
        $model->save();
        return $this->redirect(['view', 'id' => $model->id]);
    }  else {
        return $this->render('update', [
            'model' => $model,
        ]);
    }
}

```

จากนั้นให้ลองทำการ upload file ดู ลองในส่วนของการบันทึกตอนอัพเดทด้วยว่า หากไม่ได้อัพโหลดไฟล์หลังจากบันทึกไฟล์เก่าต้องยังคงอยู่ มันจะเปลี่ยนก็ต่อเมื่อมีการอัพโหลดไฟล์ใหม่เข้าไปแทนที่ค่าเดิม

## ปรับปรุงการแสดงผลที่ actionIndex()

กำหนดให้แสดงภาพใน  GridView ได้

```php
<?= GridView::widget([
    'dataProvider' => $dataProvider,
    'filterModel' => $searchModel,
    'columns' => [
        //['class' => 'yii\grid\SerialColumn'],
        [
            'options'=>['style'=>'width:150px;'],
            'format'=>'raw',
            'attribute'=>'photo',
            'value'=>function($model){
              return Html::tag('div','',[
                'style'=>'width:150px;height:95px;
                          border-top: 10px solid rgba(255, 255, 255, .46);
                          background-image:url('.$model->photoViewer.');
                          background-size: cover;
                          background-position:center center;
                          background-repeat:no-repeat;
                          ']);
            }
        ],
        //'id',
        'name',
        'surname',
        // 'photo',
        // 'photos:ntext',

        ['class' => 'yii\grid\ActionColumn'],
    ],
]); ?>

```

![](/img/easy-upload-index-preview.png)


## ปรับปรุงการแสดงผลที่ actionView()

กำหนดให้แสดงภาพใน  DetailView ได้

```php
<?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'id',
        'name',
        'surname',
        //'photoViewer:image',
        [
            'format'=>'raw',
            'attribute'=>'photo',
            'value'=>Html::img($model->photoViewer,['class'=>'img-thumbnail','style'=>'width:200px;'])
        ]
    ],
]) ?>
```

 ![](/img/easy-upload-view.png)


# การอัพโหลดทีละหลายๆ ไฟล์

เราสามารถอัพโหลดไฟล์ทีละหลายๆ ไฟล์ได้ โดยใช้ `UploadedFile::getInstances()` ซึ่งวิธีเป็นตัวอย่างง่ายๆ ยังไม่มีในส่วนของการลบไฟล์ จริงๆ แล้วการอัพโหลดทีละหลายๆ ไฟล์ควรมีตารางเก็บจะสะดวกกว่ามาก แต่ตัวอย่างนี้ จะนำเสนอว่าหลักการทำงานของการอัพโหลดทีละหลายๆ ไฟล์นั้นเป็นยังไง เพื่อให้เข้าใจหลักการทำงานและสามารถนำไปพัฒนาต่อยอดได้ ให้ทำการเพิ่มเติมโค้ดในส่วนที่เหลือดังนี้

## แก้ไข model EasyUpload.php

เพิ่มฟังก์ชัน uploadMultiple() เพื่อบันทึกทีละหลายๆ ไฟล์ และเก็บข้อมูลในรูปแบบ string คั่นด้วยคอมม่า (,) แบบนี้ 'xxxx.jpg,xxxx.png'

```php
<?php

//...

    public function rules()
    {
        return [
            [['name', 'surname'], 'required'],
            [['name', 'surname'], 'string', 'max' => 100],
            [['photo'], 'file',
              'skipOnEmpty' => true,
              'extensions' => 'png,jpg'
            ],
            [['photos'], 'file',
              'skipOnEmpty' => true,
              'maxFiles' => 3,
              'extensions' => 'png,jpg'
            ]
        ];
    }

    public function uploadMultiple($model,$attribute)
    {
      $photos  = UploadedFile::getInstances($model, $attribute);
      $path = $this->getUploadPath();
      if ($this->validate() && $photos !== null) {
          $filenames = [];
          foreach ($photos as $file) {
                  $filename = md5($file->baseName.time()) . '.' . $file->extension;
                  if($file->saveAs($path . $filename)){
                    $filenames[] = $filename;
                  }
          }
          if($model->isNewRecord){
            return implode(',', $filenames);
          }else{
            return implode(',',(ArrayHelper::merge($filenames,$model->getOwnPhotosToArray())));
          }
      }

      return $model->isNewRecord ? false : $model->getOldAttribute($attribute);
    }

    public function getPhotosViewer(){
      $photos = $this->photos ? @explode(',',$this->photos) : [];
      $img = '';
      foreach ($photos as  $photo) {
        $img.= ' '.Html::img($this->getUploadUrl().$photo,['class'=>'img-thumbnail','style'=>'max-width:100px;']);
      }
      return $img;
    }

    public function getOwnPhotosToArray()
    {
      return $this->getOldAttribute('photos') ? @explode(',',$this->getOldAttribute('photos')) : [];
    }
```

## แก้ไข \_form.php

เพิ่มเติมส่วนนี้เข้าไป

```php
<?= $form->field($model, 'photos[]')->fileInput(['multiple' => true]) ?>
<div class="well">
  <?= $model->getPhotosViewer(); ?>
</div>
```
![](/img/easy-uploads-form.png)

## แก้ไข  view.php

เพิ่มในส่วนของการเรียกแสดงผลรูปภาพทีละหลายๆ ไฟล์

```php
<?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'id',
        'name',
        'surname',
        //'photoViewer:image',
        [
            'format'=>'raw',
            'attribute'=>'photo',
            'value'=>Html::img($model->photoViewer,['class'=>'img-thumbnail','style'=>'width:200px;'])
        ],
        [
            'format'=>'raw',
            'attribute'=>'photos',
            'value'=>$model->getPhotosViewer()
        ]
    ],
]) ?>
```

![](/img/easy-uploads-view.png)


## แก้ไข EasyUploadController.php

เรียกใช้งาน uploadMultiple() เพิ่มเติมเข้าไป ทั้ง create & update

```php
<?php

//...

public function actionCreate()
{
    $model = new EasyUpload();

    if ($model->load(Yii::$app->request->post()) && $model->validate()) {
        $model->photo = $model->upload($model,'photo');
        $model->photos = $model->uploadMultiple($model,'photos');
        $model->save();
        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('create', [
            'model' => $model,
        ]);
    }
}

public function actionUpdate($id)
{
    $model = $this->findModel($id);

    if ($model->load(Yii::$app->request->post()) && $model->validate()) {
        $model->photo = $model->upload($model,'photo');
        $model->photos = $model->uploadMultiple($model,'photos');
        $model->save();
        return $this->redirect(['view', 'id' => $model->id]);
    }  else {
        return $this->render('update', [
            'model' => $model,
        ]);
    }
}
```

# สรุปความเข้าใจ

การใช้งานอัพโหลดไฟล์จะมีแค่ 2 แบบคือ
- อัพโหลดทีละไฟล์ใช้ ` UploadedFile::getInstance()`
- อัพโหลดทีละหลายๆ ไฟล์ใช้ ` UploadedFile::getInstances()`

หลักๆ ในการอัพโหลดไฟล์ก็จะมีแค่นี้ นอกนั้นจะเป็นการปรับแต่งให้ใช้งานง่ายๆ ซึ่งจะต้องใช้ extension อื่นๆ เข้ามาใช้งานเพิ่มเติม หวังว่าบทความนี้คงจะทำให้เราเข้าใจพื้นฐานการอัพโหลดไฟล์นะครับ ^^

ตัวอย่างโค้ด

- [controller](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/blob/master/backend/controllers/EasyUploadController.php)
- [Model](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/blob/master/backend/models/EasyUpload.php)
- [views](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/tree/master/backend/views/easy-upload)

@dixonSatit
