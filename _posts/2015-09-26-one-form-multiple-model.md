---
layout : post
title : การสร้างฟอร์มที่สามารถบันทึกพร้อมกันได้ทีละหลายๆ Model
---

ปกติการสร้างฟอร์มเพื่อบันทึกและแก้ไขข้อมูลนั้น เราจะทำเพียงแค่ครั้งละ 1  Model  แต่ก็มีในบางครั้งที่เราจำเป็นจะต้องบันทึกพร้อมกันทีละหลายๆ  model ซึ่งใน Yii 2 นั้นสามารถทำได้ง่ายๆ เลย จะทำกี่ model ก็ได้ ส่วนใหญ่จะเป็นตารางที่มีการเชื่อมกัน (Relation) อยู่แล้ว

# โครงสร้างและหลักการทำงาน

การทำงานจะเหมือนกันทั้งสองแบบ ต่างกันแค่เรื่องของการ validate ข้อมูลเท่านั้น ซึ่งปกติจะใช้ฟังก์ชัน validate() ผ่าน model แต่ถ้าหากต้องบันทึกพร้อมกันทีละหลายๆ model จะต้องใช้ validateMultiple() แทน ลองดูความแตกต่างตามภาพ

## Single model

![](/img/sigleModel.jpg)

ในการบันทึกข้อมูลปกติจะมีอยู่ 4 ขั้นตอนคือ

- `$model = new SomeModel()` แล้วนำไปแสดงผลเป็นฟอร์มเพื่อรับข้อมูล
- เมื่อมีการ submit จะรับข้อมูลจากฟอร์มโดยใช้ฟังก์ชัน `$model->load()`
- ตรวจสอบข้อมูลที่โหลดเข้ามาโดยใช้ `$model->validate()`
- นำข้อมูลที่ได้ไปบันทึก


## Multiple model

![](/img/mulipleModel.jpg)

เมื่อลองเปรียบเทียบกันดูจะเห็นความแตกต่าง 2 จุดคือ ปกติจะใช้ฟังก์ชัน validat() ใน model แต่ถ้าหากเป็นทีละหลายๆ model จะใช้ Model::validateMultiple()   แทน ซึ่ง validate() แบบเดิมจะไม่สามารถ validate ได้ทีละหลายๆ model
ในการบันทึกข้อมูลจะมีอยู่ 4 ขั้นตอนคือ

- `$model = new ModelA(), $model = new ModelB()` แล้วนำไปแสดงผลเป็นฟอร์มเพื่อรับข้อมูล
- เมื่อมีการ submit จะรับข้อมูลจากฟอร์มโดยใช้ฟังก์ชัน `$modelA->load(),$modelB->load()` แยกกัน
- ตรวจสอบข้อมูลที่โหลดเข้ามาโดยใช้ `Model::validateMultiple()` ร่วมกันทั้ง 2 model
- นำข้อมูลที่ได้ไปบันทึก

> ปกติในการใช้งานเราจะไม่เห็นการเรียกใช้งานฟังก์ชัน validate() เพราะมันถูกเรียกใช้งานอยู่ภายในฟังก์ชัน save() อยู่แล้ว


# ลงมือสร้างฟอร์ม

ตัวอย่างนี้เราจะลองสร้างตาราง 2 ตารางแบบง่ายๆ  โดยตารางแรกชื่อ m_user เก็บข้อมูล username,email ตารางที่ 2 m_emp เก็บข้อมูล คำนำหน้า,ชื่อ,นามสกุล  ซึ่งตาราง m_emp จะเก็บข้อมูล user_id เพื่อ relation กับ m_user  ด้วย id   และเมื่อมีการบันทึกข้อมูลเราจะให้มันสามารถบันทึกทั้ง 2 ตารางได้พร้อมกัน

ให้สร้างตารางตามโครงสร้างนี้

```sql
CREATE TABLE `m_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(150) DEFAULT NULL,
  `email` varchar(150) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8

CREATE TABLE `m_emp` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(100) DEFAULT NULL,
  `name` varchar(150) DEFAULT NULL,
  `surname` varchar(150) DEFAULT NULL,
  `user_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `user_id` (`user_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8
