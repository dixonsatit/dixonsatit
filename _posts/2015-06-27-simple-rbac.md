---
layout : post
title : การสร้าง RBAC ใน Yii 2 อย่างง่าย

---

![](/img/simple-rbac-index.jpg)

Tutorial นี้เป็นการสร้าง RBAC อย่างง่ายๆ เพื่อให้เราเข้าใจภาพรวมการใช้งาน RBAC ก่อน และเราจะสร้าง AccessRole ขึ้นมาใช้งานเองเป็นการตรวจสอบสิทธิ์แบบง่ายๆ

มาดูความหมายของ RBAC กันซักนิด RBAC ย่อมาจาก Role-Based Access Control หรือ เป็นตรวจตรวจสอบระดับของผู้ใช้งาน
Yii 2 จะแบ่ง ระดับไว้ที่ 3 ระดับ คือ

- Roles
- Tasks
- Operations

มาดูแต่ละส่วนกัน

- Operations  เป็นการกระทำเพียง 1 อย่าง หรือมองเป็น action ก็ได้ อย่างเช่น  `create`,`update`,`delete`
- Tasks เป็นชื่อกลุ่มที่รวบรวม Operations ต่างๆ ที่เกี่ยวข้องกัน อย่างเช่น `TasksUser`  จะมี Operations คือ UserCreate, UserUpdate, UserDelete มองประมาณ controller ก็ได้
- Roles คือกลุ่มของ Task,Operations และ Role  ทำให้เราจัดกลุ่มสิทธ์ต่างๆ ได้อย่างมีประสิทธิภาพ อย่างเช่น Role Admin, Role User, Role Administrator  ซึ่งในแต่ละกลุ่มก็แล้วแต่เราจะนำเอา Role,task ที่ใหนมาใช้บ้าง [ที่มา](http://yiisos.blogspot.com/2013/09/rbac.html)

ช่างมันเถอะ อาจจะงง ผมก็อธิบายไม่ถูก เดี่ยวทำไปๆ น่าจะเข้าใจ ^ ^

## เตรียม Project กันก่อน

ตัวอย่างนี้ ผมเราจะใช้ Advance Template เพราะมีระบบ login อยู่แล้ว หลังจากนั้นจะทำการสร้างระบบ Blog และระบบจัดการผู้ใช้งาน เพื่อทดลองสร้าง การตรวจสอบสิทธิ์ ให้คุณได้มองเห็นภาพและเข้าใจได้ง่ายยิ่งขึ้น โดยตัวอย่างนี้แบ่งเป็น 2 ส่วน

- การสร้าง RBAC
- การสร้าง controller User เพื่อเอาไว้จัดการสิทธิ์ให้แต่ละ User

การสร้าง RBAC
----

- สร้าง project Advance ขึ้นมา [อ่านวิธีติดตั้ง advance ได้ทีนี](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide/start-installation.md)
- รันคำสั่ง `php init`,` yii migrate` ให้เรียบร้อย สร้าง db ชื่อ `simple-rbac` และตั้งค่า config ฐานข้อมูลให้เรียบร้อย
- ลงทะเบียน user และทำการ login ให้สำเร็จ

เมื่อทำตามขั้นตอนนี้เสร็จแล้ว เราจะสร้างตาราง blog ขึ้นมาเพื่อทดสอบการใช้งานสิทธิ์โดยมีขั้นตอนดังนี้

1. สร้างตารางและทำการ gii Model, gii CRUD
2. ปรับปรุงแก้ไขไฟล์ User.php และทำการเพิ่มกลุ่มของ Role เข้าไป
3. สร้างไฟล์ AccessRole.php เพื่อเอาไว้เช็คใน controller เวลามีการเรียกใช้งานแต่ละ action
4. เรียกใช้งาน AccessRole ใน controller เพื่อควบคุมสิทธิ์การเข้าใช้งาน

ให้สร้าง Blog ขึ้นมาตามโครงสร้างนี้

```sql
CREATE TABLE `blog` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `topic` varchar(255) DEFAULT NULL COMMENT 'ชื่อเรื่อง',
  `detail` varchar(255) DEFAULT NULL COMMENT 'รายละเอียด',
  `tag` varchar(15) DEFAULT NULL COMMENT 'แท็ก',
  `created_by` int(11) DEFAULT NULL,
  `updated_by` int(11) DEFAULT NULL,
  `created_at` int(11) DEFAULT NULL,
  `updated_at` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8
```
จากนั้นไปที่ frontend และทำการ `Gii Model Blog `และ `Gii CRUD Blog` ตามนี้

**Gii Model Blog**

![](/img/simple-rbac-model-gii.png)

**Gii CRUD Blog**

![](/img/simple-rbac-gii-crud.png)

ทดสอบให้รันใช้งานได้เป็นพอ

## เพิ่มเมนู Blog ที่ Frontend

 เพิ่มเมนู blog ให้เราสามารถเข้าใช้งานได้ง่ายๆ โดยเพิ่ม เมนูที่ ``/frontend/views/layout/main.php` ในส่วนของ  $menuItems เพิ่มเมนู Blog เข้าไป

```php
$menuItems = [
              ['label' => 'Home', 'url' => ['/site/index']],
              ['label' => 'Blog', 'url' => ['/blog/index']],
              ['label' => 'About', 'url' => ['/site/about']],
              ['label' => 'Contact', 'url' => ['/site/contact']],
          ];
```


## สร้างกลุ่มสิทธ์หรือระดับการเข้าใช้งาน

 เราจะออกแบบ Role หรือระดับการเข้าใช้งาน ไว้ 3 กลุ่ม และแต่ละกลุ่มมีหน้าที่นังนี้

 - `Administrator` เป็นระดับสูงสุดทำได้ทุกอย่าง
 - `Moderator` เป็นกลุ่มที่มีหน้าที่ตรวจสอบข้อมูล
 - `User` เป็นกลุ่ม user ทั่วไป

 เราจะทำการสร้าง Role ไว้ที่ Model ​User ที่ `/common/models/User.php`

 ```php
 <?php

 class User extends ActiveRecord implements IdentityInterface
{
    const STATUS_DELETED = 0;
    const STATUS_ACTIVE = 10;

    const ROLE_USER = 10;
    const ROLE_MODERATOR = 20;
    const ROLE_ADMIN = 30;
    // ...
}
 ```

## ปรับปรุงโครงสร้างตาราง User
 ให้ทำการเพิ่มฟิวด์ชือว่า `role` เข้าไปที่ตาราง user  กำหนด type ข้อมูลเป็น `int` เพื่อเอาไว้เก็บสิทธิ์ของแต่ละคนว่าอยู่ระดับใด


## สร้าง user
 สร้าง user เพื่อทดสอบการใช้งาน RBAC โดยให้เราไปที่ frontend และไปที่หน้า signup user หรือหน้าสมัครสมาชิก และทำการลงทะเบียนไว้ 3 user กำหนด username และ password ตามด้านล่าง และแก้ไขข้อมูล role ใน table user ตามค่าที่ระบุด้านล่าง

- admin : 123456 กำหนด role = 30 เป็น admin
- mod   : 123456 กำหนด role = 20 เป็น moderator
- user  : 123456 กำหนด role = 10 เป็น user

![](/img/simple-rbac-signup.png)

จะได้ข้อมูลประมาณนี้

![](/img/simple-rbac-db.png)

## สร้าง AccessRule
 AccessRule เป็น filter และมีหน้าที่ตรวจสอบเวลาที่เราเรียกใช้งานในแต่ละ action โดยดึงข้อมูล user ที่ทำการ login และเช็คสิทธิ์จากฟิวด์ `role` ว่ามีสิทธิ์เข้าใช้งานหรือไม่

 สร้างโฟลเดอร `components` ที่ common แล้วสร้างไฟล์  `common\components\AccessRule.php` ตามนี้

 ```php
 <?php

namespace common\components;
use common\models\User;

class AccessRule extends \yii\filters\AccessRule{

	/**
	 * @inheritdoc
	 */
	protected function matchRole($user){

		if(empty($this->roles)){
			return true;
		}
		foreach ($this->roles as $role){

			if($role == '?' && $user->getIsGuest()){
					return true;
			}
			elseif($role == User::ROLE_USER && !$user->getIsGuest()){
					return true;
			}
			elseif(!$user->getIsGuest() && $role == $user->identity->role){
				return true;
			}
		}
		return false;
	}
}
 ```

## เรียกใช้งานที่ Controller

หลังจากที่เราได้สร้าง `AccessRule` ไปแล้ว เราจะเรียกใช้งานที่ `function behaviors()` ซึ่งปกติ หลังจาก gii crud มาแล้ว จะยังไม่มีการกำหนดสิทธิ์การเข้าใช้งาน และของเดิมที่ gii crud มา จะได้แบบนี้

 ```php
 <?php

namespace frontend\controllers;

use Yii;
use common\models\Blog;
use frontend\models\BlogSearch;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
use yii\filters\VerbFilter;

/**
 * BlogController implements the CRUD actions for Blog model.
 */
class BlogController extends Controller
{
    public function behaviors()
    {
        return [
            'verbs' => [
                'class' => VerbFilter::className(),
                'actions' => [
                    'delete' => ['post'],
                ],
            ],
        ];
    }

    // ...
}
 ```

 เราจะทำการกำหนดสิทธ์การใช้งาน เบื้องต้นเราได้กำหนด Role ไว้ที่ model User แล้วคือ

 - Admin
 - Moderator
 - User

 ใน Controller มี action `index`,`create`,`view`,`update`,`delete` เราจะแบ่งสิทธิ์การเข้าใช้งาน ดังนี้

- Action `index`,`create`,`view` สามารถใช้ได้ทั้ง 3 Role คือ `admin`,`moderator`,`user`
- Action `update` สามารถใช้งานได้เฉพาะ role  `admin`,`moderator`
- Action `delete` สามารถใช้งานได้เฉพาะ role `admin`

เวลาใช้งานก็แล้วแต่ว่า user จะได้รับ role อะไร แต่เราจะกำหนดค่า default Role ไว้เท่ากับ 10 ซึ่งก็คือ `ROLE_USER`

จากนั้นกำหนดค่าที่เราได้ออกแบบไว้ ที่ controller ใน `function behaviors()`

```php
<?php

namespace frontend\controllers;

use Yii;
use common\models\Blog;
use common\models\User;
use frontend\models\BlogSearch;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
use yii\filters\VerbFilter;
use yii\filters\AccessControl;
use common\components\AccessRule;

/**
 * BlogController implements the CRUD actions for Blog model.
 */
class BlogController extends Controller
{
public function behaviors(){
        return [
            'verbs' => [
                'class' => VerbFilter::className(),
                'actions' => [
                    'delete' => ['post'],
                ],
            ],
            'access'=>[
                'class'=>AccessControl::className(),
                'only'=> ['index','create','update','view','delete'],
                'ruleConfig'=>[
                    'class'=>AccessRule::className()
                ],
                'rules'=>[
                    [
                        'actions'=>['index','create','view'],
                        'allow'=> true,
                        'roles'=>[
                            User::ROLE_USER,
                            User::ROLE_MODERATOR,
                            User::ROLE_ADMIN

                        ]
                    ],
                    [
                        'actions'=>['update'],
                        'allow'=> true,
                        'roles'=>[
                            User::ROLE_MODERATOR,
                            User::ROLE_ADMIN
                        ]
                    ],
                    [
                        'actions'=>['delete'],
                        'allow'=> true,
                        'roles'=>[User::ROLE_ADMIN]
                    ]
                ]
            ]
        ];
    }
  // ...
}
```
ทดลอง login และเปลี่ยน เข้าใช้งานในแต่ละ user ดู หากไม่มีสิทธิ์เข้าใจงานจะเจอ error `Forbidden` แบบนี้ หลักๆ RBAC จะมีเพียงเท่านี้ ^ ^ ที่เหลือจะเป็นการปรับปรุงให้ระบบใช้งานได้ดียิ่งขึ้น

![](/img/simple-rbac-forbidden.png)


การสร้าง controller User เพื่อเอาไว้จัดการสิทธิ์ให้แต่ละ User
----
หลังจากที่เราได้สร้าง RBAC ไปแล้ว เราจะต้องสร้างหน้าจัดการผู้ใช้งาน เพื่อให้สามารถให้สิทธิ์ หรือแก้ไขสิทธิ์ของแต่ละ user ได้ และทำการสร้าง user, แก้ไข user ได้

## ปรับปรุงระบบลงทะเบียนใหม่ (Signup)

เราจะทำการปรับปรุงระบบลงทะเบียนใหม่ ให้มีการเพิ่ม  `role` default ให้กับ user ในตอนที่ลงทะเบียนด้วย เพื่อให้สามารถใช้งานได้ทันที

ปรับปรุงไฟล์ model Signup ที่  `application/frontend/model/SignupForm.php` ไปที่ `function signup`
เพิ่ม `$user->role = User::ROLE_USER;`

```php
<?php
namespace frontend\models;

use common\models\User;
use yii\base\Model;
use Yii;
/**
 * Signup form
 */
class SignupForm extends Model
{
// ...
    public function signup()
    {
        if ($this->validate()) {
            $user = new User();
            $user->username = $this->username;
            $user->email = $this->email;
            $user->setPassword($this->password);
            $user->generateAuthKey();
            $user->role = User::ROLE_USER; // <<--------
            if ($user->save()) {
                return $user;
            }
        }

        return null;
    }

// ...

}
```

จากนั้นทดสอบทำการลงทะเบียนใหม่ จะพบค่า role  default เท่ากับ 10 ซึ่งก็จะสามารถใช้งาน Blog ได้ทันที่

## สร้างระบบจัดการ User ที่ Backend

เราจะทำการสร้างระบบจัดการ user ที่ `Backend ``เพื่อให้สามารถปรับปรุงสิทธิ์ในแต่ละ user ได้ และสามารถเพิ่ม,ลบ,แก้ไข user ได้ โดยจะมีหน้าตาดังนี้

![](/img/simple-rbac-user.png)

![](/img/simple-rbac-update.png)

ปรับปรุง model User ที่  `/common/models/User.php`

ปรับปรุง  rules เพื่อให้สามารถ เพิ่ม,ลบ,แก้ไขได้ เพิ่มโค้ดส่วนนี้เข้าไป

```php
public function rules()
{
    return [
        ['status', 'default', 'value' => self::STATUS_ACTIVE],
        ['status', 'in', 'range' => [self::STATUS_ACTIVE, self::STATUS_DELETED]],

        ['role','integer'],
        ['role', 'default', 'value' => self::ROLE_USER],

        ['username', 'filter', 'filter' => 'trim'],
        ['username', 'required'],
        ['username', 'unique', 'targetClass' => '\common\models\User', 'message' => 'This username has already been taken.'],
        ['username', 'string', 'min' => 2, 'max' => 255],

        ['email', 'filter', 'filter' => 'trim'],
        ['email', 'required'],
        ['email', 'email'],
        ['email', 'unique', 'targetClass' => '\common\models\User', 'message' => 'This email address has already been taken.'],

        ['password', 'required'],
        ['password', 'string', 'min' => 6],
    ];
}
```

และเพิ่มโค้ดในการสร้าง itemAlias เพื่อให้สามารถดึงข้อมูลรายการไปแสดงผลที่ ActiveForm,GridView,detailView  ได้ง่ายๆ เพิ่มเติมเข้าไปที่ด้านล่างสุดของ Model User.php

```
public static function getItemsAlias($id){
    $items =  [
        'status'=>[
            self::STATUS_ACTIVE => 'Active',
            self::STATUS_DELETED => 'Deleted'
        ],
        'role' => [
            self::ROLE_ADMIN => 'Administrator',
            self::ROLE_MODERATOR => 'Moderator',
            self::ROLE_USER => 'User'
        ]
    ];
    return array_key_exists($id, $items) ? $items[$id] : [];
}

public function getItemStatus(){
    return self::getItemsAlias('status');
}

public function getStatusName(){
    $items =  $this->getItemStatus();
    return array_key_exists($this->status, $items) ? $items[$this->status] : null;
}

public function getItemRole(){
    return self::getItemsAlias('role');
}

public function getRoleName(){
    $items =  $this->getItemRole();
    return array_key_exists($this->role, $items) ? $items[$this->role] : null;
}
```

ผมคิดว่านี่คือ Pattern ที่ดีในการจัดการข้อมูล มันทำให้เราสามารถเรียกใช้งานได้ง่ายๆ ไม่ว่าจะเป็นการเรียกใช้งานเพื่อแสดงผล หรือ นำ items ไปแสดงที่ form แม้กระทั้ง filter ในส่วนของ Gridview  ก็สามารถทำได้ง่ายๆ

ข้อดีอีกอย่างเมื่อเราพัฒนาคนอื่นๆ สามารถมองโค้ดของเราเข้าใจ ทำให้สามารถพัฒนนาร่วมกันได้อย่างมีประสิทธิภาพ ตัวอย่างนี้อาจไม่ใช่ดีที่สุด แต่น่าจะเป็นจุดเริ่มต้นที่ดีในการออกแบบ เมื่อเราเริ่มจากจุดเล็กๆ รวมกันเข้ามันจะแข็งแกร่งอย่างไม่หน้าเชื่อ ลองมาดูแต่ละ function

### getItemsAlias()

เป็นฟังก์ชันรวม itemsAlias ต่างๆ ที่มีรายการไม่เยอะและไม่ได้นำไปสร้างเป็น table ผมจึงแนะนำให้สร้างแบบนี้

function ประกาศเป็น static เพื่อให้สามารถเรียกใช้ได้แม้ไม่ได้ new object ซึ่งจะรับค่าเป็น type ที่เราต้องการเรียกใช้ข้อมูล และ return ค่าออกมาเป็น array items ที่เราต้องการ

### getItemStatus()

เป็น function เพื่อเรียกใช้งาน items status เอาไว้ใช้ใน form เพื่อแสดงผลเป็น radioList,DropdownList ได้ สามารถเรียกใช้งานผ่าน `$model->getItemStatus()`

ตัวอย่าง

```php
<?= $form->field($model, 'status')->inline()->radioList($model->getItemStatus()) ?>
<?= $form->field($model, 'status')->inline()->dropdownList($model->getItemStatus()) ?>
```

### getStatusName(),getItemRole()

เป็น function ที่เอาไว้แสดงผล status,role ใน `GridView`,`DetailView` เพื่อให้เราเรียกใช้งานได้ง่ายๆ ซึ่งรูปแบบนี้เรียกว่า virtual attribute เป็นความสามารถที่มีมาตั้งแต่ Yii 1 แล้ว [อ่านเพิ่มเติมได้ที่นี่](http://www.yiiframework.com/wiki/167/understanding-virtual-attributes-and-get-set-methods/) หรืออ่านบทความที่ผมเขียนไว้[ที่นี่](/2014/11/30/relations.html)

ตัวอย่างการเรียกใช้งานสามารถเรียกแบบแสดงผลแค่ชื่อ หรือเพิ่มในส่วนของ filter เข้าไปได้ จะเห็นว่าเราสามารถเรียกใช้งานได้อย่างง่ายๆ

**Gridviw**

```php
<?php
use common\models\User;
?>
<?= GridView::widget([
        'dataProvider' => $dataProvider,
        'filterModel' => $searchModel,
        'columns' => [
            'statusName',
            [
              'attribute'=>'status',
              'filter'=>User::getItemsAlias('status'),
              'value'=>function($model){
                return $model->statusName;
              }
            ],
            'roleName',
            [
              'attribute'=>'role',
              'filter'=>User::getItemsAlias('role'),
              'value'=>function($model){
                return $model->roleName;
              }
            ],
        ],
    ]); ?>
```

**DetailView**

```php
<?= DetailView::widget([
    'model' => $model,
    'attributes' => [
        'statusName',
        'roleName',
    ],
]) ?>
```

**รวมโค้ดที่เพิ่มเข้าไปทั้งหมดใน model user**

> ผมได้เพิ่ม `public $password` เพื่อใช้ในการรับค่ารหัสผ่านจาก from  ซึ่งฟิวด์นี้จะไม่มีอยู่จริงใน table user หลังจากเรากรอก password ในform แล้ว model จะเรียยกใช้งานฟังก์ชัน setPassword() เพื่อแปลงค่าที่เรากรอกเข้าไปเป็นค่า has ที่ยาวๆ เพื่อเก็บใน db ต่อไป

```php
<?php
namespace common\models;

use Yii;
use yii\base\NotSupportedException;
use yii\behaviors\TimestampBehavior;
use yii\db\ActiveRecord;
use yii\web\IdentityInterface;

/**
 * User model
 *
 * @property integer $id
 * @property string $username
 * @property string $password_hash
 * @property string $password_reset_token
 * @property string $email
 * @property string $auth_key
 * @property integer $status
 * @property integer $created_at
 * @property integer $updated_at
 * @property string $password write-only password
 */
class User extends ActiveRecord implements IdentityInterface
{
    const STATUS_DELETED = 0;
    const STATUS_ACTIVE = 10;

    const ROLE_USER = 10;
    const ROLE_MODERATOR = 20;
    const ROLE_ADMIN = 30;

    public $password;

    /**
     * @inheritdoc
     */
    public static function tableName()
    {
        return '{{%user}}';
    }

    /**
     * @inheritdoc
     */
    public function behaviors()
    {
        return [
            TimestampBehavior::className(),
        ];
    }

    /**
     * @inheritdoc
     */
    public function rules()
    {
        return [
            ['status', 'default', 'value' => self::STATUS_ACTIVE],
            ['status', 'in', 'range' => [self::STATUS_ACTIVE, self::STATUS_DELETED]],

            ['role','integer'],
            ['role', 'default', 'value' => self::ROLE_USER],

            ['username', 'filter', 'filter' => 'trim'],
            ['username', 'required'],
            ['username', 'unique', 'targetClass' => '\common\models\User', 'message' => 'This username has already been taken.'],
            ['username', 'string', 'min' => 2, 'max' => 255],

            ['email', 'filter', 'filter' => 'trim'],
            ['email', 'required'],
            ['email', 'email'],
            ['email', 'unique', 'targetClass' => '\common\models\User', 'message' => 'This email address has already been taken.'],

            ['password', 'required'],
            ['password', 'string', 'min' => 6],
        ];
    }

    /**
     * @inheritdoc
     */
    public static function findIdentity($id)
    {
        return static::findOne(['id' => $id, 'status' => self::STATUS_ACTIVE]);
    }

    /**
     * @inheritdoc
     */
    public static function findIdentityByAccessToken($token, $type = null)
    {
        throw new NotSupportedException('"findIdentityByAccessToken" is not implemented.');
    }

    /**
     * Finds user by username
     *
     * @param string $username
     * @return static|null
     */
    public static function findByUsername($username)
    {
        return static::findOne(['username' => $username, 'status' => self::STATUS_ACTIVE]);
    }

    /**
     * Finds user by password reset token
     *
     * @param string $token password reset token
     * @return static|null
     */
    public static function findByPasswordResetToken($token)
    {
        if (!static::isPasswordResetTokenValid($token)) {
            return null;
        }

        return static::findOne([
            'password_reset_token' => $token,
            'status' => self::STATUS_ACTIVE,
        ]);
    }

    /**
     * Finds out if password reset token is valid
     *
     * @param string $token password reset token
     * @return boolean
     */
    public static function isPasswordResetTokenValid($token)
    {
        if (empty($token)) {
            return false;
        }
        $expire = Yii::$app->params['user.passwordResetTokenExpire'];
        $parts = explode('_', $token);
        $timestamp = (int) end($parts);
        return $timestamp + $expire >= time();
    }

    /**
     * @inheritdoc
     */
    public function getId()
    {
        return $this->getPrimaryKey();
    }

    /**
     * @inheritdoc
     */
    public function getAuthKey()
    {
        return $this->auth_key;
    }

    /**
     * @inheritdoc
     */
    public function validateAuthKey($authKey)
    {
        return $this->getAuthKey() === $authKey;
    }

    /**
     * Validates password
     *
     * @param string $password password to validate
     * @return boolean if password provided is valid for current user
     */
    public function validatePassword($password)
    {
        return Yii::$app->security->validatePassword($password, $this->password_hash);
    }

    /**
     * Generates password hash from password and sets it to the model
     *
     * @param string $password
     */
    public function setPassword($password)
    {
        $this->password_hash = Yii::$app->security->generatePasswordHash($password);
    }

    /**
     * Generates "remember me" authentication key
     */
    public function generateAuthKey()
    {
        $this->auth_key = Yii::$app->security->generateRandomString();
    }

    /**
     * Generates new password reset token
     */
    public function generatePasswordResetToken()
    {
        $this->password_reset_token = Yii::$app->security->generateRandomString() . '_' . time();
    }

    /**
     * Removes password reset token
     */
    public function removePasswordResetToken()
    {
        $this->password_reset_token = null;
    }


    public static function getItemsAlias($id){
        $items =  [
            'status'=>[
                self::STATUS_ACTIVE => 'Active',
                self::STATUS_DELETED => 'Deleted'
            ],
            'role' => [
                self::ROLE_ADMIN => 'Administrator',
                self::ROLE_MODERATOR => 'Moderator',
                self::ROLE_USER => 'User'
            ]
        ];
        return array_key_exists($id, $items) ? $items[$id] : [];
    }

    public function getItemStatus(){
        return self::getItemsAlias('status');
    }

    public function getStatusName(){
        $items =  $this->getItemStatus();
        return array_key_exists($this->status, $items) ? $items[$this->status] : null;
    }

    public function getItemRole(){
        return self::getItemsAlias('role');
    }

    public function getRoleName(){
        $items =  $this->getItemRole();
        return array_key_exists($this->role, $items) ? $items[$this->role] : null;
    }
}

```

## Gii CRUD User

หลังจากที่เตรียม model user เสร็จเรียบร้อย ให้เราไปที่ Backend และทำการ gii crud User
และตั้งค่าดังนี้

![](/img/simple-gii-crud-user.png)

ทดลองคร่าวๆ ว่าเข้าใช้งานได้

## เพิ่มเมนู User Management ที่ Backend

 เพิ่มเมนู User Managemen ให้เราสามารถเข้าใช้งานได้ง่ายๆ โดยเพิ่ม เมนูที่ ``/backend/views/layout/main.php` ในส่วนของ  $menuItems เพิ่มเมนู User Managemen เข้าไป

```php
$menuItems = [
              ['label' => 'Home', 'url' => ['/site/index']],
              ['label' => 'User Management', 'url' => ['/user/index']],
          ];
```

## ปรับปรุง UserController

ปรับปรุง UserController เพื่อให้สามารถเพิ่ม,ลบ,แก้ไข ข้อมูล user ได้ ผ่าน backend โดยไม่ต้อง register จาก frontend

ปรับปรุง Rule ก่อน เพื่อกำหนดให้เฉพาะ admin เท่านั้นที่สามารถจัดการข้อมูล user ได้

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
            ],
        ],
        'access'=>[
            'class'=> AccessControl::className(),
            'only' =>['index','view','create','update','delete'],
            'ruleConfig'=> [
                'class' => AccessRule::className()
            ],
            'rules'=>[
                [
                   'roles' => [
                        User::ROLE_ADMIN
                    ],
                   'allow' => true,
                ]
            ]
        ]
    ];
}
```


ปรับปรุง action create,update  ใน `/backend/controllers/UserController.php `เพื่อให้ระบบแปลงข้อมูล password ที่กรอกเข้ามาให้อยู่ในรูปแบบ has ก่อนนำไปบันทึกข้อมูล  แก้ไขเข้าไปดังนี้

> คิดว่าดูโค้ดแล้วน่าจะพอเข้าใจ

```php
<?php
// ...

/**
 * Creates a new User model.
 * If creation is successful, the browser will be redirected to the 'view' page.
 * @return mixed
 */
public function actionCreate()
{
    $model = new User();

    if ($model->load(Yii::$app->request->post()) && $model->validate()) {

        $model->setPassword($model->password);
        $model->generateAuthKey();
        $model->save();

        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('create', [
            'model' => $model,
        ]);
    }
}

/**
 * Updates an existing User model.
 * If update is successful, the browser will be redirected to the 'view' page.
 * @param integer $id
 * @return mixed
 */
public function actionUpdate($id)
{
    $model = $this->findModel($id);
    $model->password = $model->password_hash;

    if ($model->load(Yii::$app->request->post()) && $model->validate()) {
        if($model->password_hash!=$model->password ){
             $model->setPassword($model->password);
        }
        $model->save();
        return $this->redirect(['view', 'id' => $model->id]);
    } else {
        return $this->render('update', [
            'model' => $model,
        ]);
    }
}
```

ในส่วนของ Gridview ที่ `index.php`

```php
<?= GridView::widget([
        'dataProvider' => $dataProvider,
        'filterModel' => $searchModel,
        'columns' => [
          //  ['class' => 'yii\grid\SerialColumn'],

            'id',
            'username',
            // 'auth_key',
            // 'password_hash',
            // 'password_reset_token',
            'email:email',

            //'created_at',

            'updated_at:dateTime',
            //'roleName',
            [
              'attribute'=>'role',
              'filter'=>User::getItemsAlias('role'),
              'value'=>function($model){
                return $model->roleName;
              }
            ],
            //'statusName',
            [
              'attribute'=>'status',
              'filter'=>User::getItemsAlias('status'),
              'value'=>function($model){
                return $model->statusName;
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
```

ในส่วนของ DetailView ที่ `view.php`

```php
<?= DetailView::widget([
      'model' => $model,
      'attributes' => [
          'id',
          'username',
          'email:email',
          'statusName',
          'created_at:dateTime',
          'updated_at:dateTime',
          'roleName',
      ],
  ]) ?>
```

ในส่วนของ form  ที่ `_from.php`

```php
<?php

use yii\helpers\Html;
use yii\bootstrap\ActiveForm;

/* @var $this yii\web\View */
/* @var $model common\models\User */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="user-form">

    <?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'username')->textInput(['maxlength' => true]) ?>
    <?= $form->field($model, 'password')->passwordInput(['maxlength' => true]) ?>

    <?= $form->field($model, 'email')->textInput(['maxlength' => true]) ?>

	<div class="row">
		<div class="col-xs-6">
			<?= $form->field($model, 'status')->inline()->radioList($model->getItemStatus()) ?>
		</div>
		<div class="col-xs-6">
			<?= $form->field($model, 'role')->inline()->radioList($model->getItemRole()) ?>
		</div>
	</div>

    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? Yii::t('app', 'Create') : Yii::t('app', 'Update'), ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>

```

## สรุป

นี่เป็นเพียงจุดเริ่มต้นในการใช้ RBAC อย่างง่ายๆ เมื่อเราพัฒนาระบบ เราต้องออกแบบและคิดภาพรวมในหัวก่อนว่า ระบบของเรานั้น แบ่งกลุ่มผู้ใช้งานเป็นกี่กลุ่ม แต่ละกลุ่มสามารถใช้งานอะไรได้บ้าง แล้วค่อยทำการสร้าง role และแบ่งกลุ่มผู้ใช้งานขึ้นมา

ตัวอย่างนี้จึงเหมาะสำหรับระบบที่ไม่ได้มีการใช้งานที่ซับซ้อนอะไร และข้อกัดอีก 1 อย่างนี้ก็คือ ไม่สามารถ ให้สิทธ์ user ได้มากกว่า 1 กลุ่ม ซึ่งหากเป็นระบบใหญ่ๆ และซับซ้อนเราจะใช้ RBAC ที่เก็บข้อมูลใน DB แทนค่อยติดตามในบทต่อๆ ไปครับ

หวังว่าคงจะทำให้เราเข้าใจในพื้นฐานการใช้งาน RBAC บ้างนะครับ

> ปล. บทความนี้ใช้เวลา 2 วันในการเขียนและทดลองทำตัวอย่างขึ้นมา นึกว่าวันเดียวจะเสร็จฮ่าๆ หากเจอบั๊กหรือติดอะไรที่ผมเขียนไม่ครบ คอมเม้นไว้ด้านล่างได้เลยครับ

ดาวน์โหลดตัวอย่างโค้ด Simple-RBAC [ได้ที่นี่](https://github.com/dixonsatit/yii2-workshop-simple-rbac)

[ที่มา](http://code.tutsplus.com/tutorials/programming-with-yii2-user-access-controls--cms-23173)
