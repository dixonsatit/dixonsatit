---
layout: post
title: เปลี่ยน สไตล์ ActionColumn เดิมๆ เป็น Bootstrap ButtonGroup
---

เห็น button group ใน bootstrap มันสวยดีก็เลยจับมาใส่ใน actionColumn  ซักหน่อย ของเดิมดูยาก บางทีมองแทบไม่ออกว่ามันคือปุ่ม

ตัวอย่าง

![](/img/action-column/action-column.png)

{% highlight php %}
<?= GridView::widget([
     'dataProvider' => $dataProvider,
     'filterModel' => $searchModel,
     'columns' => [
         ['class' => 'yii\grid\SerialColumn'],
         //.......... someting column
         [
            'class' => 'yii\grid\ActionColumn',
            'options'=>['style'=>'width:120px;'],
            'buttonOptions'=>['class'=>'btn btn-default'],
            'template'=>'<div class="btn-group btn-group-sm text-center" role="group"> {view} {update} {delete} </div>'
         ],
    ]
 ]);?>
{% endhighlight %}