```

จากนั้นให้สร้าง model ด้วย gii ทั้ง 2 ตาราง เป็น MUser,MEmp  หลังจากสร้าง model เสร็จให้ gii สร้าง crud ตาราง m_emp ต่อเพื่อสร้างฟอร์มเพิ่มลบแก้ไข แต่ไม่ต้องสร้าง crud ของ m_user เพราะเราจะนำไปรวมที่ฟอร์ม m_emp  ทดลองเข้าใช้งาน `index?r=m-emp/index` ว่าสามารถใช้งานได้ทุกอย่างเพิ่ม, ลบ, แก้ไข


# ปรับปรุงส่วนต่างๆ

ส่วนแรกที่ต้องปรับปรุงคือ actionCreate() ไปที่ไฟล์ controllers/MEmpController.php

## ปรับปรุง actionCreate() ใน MEmpController.php

ของเดิมจะ new object ขึ้นมาแค่ตัวเดียวคือ MEmp()

```php
<?php

// ...

public function actionCreate()
{
    $model = new MEmp();

    if ($model->load(Yii::$app->request->post()) && $model->save()) {
        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('create', [
            'model' => $model,
        ]);
    }
}
```

ปรับปรุงใหม่โดย use MUser เพิ่มเติมด้านบนสุด

```php
<?php

use backend\models\MUser;

// ...

public function actionCreate()
{
    $modelEmp = new MEmp();
    $modelUser = new MUser();

    if($modelEmp->load(Yii::$app->request->post()) &&
       $modelUser->load(Yii::$app->request->post()) &&
       Model::validateMultiple([$modelEmp,$modelUser]))
    {
        if($modelUser->save()){
          $modelEmp->user_id = $modelUser->id;
          $modelEmp->save();
        }
        return $this->redirect(['view', 'id' => $modelEmp->id]);
    } else {
        return $this->render('create', [
            'model' => $modelEmp,
            'modelUser'=>$modelUser
        ]);
    }
}
```

สังเกตว่าเราจะต้อง new object ที่ต้องการขึ้นมาจะใช้กี่ตัวในฟอร์มก็ต้อง new object ตามจำนวนที่ต้องการ จากนั้นจะส่ง model ไปแสดงผลที่ฟอร์ม

เมื่อมีการ กด submit เข้ามาจะสังเกตว่า model จะเรียกข้อมูลที่ submit เข้ามาด้วยฟังก์ชัน load() ของใครของมัน จากนั้นก็เรียก `Model::validateMultiple()`  เพื่อ validate ข้อมูล

เมื่อ validate เสร็จเรียบร้อย ก็นำข้อมูลที่ได้ผ่านการตรวจสอบแล้วนำไปบันทึกโดยเราจะให้ MUser บันทึกก่อนเพราะว่า MEmp ต้องใช้ user_id ซึ่งเป็น pk ของ MUser จากนั้นก็จะสามารถบันทึกได้ทั้ง 2 ตาราง

## ปรับปรุง views/m-emp/\_form.php

เราจะทำการเรียกฟอร์มของ user เข้าไปด้วยโดยใช้ตัวแปร $modelUser และทำการปรับปรุง layout เล็กน้อย

```php
<?php

use yii\helpers\Html;
use yii\widgets\ActiveForm;

/* @var $this yii\web\View */
/* @var $model backend\models\MEmp */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="memp-form">

    <?php $form = ActiveForm::begin(); ?>
    <div class="panel panel-default">
      <div class="panel-body">
        <h3>Employee</h3>
        <div class="row">
          <div class="col-md-2">
              <?= $form->field($model, 'title')->textInput(['maxlength' => true]) ?>
          </div>
          <div class="col-md-5">
            <?= $form->field($model, 'name')->textInput(['maxlength' => true]) ?>
          </div>
          <div class="col-md-5">
            <?= $form->field($model, 'surname')->textInput(['maxlength' => true]) ?>
          </div>
        </div>    
      </div>
        </div>
  <div class="panel panel-default">
    <div class="panel-body">
        <h3>User</h3>
    <?= $form->field($modelUser, 'username')->textInput(['maxlength' => true]) ?>
    <?= $form->field($modelUser, 'email')->textInput(['maxlength' => true]) ?>
  </div>
    </div>
    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? 'Create' : 'Update', ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>

