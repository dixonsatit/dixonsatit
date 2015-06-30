---
layout: post
title : การส่งเมล์ โดยใช้  yii2-swiftmailer  ด้วย smtp-google & smtp-mandrill
excerpt :  การส่งเมล์ โดยใช้  yii2-swiftmailer  ด้วย smtp-google & smtp-mandrill, yii2, yii Framework
---

ปกติเวลาส่งเมล์ เราจะใช้ mail() ของ php ที่มีให้ มันมักจะมีปัญหาเมื่อส่งไปแล้ว จะเข้าที่ junk หรือไม่ก็ spam วันนี้จึงขอแนะนำการส่งเมล์ด้วย smtp จะช่วยแก้ปัญหานี้ได้ และยังมีบริการให้ใช้ฟรี หลายๆ ราย เช่น google,mandrill อีกทั้งเรายังสามารถทดสอบส่งเมล์ได้ ถึงแม้ระบบของเรายังไม่ขึ้น Prodruction ก็ตาม หรือจะสร้าง layout เมล์แบบต่างๆ ก็สามารถทำได้

วันนี้ขอแนะนำการส่งเมล์ด้วย smtp-google และ smtp-mandrill เพราะฟรีและสามารถทำได้ง่ายๆ ซึ่ง Yii2 เตรียมพร้อมไว้ให้เราแล้ว คือ [yii2-swiftmailer](https://github.com/yiisoft/yii2-swiftmailer) มันติดตั้งมาพร้อมแล้ว รอแค่เราตั้งค่าเพื่อเปิดใช้งาน

## เตรียมความพร้อมกันก่อน
- ติดตั้ง yii2-swiftmailer อันนี้สบายเพราะ Yii 2 ติดตั้งมาให้แล้ว
- smtp ที่ต้องการจะใช้ ซึ่งจะไม่เหมือนกัน เช่น smtp ของ google คือ `smpt.gmail.com`
- อีเมล์ account ที่ใช้งานได้จริง ตามตัวอย่างนี้ผมใช้ของ gmail
- layout อีเมล์ เราสามารถสร้างรูปแบบอีเมล์หลายๆ แบบได้ เช่น layout สำหรับตอนสมัครสมาชิก, layout สำหรับตอนส่งข่าว

## เปรียบเทียระหว่าง Gmail และ Mandrill

Gmail คงไม่ต้องอธิบายคงทราบกันดี ส่วน [Mandrill](http://www.mandrill.com/) นั้นมันเป็นระบบ SMTP Mandrill สำหรับส่งเมล์ในระบบเว็บหรือ App ต่างๆ ฟรี 12000 เมล์/เดือน คือมันสร้างมาเพื่อส่งเมล์โดยเฉพาะครับ มีกราฟแสดงให้ดู ตรวจสอบสถิติต่างๆ ส่งไปกี่เมล์ได้รับกี่เมล์ สามารถส่งได้ทีละเยอะๆ แต่ถ้าส่งไม่เกิน 12000 เมล์/เดือน ก็ฟรีครับ ลองไปสมัครใช้ดูได้

> ปล. เท่าที่ผมลองใช้ Mandrill ถ้า server อยู่ในไทย ผมลองแล้วรู้สึกว่า delay นิดนึงครับ แต่ถ้า server อยูต่างประเทศเร็วมากส่งปั๊บเข้าเลย พอดีผมใช้ [DigitalOcean](https://www.digitalocean.com/?refcode=117ef266fe2c) ใครสนใจก็ลองได้ server ราคาถูกมากๆ ถูกกว่า [Aws](http://aws.amazon.com/ec2/)

จริงๆ สองตัวนี้มีคุณสมบัติคือสามารถส่งเมล์ผ่าน smtp ได้เหมือนกัน  แล้วแต่จะเลือกใช้ครับ ^^

เปิดใช้งาน mailer และตั้งค่าตามนี้ ใส่ email และ password ของเรา

## ตั้งค่า SMTP Google

```php
return [
    //....
    'components' => [
        'mailer' => [
           'class' => 'yii\swiftmailer\Mailer',
		        'viewPath' => '@backend/mail',
		        'useFileTransport' => false,
		        'transport' => [
		            'class' => 'Swift_SmtpTransport',
		            'host' => 'smtp.gmail.com',
		            'username' => 'username@gmail.com',
		            'password' => 'password',
		            'port' => '587',
		            'encryption' => 'tls',
		        ],
		    ],
        ],
];
```

## ตั้งค่า SMTP Mandrill

```php
return [
    //....
    'components' => [
        'mailer' => [
           'class' => 'yii\swiftmailer\Mailer',
		        'viewPath' => '@backend/mail',
		        'useFileTransport' => false,
	            'transport' => [
	                'class' => 'Swift_SmtpTransport',
	                'host' => 'smtp.mandrillapp.com',
	                'username' => 'username@gmail.com',
                    'password' => 'password',
	                'port' => '587',
	                'encryption' => 'tls',
	            ],
		],
   ],
];
```

> `viewPath` ถ้าเป็นกรณี app basic ก็ตั้งค่าเป็น `@app/mail`

## สร้าง layout Mail

สร้าง folder ไว้เก็บ view สำหรับเวลาส่งอีเมล์ ชื่อ `/mail/layouts/` ไว้ที่ application ของเรา
จากนั้นสร้าง view ชื่อ `html.php` ไว้ที่ `/mail/layouts/html.php` เมื่อเรียกเป็น alias ก็คือ `@app/mail`

ซึ่งตัว layout นีจะเป็น layout หลักหรือเป็นหน้าโครงสร้าง html ให้เมล์ของเราส่วนใน layout ข้างในเราค่อยไปใส่เนื้อหาอีกที

```html
<?php
use yii\helpers\Html;
?>
<?php $this->beginPage() ?>
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=<?= Yii::$app->charset ?>" />
    <title><?= Html::encode($this->title) ?></title>
    <?php $this->head() ?>
</head>
<body>
<?php $this->beginBody() ?>

<div style="border: solid 1px rgb(236, 236, 236);border-radius: 5px;padding: 20px;">
	<?= $content ?>
</div>
<p style="text-align:right;margin-top:5px;"> โรงพยาบาลขอนแก่น <small ><i>โทร 043-336789</i></small></p>

<?php $this->endBody() ?>
</body>
</html>
<?php $this->endPage() ?>

```
ถ้าหาก preview จะได้แบบนี้

![mailer-main-layout.png](/img/mailer-main-layout.png)

จากนั้นเราจะทำการสร้าง view ที่ใช้เป็น template ในการส่ง ในตัวอย่างนี้ผมสร้างอีเมล์แจ้งเตือนการลงทะเบียนงานประชุมวิชาการโรงพยาบาลขอนแก่น

ไปที่โฟลเดอร์ `/mail/layouts/` ใน application ของคุณ สร้าง view ชื่อ register.php โดยที่ view นี้จะรับค่าตัวแปรชื่อ `$fullname` เพื่อแสดงชื่อของผู้ลงทะเบียน

```html
<h1>งานประชุมวิชาการโรงพยาบาลขอนแก่น ครั้งที่ 7 ประจำปี 2558</h1>
<h3>Theme "คุณธรรม คู่คุณภาพ เพื่อประชาชน"</h3>
<p>เนื่องในโอกาสครบรอบ 20 ปี ศูนย์แพทยศาสตรศึกษาชั้นคลินิก โรงพยาบาลขอนแก่น</p>
<p>24 - 26 มิถุนายน 2558 ณ โรงพยาบาลขอนแก่น</p>

<p>เรียนคุณ <?php echo $fullname; ?></p>
<p>ตอนนี้คุณได้ทำการลงทะเบียนเข้าร่วมงานประชุมวิชาการ รพ.ขอนแก่น 2558 เรียบร้อยแล้ว</p>
<p>คุณสามารถดูรายละเอียดกำหนดการต่างๆ  ตามไฟล์แนบได้ หรือดูรายละเอียด <a href="http://seminar.kkh.go.th">ได้ที่นี่</a> </p>

```

เมือทำการส่งจะได้ layout แบบนี้

![](/img/mailer-register-layout.png)


## ส่งอีเมล์

หลังจากที่เราตั้งค่าที่ component แล้วเราจะสามารถเรียกใช้งานด้วย `Yii::$app->mailer` ได้ ดูตัวอย่าง

```css

Yii::$app->mailer->compose('@app/mail/layouts/register',[
    'fullname'=>'สาธิต สีถาพล'
])
->setFrom(['seminar@kkh.go.th'=>'Khon Kaen Hospital'])
->setTo('emailTo@gmail.com')
->setSubject('ยินดีต้อนรับสู่งานประชุมวิชาการโรงพยาบาลขอนแก่น 2558')
->send();

```

- `compose` คือการเรียกไปที่ view ที่เราสร้างเป็น template ไว้หากต้องการใช้หลายๆ แบบ ก็สร้างไว้หลายๆ อันแล้วก็เรียกใช้ที่ตรงนี้ หาก view มีการรับค่าตัวแปร ก็ให้ส่งค่าไปด้วย ตามตัวอย่าง register.php รับค่าตัวแปร $fullname
- `setFrom` เป็นการระบุว่าส่งมาจากที่ใด และใส่เป็นชื่อได้ด้วย ![](/img/mailer-setfrom.png)
- `setTo` คืออีเมล์ที่เราต้องการส่งไปหาใคร
- `setSubject` ชื่อเรื่องอีเมล์ที่เราจะส่ง


จากนั้นทดลองเรียกใช้งาน ก็จะได้รับอีเมล์แบบนี้

![](/img/mailer-full.png)


หากต้องการแนบไฟล์ให้เพิ่ม `attach` เข้าไปดังนี้จะใส่กี่ไฟล์ก็ได้

```css
Yii::$app->mailer->compose('@app/mail/layouts/register',[
    'fullname'=>'สาธิต สีถาพล'
])
->setFrom(['seminar@kkh.go.th'=>'Khon Kaen Hospital'])
->setTo('emailTo@gmail.com')
->setSubject('ยินดีต้อนรับสู่งานประชุมวิชาการโรงพยาบาลขอนแก่น 2558')
->attach(Yii::getAlias('@webroot').'/attach/'.'brochure.pdf')
->attach(Yii::getAlias('@webroot').'/attach/'.'Poster.pdf')
->send();
```

> ไม่ควรแนบไฟล์เยอะเพราะจะทำให้ตอนส่งช้าไปด้วย ถ้าไฟล์เยอะจริงๆ แนะนำทำเป็น zip และแนบ link ดาวน์โหลดไปจะดีกว่า

เมื่อแนบไฟล์ เวลาส่งอีเมล์ผู้รับก็จะได้อีเมล์แบบนี้

![](/img/mailer-attach.png)


ในการใช้งานจริงแนะนำให้สร้างเป็น function ไว้ใน controller ก็ได้ แล้วค่อยรับค่า parameter ต่างๆ เข้ามา

```php
<?php
	//...
    public function sendMail($email,$fullname){
         Yii::$app->mailer
         ->compose('@app/mail/layouts/register',[
            'fullname'=>$fullname
          ])
         ->setFrom(['seminar@kkh.go.th'=>'Khon Kaen Hospital'])
         ->setTo($email)
         ->setSubject('ยินดีต้อนรับสู่งานประชุมวิชาการโรงพยาบาลขอนแก่น 2558')
         ->attach(Yii::getAlias('@webroot').'/attach/'.'brochure.pdf')
         ->attach(Yii::getAlias('@webroot').'/attach/'.'Poster.pdf')
         ->send();
    }
```


ขอให้สนุกกับการส่งเมล์ครับผม.. ^^ กะไว้ว่าเรื่องนี้จะเขียนประมาณซัก 20 นาทีเอาเข้าจริงๆ ปาเข้าไป 2 ชั่วโมงแระ ไม่ไหวนอนก่อนครับ
