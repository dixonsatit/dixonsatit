---
layout: post
title: สร้างลิ้งเชื่อมกันระหว่าง frontend & backend
---

ในกรณที่เราสร้าง aplication แบบ advance ตัวโปรเจคจะถูกแบ่งออกเป็น 2 ส่วนคือ backend, frontend ซึ่งเราสามารถสร้าง link เชื่อมกันระหว่าง backend & frontend โดยใช้ `urlManager`

## Frontend ลิ้งไปที่ Backend
แก้ไปไฟล์ frontend/config/main.php  เพิ่ม `urlManagerBackend` เข้าไป

```php
<?php
'component'=>[
	...........
	 'urlManager' => [
            'class' => 'yii\web\urlManager',
            'enablePrettyUrl' => false,
            'showScriptName' => true,
     ],
     'urlManagerBackend' => [
            'class' => 'yii\web\urlManager',
            'baseUrl' => '/projectName/backend/web',
            'scriptUrl'=>'/projectName/backend/web/index.php',
            'enablePrettyUrl' => false,
            'showScriptName' => true,
     ],
    ...........
 ]
 ?>
```

### เรียกใช้งาน
```php
<?php
echo Yii::$app->urlManagerBackend->createAbsoluteUrl(['site/index','id'=>4]);
echo Yii::$app->urlManagerBackend->createUrl(['site/index','id'=>4]);
echo Yii::$app->urlManagerBackend->getBaseUrl();
echo Yii::$app->urlManagerBackend->getHostInfo();
echo Yii::$app->urlManagerBackend->getScriptUrl();
?>
```

## Backend  ลิ้งไปที่ Frontend
แก้ไปไฟล์ backend/config/main.php  เพิ่ม `urlManagerFrontend` เข้าไป

```php
<?php
'component'=>[
	...........
	 'urlManager' => [
            'class' => 'yii\web\urlManager',
            'enablePrettyUrl' => false,
            'showScriptName' => true,
     ],
     'urlManagerFrontend' => [
            'class' => 'yii\web\urlManager',
            'baseUrl' => '/projectName/frontend/web',
            'scriptUrl'=>'/projectName/frontend/web/index.php',
            'enablePrettyUrl' => false,
            'showScriptName' => true,
     ],
    ...........
 ]
 ?>
```

### เรียกใช้งาน

```php
<?php
echo Yii::$app->urlManagerFrontend->createAbsoluteUrl(['site/index','id'=>4]);
echo Yii::$app->urlManagerFrontend->createUrl(['site/index','id'=>4]);
echo Yii::$app->urlManagerFrontend->getBaseUrl();
echo Yii::$app->urlManagerFrontend->getHostInfo();
echo Yii::$app->urlManagerFrontend->getScriptUrl();
?>
```
