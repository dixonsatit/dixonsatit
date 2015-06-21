---
layout : post
title : การสร้าง Theme
---

เชื่อว่าหลายๆ คนอาจจะไม่ชอบธีมเดิมๆ ที่ Yii 2  มีมาให้ ซึ่งเป็น [Bootstrap](http://getbootstrap.com) ส่วนตัวผมเองมองว่ามันก็สามารถใช้งานได้ดี และมีคอมโพเน็นส์มาให้ครบ แทบจะไม่ต้องหาอะไรเพิ่ม แต่การใช้งานจริงๆ เราคงไม่สามารถใช้แค่ธีม Bootstrap แบบเดิมๆ ได้ (มันคงเบื่อกันบ้างแหละ ^^ )

วันนี้จะแนะนำการเปลี่ยนธิมของ Yii 2 ซึ่งทำได้ง่ายมาก แค่เปลี่ยนคอนฟิกใน main.php ในส่วนของ components -> view ให้มันชี้ไปที่พาทที่เราเก็บธีมของเราไว้ (แค่นี้เองหรอ ดูเหมือนง่าย ง่ายเฉพาะคอนฟิกครับ ฮา..)

```php
'view' => [
     'theme' => [
         'pathMap' => [
            '@app/views' => '@app/themes/adminlte'
         ],
     ],
],

```

## หลัการทำงาน

หลักการทำงานของ view ใน Yii 2 เมื่อ Web Application ของเราทำงานมันจะไป render ไฟล์ view ที่โฟลเดอร์ดีฟอลด์ views ก่อน
จากนั้นก็จะเช็คว่ามีการเรียกใช้งานที่คอนโทรเลอร์ใหน มันก็จะไปค้นที่โฟล์เดอร์ที่มีชื่อตามคอนโทรเลอร์นั้นในไฟลเดอร์ views

ในการสร้างธีม เราสามารถตั้งค่าให้มันเปลี่ยนเส้นทางของการเรียกใช้งาน view จากปกติจะค้นทีโฟลเดอร์ views หรือ @app/views นั้น เราก็เปลี่ยนให้มันไปเรียกใช้งานที่โฟล์เดอร์ธีมของเราเป็นอันดับแรกแทน 

หากมีการเรียกใช้งาน view ใหน และไปค้นในธีมที่เราตั้งค่าไว้ไม่พบ มันจะกลับมาเรียกใช้งานที่พาทดีฟอลด์เดิมคือ views หรือ @app/views นั่นเอง


## ลองสร้าง Theme กัน

ตัวอย่างนี้เป็นการนำ Html Template มาสร้างเป็นธีมสำหรับ Yii 2 เราใช้ [agency](http://startbootstrap.com/template-overviews/agency/) ซึ่งเป็นธีมที่สร้างมาจาก bootstrap อยู่แล้ว และที่สำคัญมันฟรี 

![](/img/agency-theme.png)

> แนะนำให้ใช้ Theme ที่สร้างมาจาก Bootstrap เพราะจะเข้ากันได้กับ Yii2 ไม่ต้องแก้อะไรเยอะ แต่ถ้าหากคุณเข้าใจเรื่อง css อยู่แล้วก็คงไม่เป็นปัญหาอะไร แต่ถ้าหาคุณยังไม่เก่งแนะนำใช้ธีมที่สร้างมาจาก Bootstrap จะไม่ต้องแก้ css เยอะ

ก่อนอื่นทำการดาวน์โหลดไฟล์ธีม [agency](http://startbootstrap.com/template-overviews/agency/) มาก่อน จากนั้นแตกซิบไฟล์ จะพบกับไฟล์ดังนี้

![](/img/agency.png)

เราจะต้องสร้างโฟลเดอร์เพื่อเก็บธีม โดยสร้างโฟลเดอร์ชื่อ themes/agency ที่ root โปรเจ็ค จากนั้นสร้างโฟลเดอร์ใน agency ดังนี้

 - assets
 - views
 - dist

`assets` เอาไว้เก็บ Asset Bundle ที่เราสร้างขึ้น
`views` เก็บ layouts และ view ทั้งหมด
`dist` เก็บไฟล์ css,js,images

- copy โฟลเดอร์ css,img,js จาก agency theme ไว้ที่ dist
- copy โฟลเดอร์ layouts จาก /views  มาไว้ที่ /themes/agency/views/

เราจะไม่เอาโฟลเดอร์ font เพราะ Yii2 มี Bootstrap  อยู่แล้ว และ font-awesome เราจะลองสร้าง asset เอง หรือใครอยากใช้ [rmrevin/yii2-fontawesome](Asset Bundle for Yii2 with Font Awesome ) แทน ซึ่งเป็น Asset Bundle สำหรับ font-awesome  ไว้แล้วแค่ติดตั้งก็สามารถทำการเรียกใช้ได้เลย (สบาย..)

จากนั้นจะได้โครงสร้างตามภาพ

![](/img/agency-folder.png)


## สร้าง AgencyAsset

[อ่านการการสร้าง asset เพิ่มเติมได้ที่นี่](/2015/06/20/create-assets.html)

เราจะทำการสร้าง aliase ที่ `config/main.php` เพื่อชี้ไปยังโฟลเดอร์ธีมของเราให้สามารถเรียกใช้งานได้ง่าย ผ่าน namespace เพราะถ้าพาทเต็มๆ จะเป็น @app/themes/agency แต่ถ้าสร้าง aliase แล้ว ก็จะสามารถเรียกแค่ @agency สั้นๆได้

```php
$config = [
    'id' => 'basic',
    //....
    'aliases'=>[
        '@agency' => '@app/themes/agency',
    ],
    // ....
]
```
เราสร้าง Asset ชื่อ  AgencyAsset ไว้ที่ `/themes/agency/assets/AgencyAsset.php `เพื่อ include ไฟล์ css,js มาใช้งาน โดยเราดูจากธีมว่าเค้าเรียกใช้อะไรบ้าง ก็ทำการเรียกตามนั้นให้ครบ

```php
<?php
namespace agency\assets;

use yii\web\AssetBundle;

class AgencyAsset extends AssetBundle
{
    public $sourcePath = '@agency/dist'; 
    public $css = [
        'css/agency.css',
        '//fonts.googleapis.com/css?family=Montserrat:400,700',
        '//fonts.googleapis.com/css?family=Kaushan+Script',
        '//fonts.googleapis.com/css?family=Droid+Serif:400,700,400italic,700italic',
        '//fonts.googleapis.com/css?family=Roboto+Slab:400,100,300,700'
    ];
    
    public $js = [
        '//cdnjs.cloudflare.com/ajax/libs/jquery-easing/1.3/jquery.easing.min.js',
        'js/jquery-scrollspy.js',
        //'js/cbpAnimatedHeader.min.js',
        'js/classie.js',
        'js/contact_me.js',
        'js/jqBootstrapValidation.js',
        'js/agency.js',
    ];

    public $depends = [
        'yii\web\YiiAsset',
        'yii\bootstrap\BootstrapAsset',
        'agency\assets\FontAwesomeAsset'
    ];
}


```
`cbpAnimatedHeader.min.js` ผมยังไม่เรียกเพราะจะเรียกใช้เพียงแค่เฉพาะ homepage เท่านั้น เดี๋ยวไปเรียกอีกทีภายหลัง

## สร้าง FontAwesomeAsset

 ไฟล์ css ของ font-awesome เราจะเรียกใช้จากแพคเกจของ Bower คือ font-awesome โดยติดตั้งที่ composer.json และเพิ่ม ` "bower-asset/fontawesome": "4.3.*"` ในส่วน require เข้าไป 

จากนั้นรันคำสั่ง `composer update` เพื่อติดตั้งแพคเกจ

สร้าง FontAwesomeAsset ไว้ที่ ` /themes/agency/assets/FontAwesomeAsset.php`

```php
<?php
namespace agency\assets;

use yii\web\AssetBundle;

class FontAwesomeAsset extends AssetBundle 
{
    public $sourcePath = '@bower/fontawesome'; 
    public $css = [ 
        'css/font-awesome.min.css', 
    ];
    public $publishOptions = [
        'only' => [
            'fonts/',
            'css/',
        ]
    ];
}
```

## สร้าง layout

สร้าง layout `_base.php`

ที่ /themes/agency/_base.php และทำการเรียกใช้งาน `AgencyAsset::register($this)` ที่เราได้สร้างไว้

```php
<?php
use yii\helpers\Html;
use yii\bootstrap\Nav;
use yii\bootstrap\NavBar;
use yii\widgets\Breadcrumbs;
use agency\assets\AgencyAsset;

/* @var $this \yii\web\View */
/* @var $content string */

AgencyAsset::register($this);
$directoryAsset = Yii::$app->assetManager->getPublishedUrl('@agency/dist');
?>
<?php $this->beginPage() ?>
<!DOCTYPE html>
<html lang="<?= Yii::$app->language ?>">
<head>
    <meta charset="<?= Yii::$app->charset ?>">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <?= Html::csrfMetaTags() ?>
    <title><?= Html::encode($this->title) ?></title>
    <?php $this->head() ?>
</head>
<body id="page-top" class="index">
<?php $this->beginBody() ?>

<?= $content ?>

<?php $this->endBody() ?>
</body>
</html>
<?php $this->endPage() ?>

```
สร้าง layout `_header.php`

```php
   <!-- Navigation -->
   <?php
   use yii\helpers\Html;
   use yii\bootstrap\Nav;
   use yii\bootstrap\NavBar;
   use yii\helpers\ArrayHelper;
    $class = !isset($class)?'':$class;
    if(Yii::$app->layout == 'homepage'){
        $menus = [
        ['label' => 'Home', 'url' => ['/site/index']],
            ['label' => 'Service', 'url' =>'#services','linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'Portfolio', 'url' =>'#portfolio','linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'About', 'url' =>'#about','linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'Team', 'url' =>'#team','linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'Contact', 'url' =>'#contact','linkOptions'=>['class'=>'page-scroll']],
        ];
    }else{
          $menus = [
          ['label' => 'Home', 'url' => ['/site/index']],
            ['label' => 'Service', 'url' =>['index','#'=>'services'],'linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'Portfolio', 'url' =>['index','#'=>'portfolio'],'linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'About', 'url' =>['index','#'=>'about'],'linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'Team', 'url' =>['index','#'=>'team'],'linkOptions'=>['class'=>'page-scroll']],
            ['label' => 'Contact', 'url' =>['index','#'=>'contact'],'linkOptions'=>['class'=>'page-scroll']],
        ];
    }
   ?>

<?php
    $options = ['navbar','navbar-default','navbar-fixed-top'];
    NavBar::begin([
        'brandLabel' => 'Yii 2 Learning',
        'brandUrl' => Yii::$app->homeUrl,
        'brandOptions'=>[
            'class'=>'navbar-header page-scroll'
        ],
        'options' => [
            'class' => 'navbar navbar-default navbar-fixed-top '.$class,
        ],
    ]);
    echo Nav::widget([
        'options' => ['class' => 'navbar-nav navbar-right'],
        'items' =>ArrayHelper::merge($menus,
           [
            
           
            ['label' => 'Demo', 'items'=>[
                ['label' => 'About', 'url' => ['/site/about']],
                ['label' => 'Contact', 'url' => ['/site/contact']],
            ]],
            Yii::$app->user->isGuest ?
                ['label' => 'Login', 'url' => ['/site/login']] :
                ['label' => 'Logout (' . Yii::$app->user->identity->username . ')',
                    'url' => ['/site/logout'],
                    'linkOptions' => ['data-method' => 'post']],
        ]),
    ]);
    NavBar::end();
?>
```

สร้าง layout `_footer.php`

```php
   <footer>
        <div class="container">
            <div class="row">
                <div class="col-md-4">
                    <span class="copyright">Copyright &copy; Your Website 2014</span>
                </div>
                <div class="col-md-4">
                    <ul class="list-inline social-buttons">
                        <li><a href="#"><i class="fa fa-twitter"></i></a>
                        </li>
                        <li><a href="#"><i class="fa fa-facebook"></i></a>
                        </li>
                        <li><a href="#"><i class="fa fa-linkedin"></i></a>
                        </li>
                    </ul>
                </div>
                <div class="col-md-4">
                    <ul class="list-inline quicklinks">
                        <li><a href="#">Privacy Policy</a>
                        </li>
                        <li><a href="#">Terms of Use</a>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </footer>
```

สร้าง layout `main.php`

```php
<?php $this->beginContent('@agency/views/layouts/_base.php'); ?>
<?= $this->render('_header.php',['class'=>'navbar-shrink']) ?>

 <section class="pages" >
        <div class="container">
           <?= $content ?>
        </div>
</section>


<?= $this->render('_footer.php') ?>

<?php $this->endContent(); ?>
```

สร้าง layout `page_header.php`

```php

<?php $this->beginContent('@agency/views/layouts/_base.php'); ?>

<?= $this->render('_header.php') ?>

<?= $content ?>

<?= $this->render('_footer.php') ?>

<?php $this->endContent(); ?>
```

สร้าง layout `homepage.php`

```php
<?php
$directoryAsset = Yii::$app->assetManager->getPublishedUrl('@agency/dist');
$this->registerJsFile($directoryAsset.'/js/cbpAnimatedHeader.min.js');
?>
<?php $this->beginContent('@agency/views/layouts/_base.php'); ?>

<?= $this->render('_header.php') ?>

<?= $content ?>

<?= $this->render('_footer.php') ?>

<?php $this->endContent(); ?>
```

ที่ต้องแยกไฟล์ view ออกมาเยอะเช่นนี้ก็เพื่อให้เราสามารถแก้ไขไฟล์แต่ละส่วนได้อย่างสะดวกโดยแต่ละไฟล์มีหน้าที่ต่างกัน

layout สร้างไว้เพื่อเรียกใช้งาน

- `_base.php` เป็นไฟล์ทีเก็บโครงสร้าง html และเรียกใช้งาน js,css ทั้งหมด
- `_header.php` ส่วนของเมนู
- `_footer.php` ส่วนของ footer

layout หลักที่สามารถเรียกใช้งานผ่าน  `controller` โดยจะเรียกใช้งาน view  ด้านบนและปรับแต่ง html ในส่วน layout ตามต้องการ

- `main.php` เป็น layout หลักปกติ Yii2 จะเรียก layout นี้เป็นหลักเว้นแต่ว่าเรากำหนดให้เรียก view อื่นๆ
- `homepage.php` เป็น layout ที่ผมสร้างไว้สำหรับหน้าหลัก
- `page_header.php` เป็น layout สำหรับหน้าต่างๆ 



## สร้างหน้า Homepage

เราจะสร้าง view index.php ไว้ที่ /themes/agency/views/site/index.php เพื่อให้เรียกใช้งาน views/site/index.php ที่ theme  ก่อน และทำการแยก view ในแต่ละส่วนออกเป็นไฟล์และทำการเรียกใช้งาน

```php
   <?php

    Yii::$app->layout='homepage';

   $directoryAsset = Yii::$app->assetManager->getPublishedUrl('@agency/dist');
   ?>
   <!-- Header -->
    <header>
        <div class="container">
            <div class="intro-text">
                <div class="intro-lead-in">Welcome To Our Studio!</div>
                <div class="intro-heading">It's Nice To Meet You</div>
                <a href="#services" class="page-scroll btn btn-xl">Tell Me More</a>
            </div>
        </div>
    </header>

    <?= $this->render('_service.php',['directoryAsset'=>$directoryAsset ]) ?>
    <?= $this->render('_portfolio.php',['directoryAsset'=>$directoryAsset ]) ?>
    <?= $this->render('_about.php',['directoryAsset'=>$directoryAsset ]) ?>
    <?= $this->render('_team.php',['directoryAsset'=>$directoryAsset ]) ?>
    <?= $this->render('_client.php',['directoryAsset'=>$directoryAsset ]) ?>
    <?= $this->render('_contact.php',['directoryAsset'=>$directoryAsset ]) ?>

```

ทำการสร้าง view ที่ index.php เรียกใช้ จริงๆ เราสามารถเอา html ทั้งหมดมาไว้ที่ index ได้แต่ผมอยากแยกแต่ละส่วนเพื่อให้สามารถแก้ไขได้ง่ายๆ


ไฟล์ `_service.php`

```html
    <!-- Services Section -->
    <section id="services">
        <div class="container">
            <div class="row">
                <div class="col-lg-12 text-center">
                    <h2 class="section-heading">Services</h2>
                    <h3 class="section-subheading text-muted">Lorem ipsum dolor sit amet consectetur.</h3>
                </div>
            </div>
            <div class="row text-center">
                <div class="col-md-4">
                    <span class="fa-stack fa-4x">
                        <i class="fa fa-circle fa-stack-2x text-primary"></i>
                        <i class="fa fa-shopping-cart fa-stack-1x fa-inverse"></i>
                    </span>
                    <h4 class="service-heading">E-Commerce</h4>
                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Minima maxime quam architecto quo inventore harum ex magni, dicta impedit.</p>
                </div>
                <div class="col-md-4">
                    <span class="fa-stack fa-4x">
                        <i class="fa fa-circle fa-stack-2x text-primary"></i>
                        <i class="fa fa-laptop fa-stack-1x fa-inverse"></i>
                    </span>
                    <h4 class="service-heading">Responsive Design</h4>
                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Minima maxime quam architecto quo inventore harum ex magni, dicta impedit.</p>
                </div>
                <div class="col-md-4">
                    <span class="fa-stack fa-4x">
                        <i class="fa fa-circle fa-stack-2x text-primary"></i>
                        <i class="fa fa-lock fa-stack-1x fa-inverse"></i>
                    </span>
                    <h4 class="service-heading">Web Security</h4>
                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Minima maxime quam architecto quo inventore harum ex magni, dicta impedit.</p>
                </div>
            </div>
        </div>
    </section>
```

ไฟล์ `_portfolio.php`

```html
    <!-- Portfolio Grid Section -->
    <section id="portfolio" class="bg-light-gray">
        <div class="container">
            <div class="row">
                <div class="col-lg-12 text-center">
                    <h2 class="section-heading">Portfolio</h2>
                    <h3 class="section-subheading text-muted">Lorem ipsum dolor sit amet consectetur.</h3>
                </div>
            </div>
            <div class="row">
                <div class="col-md-4 col-sm-6 portfolio-item">
                    <a href="#portfolioModal1" class="portfolio-link" data-toggle="modal">
                        <div class="portfolio-hover">
                            <div class="portfolio-hover-content">
                                <i class="fa fa-plus fa-3x"></i>
                            </div>
                        </div>
                        <img src="<?=$directoryAsset?>/img/portfolio/roundicons.png" class="img-responsive" alt="">
                    </a>
                    <div class="portfolio-caption">
                        <h4>Round Icons</h4>
                        <p class="text-muted">Graphic Design</p>
                    </div>
                </div>
                <div class="col-md-4 col-sm-6 portfolio-item">
                    <a href="#portfolioModal2" class="portfolio-link" data-toggle="modal">
                        <div class="portfolio-hover">
                            <div class="portfolio-hover-content">
                                <i class="fa fa-plus fa-3x"></i>
                            </div>
                        </div>
                        <img src="<?=$directoryAsset?>/img/portfolio/startup-framework.png" class="img-responsive" alt="">
                    </a>
                    <div class="portfolio-caption">
                        <h4>Startup Framework</h4>
                        <p class="text-muted">Website Design</p>
                    </div>
                </div>
                <div class="col-md-4 col-sm-6 portfolio-item">
                    <a href="#portfolioModal3" class="portfolio-link" data-toggle="modal">
                        <div class="portfolio-hover">
                            <div class="portfolio-hover-content">
                                <i class="fa fa-plus fa-3x"></i>
                            </div>
                        </div>
                        <img src="<?=$directoryAsset?>/img/portfolio/treehouse.png" class="img-responsive" alt="">
                    </a>
                    <div class="portfolio-caption">
                        <h4>Treehouse</h4>
                        <p class="text-muted">Website Design</p>
                    </div>
                </div>
                <div class="col-md-4 col-sm-6 portfolio-item">
                    <a href="#portfolioModal4" class="portfolio-link" data-toggle="modal">
                        <div class="portfolio-hover">
                            <div class="portfolio-hover-content">
                                <i class="fa fa-plus fa-3x"></i>
                            </div>
                        </div>
                        <img src="<?=$directoryAsset?>/img/portfolio/golden.png" class="img-responsive" alt="">
                    </a>
                    <div class="portfolio-caption">
                        <h4>Golden</h4>
                        <p class="text-muted">Website Design</p>
                    </div>
                </div>
                <div class="col-md-4 col-sm-6 portfolio-item">
                    <a href="#portfolioModal5" class="portfolio-link" data-toggle="modal">
                        <div class="portfolio-hover">
                            <div class="portfolio-hover-content">
                                <i class="fa fa-plus fa-3x"></i>
                            </div>
                        </div>
                        <img src="<?=$directoryAsset?>/img/portfolio/escape.png" class="img-responsive" alt="">
                    </a>
                    <div class="portfolio-caption">
                        <h4>Escape</h4>
                        <p class="text-muted">Website Design</p>
                    </div>
                </div>
                <div class="col-md-4 col-sm-6 portfolio-item">
                    <a href="#portfolioModal6" class="portfolio-link" data-toggle="modal">
                        <div class="portfolio-hover">
                            <div class="portfolio-hover-content">
                                <i class="fa fa-plus fa-3x"></i>
                            </div>
                        </div>
                        <img src="<?=$directoryAsset?>/img/portfolio/dreams.png" class="img-responsive" alt="">
                    </a>
                    <div class="portfolio-caption">
                        <h4>Dreams</h4>
                        <p class="text-muted">Website Design</p>
                    </div>
                </div>
            </div>
        </div>
    </section>
```

ไฟล์ `_about.php`

```html
    <!-- About Section -->
    <section id="about">
        <div class="container">
            <div class="row">
                <div class="col-lg-12 text-center">
                    <h2 class="section-heading">About</h2>
                    <h3 class="section-subheading text-muted">Lorem ipsum dolor sit amet consectetur.</h3>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-12">
                    <ul class="timeline">
                        <li>
                            <div class="timeline-image">
                                <img class="img-circle img-responsive" src="<?=$directoryAsset?>/img/about/1.jpg" alt="">
                            </div>
                            <div class="timeline-panel">
                                <div class="timeline-heading">
                                    <h4>2009-2011</h4>
                                    <h4 class="subheading">Our Humble Beginnings</h4>
                                </div>
                                <div class="timeline-body">
                                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sunt ut voluptatum eius sapiente, totam reiciendis temporibus qui quibusdam, recusandae sit vero unde, sed, incidunt et ea quo dolore laudantium consectetur!</p>
                                </div>
                            </div>
                        </li>
                        <li class="timeline-inverted">
                            <div class="timeline-image">
                                <img class="img-circle img-responsive" src="<?=$directoryAsset?>/img/about/2.jpg" alt="">
                            </div>
                            <div class="timeline-panel">
                                <div class="timeline-heading">
                                    <h4>March 2011</h4>
                                    <h4 class="subheading">An Agency is Born</h4>
                                </div>
                                <div class="timeline-body">
                                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sunt ut voluptatum eius sapiente, totam reiciendis temporibus qui quibusdam, recusandae sit vero unde, sed, incidunt et ea quo dolore laudantium consectetur!</p>
                                </div>
                            </div>
                        </li>
                        <li>
                            <div class="timeline-image">
                                <img class="img-circle img-responsive" src="<?=$directoryAsset?>/img/about/3.jpg" alt="">
                            </div>
                            <div class="timeline-panel">
                                <div class="timeline-heading">
                                    <h4>December 2012</h4>
                                    <h4 class="subheading">Transition to Full Service</h4>
                                </div>
                                <div class="timeline-body">
                                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sunt ut voluptatum eius sapiente, totam reiciendis temporibus qui quibusdam, recusandae sit vero unde, sed, incidunt et ea quo dolore laudantium consectetur!</p>
                                </div>
                            </div>
                        </li>
                        <li class="timeline-inverted">
                            <div class="timeline-image">
                                <img class="img-circle img-responsive" src="<?=$directoryAsset?>/img/about/4.jpg" alt="">
                            </div>
                            <div class="timeline-panel">
                                <div class="timeline-heading">
                                    <h4>July 2014</h4>
                                    <h4 class="subheading">Phase Two Expansion</h4>
                                </div>
                                <div class="timeline-body">
                                    <p class="text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Sunt ut voluptatum eius sapiente, totam reiciendis temporibus qui quibusdam, recusandae sit vero unde, sed, incidunt et ea quo dolore laudantium consectetur!</p>
                                </div>
                            </div>
                        </li>
                        <li class="timeline-inverted">
                            <div class="timeline-image">
                                <h4>Be Part
                                    <br>Of Our
                                    <br>Story!</h4>
                            </div>
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </section>
```

ไฟล์ `_team.php`

```html
    <!-- Team Section -->
    <section id="team" class="bg-light-gray">
        <div class="container">
            <div class="row">
                <div class="col-lg-12 text-center">
                    <h2 class="section-heading">Our Amazing Team</h2>
                    <h3 class="section-subheading text-muted">Lorem ipsum dolor sit amet consectetur.</h3>
                </div>
            </div>
            <div class="row">
                <div class="col-sm-4">
                    <div class="team-member">
                        <img src="<?=$directoryAsset;?>/img/team/1.jpg" class="img-responsive img-circle" alt="">
                        <h4>Kay Garland</h4>
                        <p class="text-muted">Lead Designer</p>
                        <ul class="list-inline social-buttons">
                            <li><a href="#"><i class="fa fa-twitter"></i></a>
                            </li>
                            <li><a href="#"><i class="fa fa-facebook"></i></a>
                            </li>
                            <li><a href="#"><i class="fa fa-linkedin"></i></a>
                            </li>
                        </ul>
                    </div>
                </div>
                <div class="col-sm-4">
                    <div class="team-member">
                        <img src="<?=$directoryAsset;?>/img/team/2.jpg" class="img-responsive img-circle" alt="">
                        <h4>Larry Parker</h4>
                        <p class="text-muted">Lead Marketer</p>
                        <ul class="list-inline social-buttons">
                            <li><a href="#"><i class="fa fa-twitter"></i></a>
                            </li>
                            <li><a href="#"><i class="fa fa-facebook"></i></a>
                            </li>
                            <li><a href="#"><i class="fa fa-linkedin"></i></a>
                            </li>
                        </ul>
                    </div>
                </div>
                <div class="col-sm-4">
                    <div class="team-member">
                        <img src="<?=$directoryAsset;?>/img/team/3.jpg" class="img-responsive img-circle" alt="">
                        <h4>Diana Pertersen</h4>
                        <p class="text-muted">Lead Developer</p>
                        <ul class="list-inline social-buttons">
                            <li><a href="#"><i class="fa fa-twitter"></i></a>
                            </li>
                            <li><a href="#"><i class="fa fa-facebook"></i></a>
                            </li>
                            <li><a href="#"><i class="fa fa-linkedin"></i></a>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-8 col-lg-offset-2 text-center">
                    <p class="large text-muted">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Aut eaque, laboriosam veritatis, quos non quis ad perspiciatis, totam corporis ea, alias ut unde.</p>
                </div>
            </div>
        </div>
    </section>
```

ไฟล์ `_client.php`

```html
    <!-- Clients Aside -->
    <aside class="clients">
        <div class="container">
            <div class="row">
                <div class="col-md-3 col-sm-6">
                    <a href="#">
                        <img src="<?=$directoryAsset?>/img/logos/envato.jpg" class="img-responsive img-centered" alt="">
                    </a>
                </div>
                <div class="col-md-3 col-sm-6">
                    <a href="#">
                        <img src="<?=$directoryAsset?>/img/logos/designmodo.jpg" class="img-responsive img-centered" alt="">
                    </a>
                </div>
                <div class="col-md-3 col-sm-6">
                    <a href="#">
                        <img src="<?=$directoryAsset?>/img/logos/themeforest.jpg" class="img-responsive img-centered" alt="">
                    </a>
                </div>
                <div class="col-md-3 col-sm-6">
                    <a href="#">
                        <img src="<?=$directoryAsset?>/img/logos/creative-market.jpg" class="img-responsive img-centered" alt="">
                    </a>
                </div>
            </div>
        </div>
    </aside>
```

ไฟล์ `_contact.php`

```html
    <!-- Contact Section -->
    <section id="contact">
        <div class="container">
            <div class="row">
                <div class="col-lg-12 text-center">
                    <h2 class="section-heading">Contact Us</h2>
                    <h3 class="section-subheading text-muted">Lorem ipsum dolor sit amet consectetur.</h3>
                </div>
            </div>
            <div class="row">
                <div class="col-lg-12">
                    <form name="sentMessage" id="contactForm" novalidate>
                        <div class="row">
                            <div class="col-md-6">
                                <div class="form-group">
                                    <input type="text" class="form-control" placeholder="Your Name *" id="name" required data-validation-required-message="Please enter your name.">
                                    <p class="help-block text-danger"></p>
                                </div>
                                <div class="form-group">
                                    <input type="email" class="form-control" placeholder="Your Email *" id="email" required data-validation-required-message="Please enter your email address.">
                                    <p class="help-block text-danger"></p>
                                </div>
                                <div class="form-group">
                                    <input type="tel" class="form-control" placeholder="Your Phone *" id="phone" required data-validation-required-message="Please enter your phone number.">
                                    <p class="help-block text-danger"></p>
                                </div>
                            </div>
                            <div class="col-md-6">
                                <div class="form-group">
                                    <textarea class="form-control" placeholder="Your Message *" id="message" required data-validation-required-message="Please enter a message."></textarea>
                                    <p class="help-block text-danger"></p>
                                </div>
                            </div>
                            <div class="clearfix"></div>
                            <div class="col-lg-12 text-center">
                                <div id="success"></div>
                                <button type="submit" class="btn btn-xl">Send Message</button>
                            </div>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </section>
```

และสร้าง ไฟล์ `about.php` เพื่อเป็นตัวอย่างการเรียกใช้งานอีกแบบ

```php
<?php
use yii\helpers\Html;

/* @var $this yii\web\View */
$this->title = 'About';
$this->params['breadcrumbs'][] = $this->title;
Yii::$app->layout='page_header';

$directoryAsset = Yii::$app->assetManager->getPublishedUrl('@agency/dist');
$this->registerJsFile($directoryAsset.'/js/cbpAnimatedHeader.min.js');
?>

<header style="background-image:url(<?=$directoryAsset.'/img/header-bg2.jpg'?>);">
    <div class="container">
        <div class="intro-text">
           
            <div class="intro-heading">About</div>
             <div class="intro-lead-in">Yii2 Learning</div>
        </div>
    </div>
</header>

<section class="page" id="about">
<div class="container">
<div class="site-about">
<h1><?= Html::encode($this->title) ?></h1>

<p>
This is the About page. You may modify the following file to customize its content:
</p>

<code><?= __FILE__ ?></code>
</div>

</div>
</section>

```

เมื่อสร้างไฟล์ครบเราจะได้โครงสร้างโฟลเดอร์ตามนี้

![](/img/agency-folder.png)


## เปิดใช้งาน

ไปที่ config/main.php

```php

    'components' => [
        'view' => [
             'theme' => [
                 'pathMap' => [
                    '@app/views' => '@agency/views'
                 ],
             ],
        ],
        //....
]
```

ทดลองใช้งาน ก็จะได้ theme ใหม่ดังภาพด้านล่าง สามารถดาวโหลดไฟล์ทั้งหมดได้ [ที่นี่](https://github.com/dimpled/Yii2-Learning-Source)

![](/img/agency-demo.png)


## สรุปการสร้าง Theme

- สร้าง folder themes/agency
- copy โฟลเดอร์จาก layouts จาก views เดิมมาไวัที่ agency
- สร้าง assets เพื่อเรียกใช้งาน css,js
- สร้าง layout แบบอื่นๆ เพื่อใช้งานในรูปแบบที่แตกต่างกัน
- ตั้งค่าที่ view  ให้ทำการเรียกใช้งาน view ที่ theme ของเรา

 



