---
layout : post
title : การสร้างฟอร์มค้นหาแบบ ajax ด้วย pjax
---

![](/img/thumbnail/pjax-gridview.jpg)

วันนี้เราจะลองเปลี่ยนฟอร์มค้นหาแบบเดิมที่เรา Gii CRUD มาแล้วให้เป็น ajax โดยใช้ pjax ซึ่งเป็น widget ที่มีมาพร้อมกับ Yii 2 อยู่แล้วสามารถเรียกใช้ได้ทันที

## Pjax คืออะไร

Pjax คือการนำเอาความสามารถของ pushState (Html5 History Api) มารวมกับ Ajax ซึ่งมันทำให้ ajax ธรรมดาๆ มันน่าใช้ยิ่งขึ้นลองไปดูรายละเอียดกัน

### Ajax

จริงๆ ต้องเกริ่นก่อนว่า ajax ธรรมดานั้น มันมีข้อดีข้อเสียยังไง ข้อดีคิดว่าทุกคนคงรู้กันอยู่ ส่วนข้อเสียหลักๆ คือ ในการใช้ ajax เมื่อมีการเรียกใช้งาน ajax แล้วเราไม่สามารถที่จะกดปุ่ม back หรือ forward ใน History ของ Browser ได้ หากกด back กลับไปแล้ว กด forward กลับมาข้อมูลจะไม่เหมือนเดิม จะต้องทำการเรียก request ใหม่ถึงได้จะได้ข้อมูล เหมือนเดิม เพราะฉะนั้นพวก search engine ก็จะไม่สามารถเก็บข้อมูลตาม Url ได้

### pushState (Html5 History Api)

 Html5 History Api คือการที่เราสามารถเก็บหน้าเว็บไซต์ต่างๆ ที่รันแล้ว เก็บไว้ที่ History Api ของ Browser ทำให้สามารถกดปุ่ม back หรือ forward ได้ โดยที่ไม่ต้องไปเรียกข้อมูลอีกครั้ง โดยใช้คำสั่ง history.pushState() ของ html5

 > ที่ผมเข้าใจน่าจะประมาณนี้ แต่ถ้าหากไม่ถูกต้องหรือใครต้องการเพิ่มเติมก็แนะนำมาได้ครับ

### Pjax = pushState + Ajax  