```

เมื่อทดสอบเรียกไปที่หน้า `index.php?r=m-emp/create` ก็จะพบหน้าตาดังนี้

![](/img/multiple-model-form.png)

ทดลอง create ดู

## ปรับปรุง actionUpdate() ใน MEmpController.php

```php
<?php

public function actionUpdate($id)
{
    $model = $this->findModel($id);
    $modelUser = $this->findModelUser($model->user_id);

    if (
    $model->load(Yii::$app->request->post()) &&
    $modelUser->load(Yii::$app->request->post()) &&
    Model::validateMultiple([$model,$modelUser])
    ) {
        if($modelUser->save()){
          $model->save();
        }
        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('update', [
          'model' => $model,
          'modelUser'=>$modelUser
        ]);
    }
}

protected function findModelUser($id)
{
    if (($model = MUser::findOne($id)) !== null) {
        return $model;
    } else {
        throw new NotFoundHttpException('The requested page does not exist.');
    }
}
```

ฟังก์ชัน update จะแตกต่างจาก create เล็กน้อยตรงที่ดึงข้อมูลมาก่อนเพื่อ update ซึ่งปกติหากเรา gii crud จะมี findModel() ซึ่งเป็นของ MEmp เพราะฉะนั้นเราจึงต้องสร้าง `findModelUser()` ขึ้นมาเพิ่มเติมเพื่อดึงข้อมูลส่วนของ user

![](/img/multiple-model-form-update.png)

ทดลอง create,update ข้อมูลดู

> ในการทำ one form multiple model ส่วนที่สำคัญที่สุดจะมีเพียงส่วน Model::multipleValidate() เท่านั้น

## แก้ไข models/MEmp.php

เพิ่มการ validate required เข้าไป และสร้าง relation เชื่อกับ model MUser เพิ่มคำอธิบายฟิวด์ในส่วน attributeLabels()

```php
<?php

// ...

public function rules()
{
    return [
        [['title','name','surname'], 'required'],
        [['user_id'], 'integer'],
        [['title'], 'string', 'max' => 100],
        [['name', 'surname'], 'string', 'max' => 150]
    ];
}

public function attributeLabels()
{
    return [
        'id' => 'ID',
        'title' => 'Title',
        'name' => 'Name',
        'surname' => 'Surname',
        'user_id' => 'User ID',
        'username' => 'รหัสผู้ใช้งาน',
        'fullname' => 'ชื่อ-นามสกุล',
    ];
}

// ...

public function getUser(){
  return $this->hasOne(MUser::className(),['id'=>'user_id']);
}

public function getFullname(){
  return $this->title.$this->name.' '.$this->surname;
}

public function getUsername(){
  return $this->user->username;
}


```

แต่ละตัวมีหน้าที่ดังนี้

- `getUser()` เป็นฟังก์ชันที่เอาไว้เชื่อมกับตาราง m_user
- `getFullname()` แสดงคำนำหน้าชื่อ, ชื่อ, นามสกุล
- `getUsername()` แสดงค่า username ของตาราง m_user ซึ่งเรียกผ่านตัว relation ที่ชื่อ `getUser` ตัวอย่างการเรียก ฟังก์ชัน เช่น `$model->fullnam`

โดยเราเตรียมฟังก์ชันเหล่านี้ไว้เพื่อใช้ในตอนแสดงผลที่ gridview หรือ DetailView หรือที่อื่นๆ ก็ได้


## แก้ไข models/MUser.php

เพิ่มเฉพาะในส่วนฟังก์ชัน rules()

```php
<?php

// ...

public function rules()
{
    return [
        [['username'], 'required'],
        [['id'], 'integer'],
        [['username', 'email'], 'string', 'max' => 150],
        ['email','email']
    ];
}

```

# เพิ่มการค้นหาและจัดเรียงข้อมูล ( Filter & Sort  )

เราจะเพิ่มเติมส่วนค้นหาและการจัดเรียงข้อมูลในส่วนของหัวคอลัมน์

## แก้ไข models/MEmpSearch.php

ประกาศตัวแปรเพิ่มเติม เพื่อให้สามารถเรียกใช้งานใน model ได้เพราะ 2 ฟิวด์นี้ไม่มีอยู่จริงใน table m_emp

```
    public $fullname;
    public $username;
