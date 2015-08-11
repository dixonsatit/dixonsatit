---
layout : post
title : การใช้งาน GridView หลายๆ ตัวในหน้าเดียวกัน
---

ปกติส่วนใหญ่เราก็จะใช้ GridView เพียงแค่ตัวเดียวต่อ 1 page แต่ถ้าเมื่อไหร่เราใช้ 2 GridView แล้วละก็ ปัญหามันจะเกิด ตอนทีเราคลิกเลือก page ในตอนแบ่งหน้าหรือตอนคลิก sort บนหัวคอลัมน์ เราจะต้องตั้งค่าตัวแปรให้ `pagination`,`sort` ใน GridView ใหม่ เพราะไม่งั้นค่ามันจะเป็นตัวเดียวกัน GridView อาจจะเอ๋อๆ ได้

```php
<?php  

use yii\grid\GridView;

$userProvider->pagination->pageParam = 'user-page';
$userProvider->sort->sortParam = 'user-sort';

$postProvider->pagination->pageParam = 'post-page';
$postProvider->sort->sortParam = 'post-sort';

echo '<h1>Grid Users</h1>';
echo GridView::widget([
    'dataProvider' => $userProvider,
]);

echo '<h1>Grid Posts</h1>';
echo GridView::widget([
    'dataProvider' => $postProvider,
]);
```

แค่นี้เราก็สามารถใช้งาน gridview หลายๆ ตัวใน page เดียวได้แล้วครับ

@dixonSatit
