---
layout : post
title : การใช้งาน FlashMessage + Growl Widget
---

หากคุณเคยใช้ flashMessage มาบ้างแล้ว หากยังไม่เคยใช้ [ลองอ่านที่นี่](/2015/06/17/flash-message.html) คุณอาจจะรูสึกว่ามันไม่ค่อยสวยเลย วันนี้ผมมี `yii2-widget-growl` มาแนะนำ ซึ่งตัวนี้สวยงามและน่าใช้เลยทีเดียว

ก่อนอื่นไปติดตั้งกันก่อน [yii2-widget-growl](https://github.com/kartik-v/yii2-widget-growl) 

```
composer  require kartik-v/yii2-widget-growl "*"
```

## กำหนดค่า

```php
<?php 
    Yii::$app->getSession()->setFlash('alert1', [
        'type' => 'success',
        'duration' => 12000,
        'icon' => 'fa fa-users',
        'title' => Yii::t('app', Html::encode('Submission')),
        'message' => Yii::t('app',Html::encode('บันทึกข้อมูลเสร็จเรียบร้อย')),
        'positonY' => 'top',
        'positonX' => 'right'
    ]);

     Yii::$app->getSession()->setFlash('alert2', [
        'type' => 'warning',
        'duration' => 12000,
        'icon' => 'fa fa-users',
        'title' => Yii::t('app',Html::encode('Submission')),
        'message' => Yii::t('app', Html::encode('นำเข้าข้อมูลเร็จเรียบร้อย..')),
        'positonY' => 'top',
        'positonX' => 'right'
    ]);
?>

```


## การแสดงผล

```php
<?php 
use Yii;
use yii\helpers\Html;
 
//Get all flash messages and loop through them
<?php foreach (Yii::$app->session->getAllFlashes() as $message):; ?>
<?php
echo \kartik\widgets\Growl::widget([
    'type' => (!empty($message['type'])) ? $message['type'] : 'danger',
    'title' => (!empty($message['title'])) ? Html::encode($message['title']) : 'Title Not Set!',
    'icon' => (!empty($message['icon'])) ? $message['icon'] : 'fa fa-info',
    'body' => (!empty($message['message'])) ? Html::encode($message['message']) : 'Message Not Set!',
    'showSeparator' => true,
    'delay' => 1, //This delay is how long before the message shows
    'pluginOptions' => [
        'delay' => (!empty($message['duration'])) ? $message['duration'] : 3000, //This delay is how long the message shows for
        'placement' => [
            'from' => (!empty($message['positonY'])) ? $message['positonY'] : 'top',
            'align' => (!empty($message['positonX'])) ? $message['positonX'] : 'right',
        ]
    ]
]);
?>
<?php endforeach; ?>
```
![](/img/flashmessage-glow.png)