```

และอย่าลืมกำหนด validate ใน rule เพื่อให้ model สามารถใช้งาน 2 ฟิวด์นี้ได้

```
public function rules()
{
    return [
        [['id', 'user_id'], 'integer'],
        [['title', 'name', 'surname','fullname','username'], 'safe'],
    ];
}
```

เราจะเพิ่มส่วนการจัดเรียงข้อมูลเข้าไป มี fullname,username ดังนี้

```php
$dataProvider->sort->attributes['fullname']=[
  'asc'=>['name'=>SORT_ASC,'surname'=>SORT_DESC],
  'desc'=>['name'=>SORT_DESC,'surname'=>SORT_DESC],
];
$dataProvider->sort->attributes['username']=[
  'asc'=>['m_user.username'=>SORT_ASC],
  'desc'=>['m_user.username'=>SORT_DESC],
];
```

ส่วนการค้นหาเพิ่ม fullname

```php
 $query->andWhere('name like "%'.$this->fullname.'%" or surname like "%'.$this->fullname.'%" ');
```


โค้ดทั้งหมด

```php
<?php

namespace backend\models;

use Yii;
use yii\base\Model;
use yii\data\ActiveDataProvider;
use backend\models\MEmp;

/**
 * MEmpSearch represents the model behind the search form about `backend\models\MEmp`.
 */
class MEmpSearch extends MEmp
{
    public $fullname;
    public $username;
    /**
     * @inheritdoc
     */
    public function rules()
    {
        return [
            [['id', 'user_id'], 'integer'],
            [['title', 'name', 'surname','fullname','username'], 'safe'],
        ];
    }

    /**
     * @inheritdoc
     */
    public function scenarios()
    {
        // bypass scenarios() implementation in the parent class
        return Model::scenarios();
    }

    /**
     * Creates data provider instance with search query applied
     *
     * @param array $params
     *
     * @return ActiveDataProvider
     */
    public function search($params)
    {
        $query = MEmp::find()->joinWith('user');

        $dataProvider = new ActiveDataProvider([
            'query' => $query,
        ]);

        $dataProvider->sort->attributes['fullname']=[
          'asc'=>['name'=>SORT_ASC,'surname'=>SORT_DESC],
          'desc'=>['name'=>SORT_DESC,'surname'=>SORT_DESC],
        ];
        $dataProvider->sort->attributes['username']=[
          'asc'=>['m_user.username'=>SORT_ASC],
          'desc'=>['m_user.username'=>SORT_DESC],
        ];

        $this->load($params);

        if (!$this->validate()) {
            // uncomment the following line if you do not want to return any records when validation fails
            // $query->where('0=1');
            return $dataProvider;
        }

        $query->andWhere('name like "%'.$this->fullname.'%" or surname like "%'.$this->fullname.'%" ');

        $query->andFilterWhere([
            'id' => $this->id,
            'user_id' => $this->user_id,
            'm_user.username' => $this->username,
        ]);


        // $query->andFilterWhere(['like', 'title', $this->title])
        //     ->andFilterWhere(['like', 'name', $this->name])
        //     ->andFilterWhere(['like', 'surname', $this->surname]);

        return $dataProvider;
    }
}

```

## แก้ไข view/m-emp/index.php

เราจะปรับปรุงส่วนค้นหาใน GridView ขั้นตอนแรก เราจะเรียกใช้งาน fullname, username ที่เราได้สร้างไว้ที่ model MEmp

```php
<?php

use yii\helpers\Html;
use yii\grid\GridView;

/* @var $this yii\web\View */
/* @var $searchModel backend\models\MEmpSearch */
/* @var $dataProvider yii\data\ActiveDataProvider */

$this->title = 'Memps';
$this->params['breadcrumbs'][] = $this->title;
?>

<div class="panel panel-default">
  <div class="panel-body">

    <h1><?= Html::encode($this->title) ?></h1>
    <?php // echo $this->render('_search', ['model' => $searchModel]); ?>

    <p>
        <?= Html::a('Create Memp', ['create'], ['class' => 'btn btn-success']) ?>
    </p>

    <?= GridView::widget([
        'dataProvider' => $dataProvider,
        'filterModel' => $searchModel,
        'columns' => [
            ['class' => 'yii\grid\SerialColumn'],
            //'id',
            // 'title',
            // 'name',
            // 'surname',
            // 'user_id',
            'fullname',
            'username',
            // [
            //   'attribute'=>'user_id',
            //   'value'=>'user.username'
            // ],
            [
              'class' => 'yii\grid\ActionColumn',
              'options'=>['style'=>'width:120px;'],
              'buttonOptions'=>['class'=>'btn btn-default'],
              'template'=>'<div class="btn-group btn-group-sm text-center" role="group"> {view} {update} {delete} </div>'
           ],
        ],
    ]); ?>

  </div>
