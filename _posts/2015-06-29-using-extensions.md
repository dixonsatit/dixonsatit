---
layout : post
title : การติดตั้ง และการใช้งาน composer กับ Yii 2
excerpt : การติดตั้ง และการใช้งาน composer กับ Yii 2,yii2,yii Framework
---
![](/img/composer.jpg)

ในการพัฒนาระบบโดยใช้ php นั้น แต่ก่อนจะไม่มีระบบจัดการแพ็จเกจมาให้ เราจะต้องเข้าไปดาวน์โหลดที่เว็บไซต์ของ library เองจากนั้นทำการแตกซิบไฟล์แล้ว include เข้ามาใช้งาน วันดีคืนดี หาก library มีการอัพเดท ก็ต้องตามไปโหลด library มาแล้วนำไฟล์มาทับของเก่า ซึ่งก็เป็นเรื่องที่น่าเบื่อมากๆ  ยิ่งถ้าเราใช้ library เยอะๆ นี่ตามไปโหลดแทบไม่ไหว

php จึงมองเห็นความลำบากนี้ (จริงๆ ภาษาอื่นๆ เค้ามีกันไปนานแล้วววว เอาเถอะยังไงมาช้าก็ดีกว่าไม่มา  )
จึงมีระบบจัดการแพ็ตเกจของ php ขึ้นชื่อว่า `composer` ซึ่งเป็นมาตรฐานกลาง ทำให้เราสามารถจัดการแพ็คเกจต่างๆ ได้ที่เดียว ไม่ต้องตามไปดาวน์โหลดจากเว็บไซต์ของ library ต่างๆ

## การติดตั้ง Composer

สามารถทำได้ง่ายๆ ติดตั้งได้ทั้ง Windows,Linux,Mac ตัว composer เป็นแค่ไฟล์ php เล็กๆ ไฟล์นึงเท่านั้น เอาไว้จัดการ เพ็คเกจต่างๆ

### Windows
สามารถโหลดตัว exe มาติดตั้งได้เลย [ดาวน์โหลดตัวติดตั้ง](https://getcomposer.org/doc/00-intro.md#installation-windows) จากนั้นจะได้ `Composer-Setup.exe` ทำการคลิกและติดตั้งเองได้เลย

> หากมี error เกี่ยวกับ `http ` ให้ทำการเปิด extension `php_openssl.dll` ใน php.ini และหากติดตั้งแล้วยังไม่ได้ ให้ลองติดตั้งซ้ำใหม่อีกรอบ ส่วนใหญ่จะเจอแค่ xp

### Linux & Mac

ติดตั้งผ่านคำ curl จากนั้นรันคำสั่ง mv เพื่อเปลี่ยนชื่อไฟล์ เป็น composer และย้ายไปไว้ที่ bin

```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```

## เรียกใช้งาน
เปิด command หรือ terminal  ขึ้นมาแล้ว รันคำสั่ง `composer` จะพบกับหน้าจอ composer แบบนี้ แสดงว่าพร้อมใช้งานแล้ว

```
______
/ ____/___  ____ ___  ____  ____  ________  _____
/ /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                  /_/
Composer version 1.0-dev (4f80e7ff68466cab970c22b425725b8058c32970) 2015-06-09 00:03:48

Usage:
command [options] [arguments]

Options:
--help (-h)           Display this help message
--quiet (-q)          Do not output any message
--verbose (-v|vv|vvv) Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
--version (-V)        Display this application version
--ansi                Force ANSI output
--no-ansi             Disable ANSI output
--no-interaction (-n) Do not ask any interactive question
--profile             Display timing and memory usage information
--working-dir (-d)    If specified, use the given directory as working directory.

Available commands:
about            Short information about Composer
archive          Create an archive of this composer package
browse           Opens the package's repository URL or homepage in your browser.
clear-cache      Clears composer's internal package cache.
clearcache       Clears composer's internal package cache.
config           Set config options
create-project   Create new project from a package into given directory.
depends          Shows which packages depend on the given package
diagnose         Diagnoses the system to identify common errors.
dump-autoload    Dumps the autoloader
dumpautoload     Dumps the autoloader
global           Allows running commands in the global composer dir ($COMPOSER_HOME).
help             Displays help for a command
home             Opens the package's repository URL or homepage in your browser.
info             Show information about packages
init             Creates a basic composer.json file in current directory.
install          Installs the project dependencies from the composer.lock file if present, or falls back on the composer.json.
licenses         Show information about licenses of dependencies
list             Lists commands
remove           Removes a package from the require or require-dev
require          Adds required packages to your composer.json and installs them
run-script       Run the scripts defined in composer.json.
search           Search for packages
self-update      Updates composer.phar to the latest version.
selfupdate       Updates composer.phar to the latest version.
show             Show information about packages
status           Show a list of locally modified packages
update           Updates your dependencies to the latest version according to composer.json, and updates the composer.lock file.
validate         Validates a composer.json
```

> เราสามารถสั่งให้มันอัพเดทตัวเองได้  โดยใช้คำสั่ง `composer self-update` และคำสังอื่นๆ สามารถดูได้ที่เวลาเราพิมพ์คำสั่ง composer เข้าไปเฉยๆ แล้วดูตรง `Available commands` เหมือนด้านบน

ใน project Yii ที่เราได้สร้างขึ้น จะมีไฟล์ชื่อว่า composer.json เป็นตัวจัดการ packages ทั้งหมดและสามารถบอกได้ว่า Applicaion  ของเราได้ติดตั้ง extension อะไรไปบ้าง หากมีการติดตั้ง extension เพิ่มเติมมันจะทำการแก้ไขไฟล์นี้ให้อัตโนมัติ ยกเว้นว่าเราเพิ่ม extension เองในไฟล์นี้

### การติดตั้ง extension
การติดตั้ง extension ผ่าน composer นั้นทำได้ 2 วิธีคือ

**ติดตั้งผ่านคำสั่ง `composer require`**

วิธีนี้สามารถติดตั้งได้ทีละ 1 extension ตัวอย่างการติดตั้ง

```
composer require --prefer-dist yiisoft/yii2-imagine
```
หลังจากติดตั้งเสร็จ composer จะแก้ไขไฟล์ composer.json ให้เราเอง โดยเพิ่ม ชื่อ extension ที่ tag require ที่เราได้ติดตั้งเข้าไป

**ติดตั้งโดยเข้าไปแก้ไขไฟล์ composer.json**

 สามารถเข้าไปแก้ไขไฟล์โดยตรงในส่วนของ `require` สามารถติดตั้งได้ทีละหลายๆ ตัว ตัวอย่างนี้เพิ่มเข้าไป 2 extension คือ `yii2-chartjs-widget`,`yii2-imagine`

```
"require": {
      "php": ">=5.4.0",
      "yiisoft/yii2": ">=2.0.4",
      "yiisoft/yii2-bootstrap": "*",
      "yiisoft/yii2-swiftmailer": "*",
      "2amigos/yii2-chartjs-widget": "~2.0",
      "yiisoft/yii2-imagine": "*"
  },
```
หลังจากเพิ่ม extension ที่ต้องการจะติดตั้งเสร็จ ให้ทำการรันคำสั่ง `composer update` composer จะทำการตรวจสอบและ install extension ให้เรา

```
composer update
```

หลังจากนั้น extension ที่เราติดตั้งจะถูกโหลดไปเก็บไว้ที่ `vendor` เพื่อรอให้เราเรียกใช้งานผ่าน namespace ได้