เมื่อนำ 2 สิ่งนี้มารวมกันมันก็เกิดระเบิดตู้มเป็นโกโก้ครั้น (55 ล้อเล่น) มันก็ทำให้เราสามารถใช้ ajax ได้และยังสามารถกดปุ่ม back หรือ forward ได้เพราะมันเอาข้อมูลที่เรารันโดยใช้ ajax ไว้ไปเก็บไว้ที่ history api เรียบร้อย ซึ่งตรงนี้ทำให้ search engine สามารถ index url ของเราได้แล้วเป็นผลดีกับ SEO ด้วย

 ต้องขอบคุณ คุณ [Chris Wanstrath](https://github.com/defunkt/jquery-pjax) ที่ได้สร้าง pjax ให้เราได้ใช้งานกัน

ในส่วนของ Yii 2 นั้นได้นำมารวมไว้เป็น widget หลักเลยทีเดียวไม่ต้องติดตั้งเพิ่ม สามารถเรียกใช้งานได้ทันที แจ่มมากๆ แสดงว่าดีจริง หากเราจะใช้งานให้ได้ดีๆ จริงๆแนะนำให้ศึกษา pjax ให้เข้าใจก่อนถึงจะสามารถนำมาใช้ กับ yii2 ได้อย่างมีประสิทธิภาพ

จริงๆ หลักการหลักๆ ไม่มีอะไรมากแค่เอา widget pjax มาครอบส่วนที่เราต้องการทำ pjax  เอาล่ะมาลองสร้างกัน

## ลองใช้ pjax แบบ Basic

อย่างที่ผมบอกมันง่ายมากๆ แค่นำ widget pjax มาครอบส่วนที่ต้องการ มันก็กลายเป็น ajax ทันที โอ้ว!

เช่น ในตัวอย่างนี้เราจะให้การค้นหาใน gridView ในส่วนของ filter แบบปกติ ให้มันค้นเป็น ajax ได้ เราก็แค่เรียกใช้งาน pjax และนำมันมาครอบ gridView ของเราไว้ ตั้งชื่อและกำหนด timeout ไว้ซัก 5 วิ เพราะหากมัน request เกิน 5 วิมันจะ refresh เหมือนเดิม

รูปแบบก็จะประมาณนี้

```php
<?php
use yii\helpers\Html;
use yii\grid\GridView;
use yii\widgets\Pjax;
 ?>
<?php yii\widgets\Pjax::begin(['id' => 'grid-user-pjax','timeout'=>5000]) ?>
   // ......
<?php yii\widgets\Pjax::end() ?>
```

จากนั้นนำ GridView มาใส่

```php
<?php
use yii\helpers\Html;
use yii\grid\GridView;
use yii\widgets\Pjax;
 ?>
<?php yii\widgets\Pjax::begin(['id' => 'grid-user-pjax','timeout'=>5000]) ?>
<?= GridView::widget([
                'id'=>'grid-user',
                'dataProvider' => $dataProvider,
                'filterModel' => $searchModel,
                'tableOptions' => [
                  'class' => 'table table-bordered  table-striped table-hover',
                ],
                'columns' => [
                    ['class' => 'yii\grid\SerialColumn'],
                    'username',
                    'email:email',
                    [
                      'label'=>'Active',
                      'format'=>'html',
                      'value'=>function($model){
                          return $model->status==0?'<i class="glyphicon glyphicon-remove"></i> <span class="text-danger">Not Active</span>':'<i class="glyphicon glyphicon-ok"></i> <span class="text-success">Active</span>';
                      }
                    ],
                    [
                      'attribute'=>'levelName',
                      'format'=>'html',
                      'value'=>function($model){
                        return $model->levelName;
                      }
                    ],
                    [
                      'class' => 'yii\grid\ActionColumn',
                      'options'=>['style'=>'width:120px;'],
                      'buttonOptions'=>['class'=>'btn btn-default'],
                      'template'=>'<div class="btn-group btn-group-sm text-center" role="group"> {view} {update} {delete} </div>'
                   ],
                ],
            ]); ?>
<?php yii\widgets\Pjax::end() ?>
```
ก็จะได้หน้าตาแบบนี้

![](/img/pjax-basic.png)

เมื่อมีการค้นหาในช่อง filter ใน gridView ก็จะเห็น url ยาวๆ แบบนี้ ชื่อ url นี้แหละมันจะนำไปเป็น key ในการเรียกใช้งาน pushState ของ html5 history api

![](/img/pjax-addressbar.png)

ก็จะเห็นหน้าตาค้นหาแบบนี้ สังเกตว่าหน้าเว็บไซต์จะไม่มีการ refresh

![](/img/pjax-search.png)

ให้เราลองเปิด debug ของ Browser ดูโดยคลิกขวาแล้วเลือก Inspect Element แล้วเลือก tab network แล้วลองค้นหาดูจะพบว่ามี request ajax จากนั้นลองค้นหลายๆ ค่าดูแล้วกด back หรือ forward ใน Browser ดูจะพบว่ายังมีค่าเหมือนเดิม ซึ่งมันจะไม่ได้โหลดค่าใหม่แต่เอามาจาก Htm5 History Api

![](/img/pjax-inspact.png)

คงจะพอเข้าใจหลักในการสร้างฟอร์มค้นหาด้วย pjax อย่างง่ายๆ ทีนี้เราจะลองปรับแต่งโดยเพิ่มฟอร์มค้นหาที่เราสร้างขึ้นมาเอง ลองไปดูกัน

## สร้างฟอร์มค้นหาแบบ ajax

เราจะเพิ่มเติมฟอร์มค้นหาเข้าไป โดยออกแบบฟอร์มค้นหาใหม่และไม่ค้นที่ filter ของ gridView ผมเองมองว่ามันไม่ค่อยสะดวก ถ้าทำเองจะได้ปรับแต่งง่ายๆ และยังสามารถใข้ pjax ได้เหมือนเดิมอีกด้วย

โดยเราจะสร้างฟอร์มหน้าตาแบบนี้

![](/img/pjax-form-search.png)

จะสังเกตุว่าเราจะใช้ช่องค้นหาเพียงช่องเดียว เพราะฉะนั้นเราจะต้องทำการปรับปรุง modeSearch (ของผมต้อนนี้คือ UserSearch) เพื่อให้สามารถค้นหาด้วยช่องเดียวได้ ก่อนจะสร้างฟอร์มให้เราไปทำการแก้ไข modeSearch ให้เสร็จเรียบร้อยก่อน

> ปกติในตอน Gii CRUD เราจะได้ Model ที่มีชื่อตามท้ายว่า Search เช่น model User ก็จะได้เป็นชื่อไฟล์ UserSearch.php

## ปรับปรุง ModelSearch

เราจะทำการสร้างฟิวด์ขึ้นมาใหม่ชื่อ q เพื่อเอาไว้รับค่าในช่องที่เรากรอกข้อความเพื่อค้นหา

ไปที่ UserSearch หรือ ModelSearch ของคุณแล้วทำการประกาศตัวแปรชื่อ $q ดังนี้

```php
<?php

// ...
class UserSearch extends User
{
    public $q;
    // ....
}
```

และทำการเพิ่ม validation ให้กับตัวแปร `$q` เพื่อให้มันสามารถรับค่าได้โดยกำหนด validation type เป็น `safe` ใน `function rules()` ดังนี้ หากไม่กำหนด model จะไม่สามารถรับค่าได้เพราะมันจะไม่รู้จัก

```php
<?php

//....
public function rules()
{
    return [
        [['id', 'status', 'created_at', 'updated_at'], 'integer'],
        [['username', 'auth_key', 'password_hash', 'password_reset_token', 'email','q'], 'safe'],
    ];
}
//...
```

จากนั้นทำการปรับปรุง `function Search()` เพื่อให้มันสามารถค้นหาข้อความจาก ฟิวด์ เดียวได้ เพราะปกติตัว search ที่ gii มามันจะใช้เงื่อนไข and แล้วก็ like ด้วย เราจะเปลี่ยนมาใช้ or และ like แทน สังเกตจะเรียกใช้งาน `andFilterWhere()` อยู่ด้านล่าง ให้ทำการลบฟิลด์ที่ไม่ได้ค้นออกไปทั้งหมดเหลือไว้เฉพาะที่ต้องการ ส่วนของผมเลือกแค่ 2 ฟิวด์คือ username,email จะสังเกตว่าตรงค่าที่มันนำมาใช้เรียก andFilterWhere() ก็จะเป็นชื่อฟิวด์นั้นๆ แยกกันไป ของเดิมจะเป็นแบบนี้

```php
<?php

public function search($params)
  {
      $query = User::find();

      $dataProvider = new ActiveDataProvider([
          'query' => $query,
      ]);

      $this->load($params);

      if (!$this->validate()) {
          // uncomment the following line if you do not want to return any records when validation fails
          // $query->where('0=1');
          return $dataProvider;
      }


      $query->andFilterWhere(['like', 'username', $this->username])
          ->andFilterWhere(['like', 'email', $this->email]);

      return $dataProvider;
  }
```

เราจะต้องทำการปรับปรุงส่วนรับค่า ให้รับค่ามาจากฟิวด์ที่ชื่อ $q ที่เราตั้งขึ้นมาใหม่เพื่อไว้รับค่าค้นหาโดยเฉพาะ และทำการเปลี่ยนชื่อฟิวด์อื่นๆ เป็น $q ทั้งหมด และเปลี่ยนจาก `andFilterWhere`  เป็น `orFilterWhere` ซึ่งก็จะเหมือนเราใช้คำสั่ง  sql ประมาณนี้ `(username like "%%") or (email like "%%")` ทำการแก้ไขโค้ดให้เป็นดังนี้

```php
<?php

public function search($params)
  {
      $query = User::find();

      $dataProvider = new ActiveDataProvider([
          'query' => $query,
      ]);

      $this->load($params);

      if (!$this->validate()) {
          // uncomment the following line if you do not want to return any records when validation fails
          // $query->where('0=1');
          return $dataProvider;
      }

      $query->orFilterWhere(['like', 'username', $this->q])
          ->orFilterWhere(['like', 'email', $this->q]);

      return $dataProvider;
  }
```

หลังจากแก้ modeSearch ของเราเสร็จเรียบร้อย ModelSearch ของเราก็สามารถรองรับการค้นหาได้แล้ว


### สร้างฟอร์มค้นหา

เราจะทำการสร้างฟอร์มค้นหาขึ้นมาใหม่ โดยให้ทำการสร้าง view ชื่อ \_search.php และใส่โค้ดเข้าไปดังนี้

> ฟอร์มที่เราจะนำมาใช้กับ pjax ต้องเพิ่ม properties `'data-pjax' => true` ในส่วนของ options ด้วย ไม่งั้นมันจะ submit form ปกติแบบ refresh


```php
<?php

use yii\helpers\Html;
use yii\widgets\ActiveForm;

/* @var $this yii\web\View */
/* @var $model backend\models\UserSearch */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="user-search">

    <?php $form = ActiveForm::begin([
        'action' => ['index'],
        'method' => 'get',
        'options' => ['data-pjax' => true ]
    ]); ?>
    <div class="input-group">
      <?= Html::activeTextInput($model, 'q',['class'=>'form-control','placeholder'=>'ค้นหาข้อมูล...']) ?>
      <span class="input-group-btn">
        <button class="btn btn-default" type="submit"><i class="glyphicon glyphicon-search"></i> ค้นหา</button>
        <?= Html::a('<i class="glyphicon glyphicon-plus"></i> '.Yii::t('app', 'Create User'), ['create'], ['class' => 'btn btn-default']) ?>
      </span>
    </div>
    <?php ActiveForm::end(); ?>

</div>

```

## เปิดใช้งาน pjax

เราจะนำ widgets pjax มาครอบ gridView ไว้และทำการเรียก ฟอร์มค้นหาไว้ใน pjax ด้วยดังนี้

> เราจะทำการปิด filter ของ gridView ออกไป โดยลบบรรทัดนี้ออกไปหรือคอมเม้นไว้   `'filterModel' => $searchModel`

```php
<?php
use yii\helpers\Html;
use yii\grid\GridView;
use yii\widgets\Pjax;
 ?>
<?php yii\widgets\Pjax::begin(['id' => 'grid-user-pjax','timeout'=>5000]) ?>

<!-- เรียก view _search.php -->
<?php echo $this->render('_search', ['model' => $searchModel]); ?>
<br>

<?= GridView::widget([
                'id'=>'grid-user',
                'dataProvider' => $dataProvider,
                //'filterModel' => $searchModel,
                'tableOptions' => [
                  'class' => 'table table-bordered  table-striped table-hover',
                ],
                'columns' => [
                    ['class' => 'yii\grid\SerialColumn'],
                    'username',
                    'email:email',
                    [
                      'label'=>'Active',
                      'format'=>'html',
                      'value'=>function($model){
                          return $model->status==0?'<i class="glyphicon glyphicon-remove"></i> <span class="text-danger">Not Active</span>':'<i class="glyphicon glyphicon-ok"></i> <span class="text-success">Active</span>';
                      }
                    ],
                    [
                      'attribute'=>'levelName',
                      'format'=>'html',
                      'value'=>function($model){
                        return $model->levelName;
                      }
                    ],
                    [
                      'class' => 'yii\grid\ActionColumn',
                      'options'=>['style'=>'width:120px;'],
                      'buttonOptions'=>['class'=>'btn btn-default'],
                      'template'=>'<div class="btn-group btn-group-sm text-center" role="group"> {view} {update} {delete} </div>'
                   ],
                ],
            ]); ?>
<?php yii\widgets\Pjax::end() ?>
```

เมื่อลองรันก็จะได้หน้าตาแบบนี้

![](/img/pjax-gridview.png)


ให้ทดลองค้นหาดูก็จะพบว่ามันเป็น ajax แล้ว ซึ่ง url ก็จะเปลี่ยนไปตามค่า parameter ที่เราค้นหา ลองทดสอบค้นหลายๆ คำ แล้วลองกด back หรือ forward ใน Browser ดูจะพบว่าข้อมูลไม่หายไปสามารถกดดูได้อย่างรวดเร็วเลย เพราะมันก็ข้อมูลไว้ที่ History Api นั้นเอง

หวังว่าคงทำให้เข้าใจเรื่อง pjax มากขึ้นนะครับ

บทความนี้เป็นส่วนหนึ่งของ Workshop@Co-Space [ ดูหน้าตัวอย่าง](https://github.com/dixonsatit/yii2-workshop-co-sapce/blob/master/backend/views/user-manage/index.php)

@dixonsatit