</div>

```

![](/img/multiple-model-index.png)

ทดลองเพิ่มลบแก้ไข จะพบว่าเราสามารถบันทึกข้อมูลได้แล้ว และบันทึกได้พร้อมๆ กัน

# ตรวจสอบความถูกต้องด้วย Transaction

เพื่อทำให้ระบบของเราน่าเชื่อถือมากขึ้นควรตรวจสอบการบันทึกข้อมูลว่าสำเร็จทุกตารางหรือไม่โดยใช้ Transaction เข้ามาช่วยตรวจสอบกระบวนการทำงานว่าถูกต้องหรือไม่

รูปแบบโครงสร้าง

```
use yii\base\Model;

$transaction = Model::getDb()->beginTransaction();
try {
    // ...โค้ดส่วนที่เราต้องการดัก errror
   $transaction->commit();
} catch (\Exception $e) {
  // เมื่อมี error
   $transaction->rollBack();
}
```

เมื่อนำมารวมกับ actionCreate() จะได้ประมาณนี้

```php
public function actionCreate()
    {
        $modelEmp = new MEmp();
        $modelUser = new MUser();

        if($modelEmp->load(Yii::$app->request->post()) &&
           $modelUser->load(Yii::$app->request->post()) &&
           Model::validateMultiple([$modelEmp,$modelUser]))
        {
          $transaction = $modelUser::getDb()->beginTransaction();
          try {
            if($modelUser->save()){
              $modelEmp->user_id = $modelUser->id;
              $modelEmp->save();
            }
             $transaction->commit();
          } catch (\Exception $e) {
             $transaction->rollBack();
          }
            return $this->redirect(['view', 'id' => $modelEmp->id]);
        } else {
            return $this->render('create', [
                'model' => $modelEmp,
                'modelUser'=>$modelUser
            ]);
        }
    }
```

# เพิ่ม relation เองโดยให้ model จัดการ

เป็นการใช้ความสามารถของ model คือ link() โดยที่เราไม่ต้องตั้งค่า id เพื่อให้มัน relation กัน เราสามารถเรียกใช้ link() แทนเดียวมันทำให้เอง

ปกติ

```php
// ...

if($modelUser->save()){
  $modelEmp->user_id = $modelUser->id;
  $modelEmp->save();
}

//...

```
ใช้ความสามารถของ model โดยใช้ link() ([ดูเพิ่มเติม](http://www.yiiframework.com/doc-2.0/guide-db-active-record.html#saving-relations))

```php
// ...

  if($modelUser->save()){
    $modelEmp->link('user',$modelUser); // easy way saving relations
  }

//...

```

# สรุป

หลักการของการบันทึกทีละหลายๆ model ในฟอร์มเดียว มีแค่ 3 จุดคือ

1. ต้องสร้าง object model ที่เราต้องการใช้แล้วส่งไปแสดงผลที่ฟอร์ม
2. ในตอนรับข้อมูลจาการ submit ต้องใช้ load() แยกกัน ส่วน validate ใช้ Model::multipleValidate() ร่วมกัน
3. บันทึกตารางหลักก่อนแล้วค่อยบันทึกตารางรองอื่นๆ ตามลำดับ

คิดว่าคงทำให้เราพอเข้าใจหลักการทำงานเบื้องต้นและสามารถนำไปประยุกต์ใช้ในงานแบบต่างๆ ได้ นะครับ ^ ^

ตัวอย่าง SourceCode

- [controller](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/blob/master/backend/controllers/MEmpController.php)
- [model](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/blob/master/backend/models/MEmp.php)
- [modelSearch](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/blob/master/backend/models/MEmpSearch.php)
- [view](https://github.com/dixonsatit/yii2-workshop-rbac-db-02/tree/master/backend/views/m-emp)

@dixonsatit
