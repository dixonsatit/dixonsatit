---
layout : post
title : RBAC คืออะไร? พร้อมสอนวิธีใช้ RBAC DB
categorys : authorization
tags : rbac
---

ในการพัฒนาระบบ เราจำต้องมีการควบคุมการเข้าใช้งานในส่วนต่างๆ ถ้าหากระบบของเรามีเพียงแค่ login,logout นั้นคงไม่พอครับ เพราะในการใช้งานปกติ ผู้ใช้งานแต่ละคนอาจจะมีสิทธิ์ในการเข้าใช้งานที่แตกต่างกัน อย่างน้อยก็เพื่อเป็นการปกป้องข้อมูลหรือการใช้งานในแต่ละส่วนให้ถูกต้องเหมาะสม  ถ้าหากระบบที่เราได้พัฒนาขึ้นแล้ว แต่ยังไม่มีการจัดการ RBAC ผมคิดว่าคุณควรต้องคิดใหม่..เพราะตรงนี้  Yii2 ได้เตรียมไว้ให้เราครบ พร้อมเสริฟ รอเพียงแค่กดน้ำร้อน ปรุงรสตามใจชอบ เท่านั้นเอง ดี้ดี ^ ^

เนื้อหาในเรื่องนี้ผมจะอธิบายตั้งแต่การออกแบบ RBAC ไปจนถึงตัวอย่างการใช้งานกับโปรเจ็คง่ายๆ เพื่อให้เราสามารถออกแบบและใช้งาน RBAC กับ Application ที่เราพัฒนาได้อย่างเหมาะสม

#เนื้อหามีอะไรบ้าง#

- [RBAC คืออะไร?](#rbac-คืออะไร)
- ลองออกแบบ RBAC
- สร้างโปรเจ็คเพื่อใช้งาน RBAC
- การติดตั้งและสร้าง RBAC
- รู้จักกับ Access Control Filter
- การกำหนดค่าและเรียกใช้งาน RBAC ใน Controller
- สร้าง Rule เพื่อตรวจสอบผู้ใช้งาน
- ปรับปรุงระบบ Registration เพื่อใส่ค่า default Role
- การสร้างระบบจัดการผู้ใช้งานเพื่อมอบสิทธิ์ให้กับผู้ใช้งาน

#RBAC คืออะไร?

RBAC คือ การจัดการสิทธิ์การเข้าใช้งานหรือที่เรียกว่า Role Based Access Control (RBAC) ซึ่งเป็นการควบคุมการเข้าใช้งานในส่วนต่างๆ ของ application ที่เราได้สร้างขึ้น โดยเมื่อมีการเรียกใช้งานในแต่ละส่วน RBAC จะคอยเช็คผู้ใช้งานแต่ละคนว่า มีสิทธิ์ที่จะเข้าใช้งานในส่วนนี้หรือไม่ หากมีก็จะยอมให้เข้าใช้งาน แต่ถ้าหากไม่มีก็จะทำการแสดง error forbidden ออกไป เพื่อให้ผู้ใช้งานทราบว่าไม่สามารถเข้าใช้งานในส่วนนี้ได้




โดยปกติแล้ว ผู้ใช้งานแต่ละคนจะมีสิทธ์ในการเข้าใช้งานแตกต่างกัน เพราะฉะนั้นเราจึงจำเป็นต้องแยก role ออกเป็นกลุ่มๆ โดยให้แต่ละกลุ่มมีสิทธิ์ในการเข้าใช้งานที่ไม่เหมือนกัน เมื่อผู้ใช้งานได้รับ role ใหนก็จะสามารถเข้าใช้งานได้ตามสิทธิ์ที่ role นั้นมี ทำให้เราสามารถควบคุมการเข้าใช้งานในส่วนต่างๆ ได้อย่างเหมาะสมและยืดยุ่น

![](/img/rbac-flow.jpg)

ส่วนประกอบหลักๆ ของ rbac ใน yii ที่เราสามารถสร้างได้จะประกอบไปด้วย 3 ส่วนคือ

- Permission ก็เปรียบเสมือนใบอนุญาติที่บอกว่าเราสามารถทำอะไรได้บ้าง
- Role เป็นกลุ่มของ Permission หลายๆ ตัว ที่เกี่ยวข้องกัน รวมตัวกัน เพื่อให้สามารถเรียกใช้งานได้สะดวก ไม่ต้องเรียกใช้งานที่ Permission ทีละอัน
- Rule จะคล้ายๆ กับ Role แต่เป็น Role ที่เราสร้างขึ้นเป็นไฟล์ class เพื่อใช้ในการตรวจสอบค่าบางอย่าง

ถ้าดูจากภาพด้านบนจะเห็นว่ามี Role ทั้งหมด 5 ตัว ซึ่งแต่ละตัวมีความสามารถที่แตกต่างกัน ภายใน Role แต่ละตัวอาจมี Role หรือ Permission อยู่ภายใต้สังกัดได้ ส่วน Permission จะไม่อยู่ภายใต้ Role ก็ได้ แต่ในความเป็นจริง เมื่อใช้งานมันอาจจะไม่สะดวก เพราะหากมี Permission ทั้งหมด 10 ตัวแล้วเราต้องการใช้ User ใช้งานได้เราต้อง add Permission ทั้งหมด 10 ตัวให้ user คนนั้น แต่ถ้าหากเราย้ายให้มันสังกัดที่ Role แล้วเวลาให้สิทธิ์เราก็ให้ Role แค่ตัวเดียว ผู้ใช้งานก็จะได้สิทธิ์การเข้าใช้งานทั้ง 10 ตัว ซึ่งจะสะดวกและยืดยุ่นกว่ามากๆ

เพื่อให้สามารถเข้าใจได้ง่ายๆ ผมจะยกตัวอย่าง โดยใช้ Role ที่กำหนดขึ้นมาใหม่ 3 ตัว คือ

- Blog การใช้งาน blog ทั้งหมดเช่น create,update,delete
- Report การดูรายงานต่างๆ
- Export การ export ข้อมูลในรูปแบบต่างๆ

**ตัวอย่างที่ 1**

![](/img/rbac-03-1.jpg)

เมื่อเราดูตามภาพแล้วจะเห็นว่า มีการมอบ role `Blog` ให้กับ User เมื่อ User ได้รับก็จะสามารถเข้าใช้งานในส่วนของ blog ได้แต่ในส่วนของ role `Report` และ `Export` จะไม่สามารถเข้าใช้งานได้เพราะ user ไม่ได้รับ role นั้น

**ตัวอย่างที่ 2**

![](/img/rbac-03-2.jpg)

User ได้รับมอบ Role `Report` และ `Export` เพราะฉะนั้นก็จะเข้าใช้งาน report,export ได้ ยกเว้น blog เพราไม่ได้รับ role `Blog` มา

**ตัวอย่างที่ 3**

![](/img/rbac-03-3.jpg)

สังเกตว่า User จะได้รับมอบในส่วนของ Role `Blog` เพียงอย่างเดียว แต่ User สามารถเข้าใช้งานทั้ง `Blog` และ `report` ได้ เพราะว่า report ในอยู่ภายใต้ Role `Blog` เพราะฉะนั้นใครก็ตามที่ได้รับ role `Blog` ก็จะสามารถใช้งาน reprot ได้ด้วย

**ตัวอย่างที่ 4**

![](/img/rbac-03-4.jpg)

User ได้รับมอบ role `Report` ก็จะสามารถเข้าใช้งาน report ได้เพียงอย่างเดียวแม้ว่า role `Report` จะอยู่ภายใต้ `Blog` ก็ตาม แต่ถ้าหามี role อะไรก็ตามอยู่ภายใต้ role `Report` มันก็จะสามารถเข้าใช้งานได้ตามไปด้วย

**ตัวอย่างที่ 5**

![](/img/rbac-03-5.jpg)

ในตัวอย่างนี้เราแก้ปัญหาในเรื่องที่ต้องมอบ role ให้กับผู้ใช้งาน โดยปกติถ้าหากแต่ละ role ไม่ได้ขึ้นต่อกันแล้ว เราจะต้องทำการ assignment role ทั้ง 3 ตัวให้กับ user แต่ละคน คนละ 3 role มีกี่คนก็คูน 3 เข้าไปซึ่งมันจะเยอะมาก

```sql
+-----------+---------+
| item_name | user_id |
+-----------+---------+
| Blog      | 1       |
| Report    | 1       |
| Export    | 1       |
| Blog      | 2       |
| Report    | 2       |
| Export    | 2       |
| Blog      | 3       |
| Report    | 3       |
| Export    | 3       |
+-----------+---------+
```

แต่ถ้าเราปรับใหม่โดยให้ role ที่เกี่ยวข้องกันจัดเป็นลำดับชั้นไว้ตามภาพด้านบน เช่น ตัวอย่างจะเรียงลำดับตามนี้ Blog->Report->Export โดยให้ `Blog` เป็น Role สูงสุด ถ้าหากอยากให้ผู้ใช้งานสามารถเข้าใช้งานได้ทั้งหมด 3 role ก็แค่มอบ role Blog ให้กับผู้ใช้งานหรือ User นั้นๆ เมื่อดูข้อมูล

```sql
+-----------+---------+
| item_name | user_id |
+-----------+---------+
| Blog      | 1       |
| Blog      | 2       |
| Blog      | 3       |
+-----------+---------+
```

ผู้ใช้งานหรือ User ก็จะสามารถเข้าใช้งานได้ทั้ง 3 ส่วนคือ Blog,Report,Export

> ตารางผมแค่ทำให้มองเห็นภาพว่าเวลาที่มันเก็บข้อมูลมันเก็บยังไง


**ตัวอย่างที่ 6**

![](/img/rbac-03-6.jpg)

ตัวอย่างนี้ได้เพิ่ม role `Admin` ขึ้นมาเพื่อเป็นตัวควบคุมสูงสุด เพื่อที่จะทำให้เราสามารถมอบ role ต่างๆ ได้อิสระมากขึ้น สังเกตุว่า `Admin` จะมี role ที่เอาไว้จัดการข้อมูลผู้ใช้งานคือ role `ManageUser` เพื่อให้ `Admin` สามารถจัดการข้อมูลผู้ใช้ได้ จึงให้ role `ManageUser` ไปอยู่ภายใต้ role `Admin`

**ตัวอย่างที่ 7**

![](/img/rbac-03-7.jpg)

ถ้าเรามอบ role `Blog` ให้กับ user แล้ว user ก็ใช้งานได้แค่ role `Blog,Report,Export`

**ตัวอย่างที่ 8**

![](/img/rbac-03-8.jpg)

ถ้าหาก user ได้รับมอบ role `ManageUser` แล้วก็จะสามารถใช้งานได้แค่ส่วนจัดการข้อมูลผู้ใช้งานเท่านั้น

**ตัวอย่างที่ 9**

![](/img/rbac-03-9.jpg)

ถ้าหากต่อไปมีการเพิ่ม role ใหม่เข้ามาชื่อ `Export-csv` ใครก็ตามที่อยู่บน role Export-csv  ก็จะสามารถใช้งานได้ทันทีโดยไม่ต้องแก้ไขข้อมูลการ assignment ใหม่เลย อย่างเช่น ถ้าได้รับ role `Blog` ก็จะสามารถใช้งาน Blog->Report->Export->Export-csv ได้เลย เพราะ Export-csv อยู่ภายใต้ Blog อยู่แล้ว

**ตัวอย่างที่ 10**

![](/img/rbac-03-10.jpg)

ถ้า User ได้รับ role `Blog` ก็จะสามารถใช้งาน Blog->Report->Export->Export-csv ได้เลย เพราะ Export-csv อยู่ภายใต้ Blog อยู่แล้ว

ตัวอย่างทั้งหมดคิดว่าคงพอทำให้เราเข้าใจและสามารถออกแบบ rbac ที่เหมาะสมกับงานของตัวเองได้ ต่อไปเราจะทำการลงมือสร้าง rbac จริงๆ เพื่อใช้งานกับ application ของเรา

# ลงมือสร้าง RBAC

หลักๆ ในการสร้าง RBAC นั้น เราจะต้องทำการวิเคราะห์ว่า application ของเรา มีการทำงานหลักๆ อะไรบ้าง ในแต่ละส่วนต้องทำอะไรบ้าง เพื่อให้สามารถสร้าง rbac ได้อย่างง่ายๆ ผมจะยกตัวอย่างการสร้างระบบ Blog ตามตัวอย่าง [doc-2.0](http://www.yiiframework.com/doc-2.0/guide-security-authorization.html) ได้เขียนอธิบายไว้ เพราะเข้าใจง่าย

การออกแบบจะใช้ตัวอย่างการสร้าง Blog แบบง่ายๆ ไม่ซับซ้อนเพื่อให้เราสามารถทำความเข้าใจได้อย่างรวดเร็ว ก่อนอื่นเราจะต้องแบ่งกลุ่มผู้ใช้งานว่ามีอะไรบ้างหลักๆ แยกออกมา หลังจากนั้นให้เราเขียนความสามารถของระบบว่าทำอะไรได้บ้างหลักๆ ออกเป็นข้อๆ ดังนี้

## กลุ่มผู้ใช้งาน

ระบบแบ่งผู้ใช้งานเป็น 4 กลุ่มใหญ่ๆ คือ

- Author ผู้ใช้งานที่เขียนบทความ
- Editor ผู้ตรวจสอบ
- ManageUser จัดการข้อมูลผู้ใช้งาน
- Admin ผู้ดูแลระบบ

ซึ่งในส่วนนี้เราจะเรียกมันว่า Role

## สิทธิ์การเข้าใช้งาน

- สามารถ create บทความได้
- สามารถ update บทความได้
- สามารถ update บทความคนอื่นได้
- สามารถ delete บทความคนอื่นได้
- สามารถ จัดการข้อมูลผู้ใช้งาน

เมื่อเราทำการวิเคราะห์และออกแบบระบบหลักๆ ได้แล้วเราจะต้องทำการให้สิทธิ์ แก่ Role หรือกลุ่มผู้ใช้งานที่เราได้ทำการออกแบบไว้ว่า กลุ่มใหนสามารถทำอะไรได้บ้างดังนี้

![](/img/rbac-role.jpg)

- Author
 - create บทความ
 - update บทความ

- Editor
 - update บทความของ Author ได้
 - delete บทความของ Author ได้

- ManageUser
  - สามารถจัดการข้อมูลผู้ใช้งานได้

- Admin
 - เป็นผู้ดูแลระบบ

เมื่อเราออกแบบและเข้าใจโครงสร้างระบบโดยรวมแล้ว จะทำให้เราสามารถสร้าง RBAC ได้ง่าย และไม่สับสน

## เตรียม project ให้พร้อม

ก่อนอื่นเราจะต้องเตรียม project ให้พร้อม โดยตัวอย่างนี้เราจะใช้ Advanced Project Template เพราะมีระบบ login มาให้อยู่แล้ว ให้ทำการสร้าง Advanced Project Template ดูวิธีการสร้าง[ได้ที่นี่ ](https://github.com/yiisoft/yii2-app-advanced/blob/master/docs/guide/README.md) หลังจากติดตั้งเสร็จแล้วให้ทำตามขั้นตอนต่อไปนี้

- php yii/init
- สร้างฐานข้อมูลไว้ ชื่อ yii2-workshop-rbac-db
- ทำการ config ฐานข้อมูลที่ common/config/main-local.php ใส่ชื่อฐานข้อมูลและ username & password เปลี่ยน localhost เป็น 127.0.0.1 ให้ถูกต้อง
- ทำการนำเข้าฐานข้อมูล user โดยใช้คำสั่ง `php yii migrate`
- ลงทะเบียนผู้ใช้งาน โดยกำหนด user,password,email เพื่อใช้สำหรับเข้าทดสอบการใช้งาน rbac ที่เราจะสร้างขึ้น มีข้อมูลดังนี้

   - user-a, user-a@gmail.com, 123456
   - user-b, user-b@gmail.com, 123456
   - user-c, user-c@gmail.com, 123456
   - user-d, user-d@gmail.com, 123456

ไปที่ frontend และทำการสมัครสมาชิกที่หน้า `site/signup` ด้วย username,email, password ที่กำหนดให้ในด้านบน

![](/img/rbac-db-register.png)

 หลังจากลงทะเบียนเสร็จเรียบร้อยให้ทำการทดสอบ login ว่าสามารถใช้งานได้จริงหรือไม่ เมื่อสมัครสมาชิกเสร็จ ให้เราทำการตรวจสอบข้อมูลที่ตาราง user จะพบรายการดังนี้

```sql
mysql> select username,email from user;
+----------+------------------+
| username | email            |
+----------+------------------+
| user-a   | user-a@gmail.com |
| user-b   | user-b@gmail.com |
| user-c   | user-c@gmail.com |
| user-d   | user-d@gmail.com |
+----------+------------------+
3 rows in set (0.00 sec)
```



## เปิดการใช้งาน RBAC

Yii 2 มี RBAC ให้เลือกใช้งาน 2 แบบ คือ `yii\rbac\PhpManager` and `yii\rbac\DbManager` ทั้ง 2 ตัวนี้การทำงานหลักๆ ไม่ต่างกัน ต่างกันที่การเก็บข้อมูลเท่านั้นแล้วแต่จะเลือกใช้งาน ตามตัวอย่างวันนี้จะเลือกใช้งาน `yii\rbac\DbManager` เพราะสามารถจัดการได้ง่าย สามารถดูข้อมูลได้ที่ database ของเรา


### เปิดใช้งาน yii\rbac\DbManager

```php
return [
    // ...
    'components' => [
        'authManager' => [
            'class' => 'yii\rbac\DbManager',
        ],
        // ...
    ],
];
```

เนื่องจาก  `DbManager` เก็บข้อมูลที่ db ดังนั้นเราจะต้องนำเข้าโครงสร้างตารางที่ yii เตรียมไว้ให้แล้วโดยใช้ migration ให้เปิด command line แล้วเข้าไปที่  root โปรเจ็คของเราจากนั้นทำการรันคำสั่ง

```
php yii migrate --migrationPath=@yii/rbac/migrations
```

ก็จะพบข้อความ

```
Yii Migration Tool (based on Yii v2.0.6)

Total 1 new migration to be applied:
m140506_102106_rbac_init

Apply the above migration? (yes|no) [no]:yes
*** applying m140506_102106_rbac_init
  > create table {{%auth_rule}} ... done (time: 0.050s)
  > create table {{%auth_item}} ... done (time: 0.019s)
  > create index idx-auth_item-type on {{%auth_item}} (type) ... done (time: 0.025s)
  > create table {{%auth_item_child}} ... done (time: 0.027s)
  > create table {{%auth_assignment}} ... done (time: 0.024s)
*** applied m140506_102106_rbac_init (time: 0.174s)


Migrated up successfully.
```

และทำการเช็คดูใน​ฐานข้อมูลของเราจะพบว่ามีตารางใหม่ขึ้นมา 4 ตาราง ดังนี้

```sql
mysql> show tables;
+---------------------------------+
| Tables_in_yii2-workshop-rbac-db |
+---------------------------------+
| auth_assignment                 |
| auth_item                       |
| auth_item_child                 |
| auth_rule                       |
| migration                       |
| user                            |
+---------------------------------+
6 rows in set (0.01 sec)
```

ตารางของ RBAC จะมี prefix นำหน้าว่า `auth_` เสมอ

> ถ้าหากพบ error อาจต้องตรวจสอบคอนฟิกในส่วนฐานข้อมูล ว่าถูกต้องหรือไม่ หากยังไม่ได้ให้ลองเปลี่ยน localhost เป็น 127.0.0.1 ดู

เมื่อ migration เสร็จเรียบร้อย ให้เราตรวจสอบฐานข้อมูลจะพบว่ามีตารางใหม่ขึ้นมา 4 ตารางคือ

- *itemTable* คือข้อมูลรายการ role และ permission ทั้งหมดซึ่งจะเก็บไว้ที่ตาราง `auth_item`
- *itemChildTable* เป็นข้อมูลที่บอกว่า role ตัวใหนอยู่ภายใต้ role อะไรบ้างซึ่งเก็บไว้ที่ตาราง  `auth_item_child`
- *assignmentTable* เป็นตารางที่ระบุว่า user แต่ละคนมี role เป็นอะไรหรือมีสิทธิ์ทำอะไรบ้าง ตรงนี้จะเป็นตัวระบุ เก็บไว้ที่ตาราง `auth_assignment`
- *ruleTable* : เป็นตารางที่เก็บข้อมูลให้กับ Rule เก็บไว้ที่ตาราง `auth_rule`


เมื่อตั้งค่าเปิดการใช้งาน authManager และเตรียมฐานข้อมูลเสร็จเรียบร้อย เราก็จะสามารถเรียกใช้งาน `authManager` ผ่าน Yii::$app ได้ ดังนี้

```php
 $auth = Yii::$app->authManager;
```
สามารถดูรายละเอียดเพิ่มเติมว่า DbManager ทำอะไรได้บ้าง [ได้ที่นี่](http://www.yiiframework.com/doc-2.0/yii-rbac-dbmanager.html)

## สร้าง RBAC

หลังจากที่เราออกแบบและเปิดการใช้งาน authManager ไว้แล้ว ขั้นตอนต่อไปคือเราจะทำการสร้าง RbacController ซึ่งเป็น Controller แบบ console คือต้องใช้ผ่าน command line เท่านั้น ที่ต้องทำผ่าน command line เพราะปกติการสร้าง rbac เราก็จะทำแค่ครั้งเดียว ยกเว้นว่ามีการแก้ไข role ต่างๆ

เราจะทำการสร้างไฟล์ RbacController ไว้ที่ console/controllers/RbacController.php แล้วสร้างโค้ดตามนี้

```php
<?php
namespace console\controllers;

use Yii;
use yii\helpers\Console;

class RbacController extends \yii\console\Controller {

  public function actionInit(){
     Console::output('Yii 2 Learning.');
  }

}
?>
```

เราสร้าง `RbacController` ขึ้นมาเพื่อใช้ในการสร้าง Rbac หากสังเกตจะพบว่า controller extends ด้วย yii\console\Controller  ซึ่งจะไม่ใช่ controller ปกติที่เราใช้งานกัน ตอนนี้ RbacController มี 1 action คือ Init โดยให้ echo ข้อความออกมาเป็น "Yii 2 Learning"

ทดลองเรียกใช้งาน โดยเปิด command แล้ว cd เข้าไปที่โปรเจคของเรา จากนั้นรันคำสั่ง

```
php yii
```

จะพบกับข้อความประมาณนี้ ให้สังเกตุด้านล่างจะมี rbac/init ที่เราได้สร้างไว้ขึ้นมาแล้ว

```
This is Yii version 2.0.6.

The following commands are available:

- asset                        Allows you to combine and compress your
                               JavaScript and CSS files.
    asset/compress (default)   Combines and compresses the asset files according
                               to the given configuration.
    asset/template             Creates template of configuration file for
                               [[actionCompress]].
// ......

- rbac
    rbac/init


To see the help of each command, enter:

  yii help <command-name>
```

 เราสามารถเรียกใช้งานได้เลย

 ```
 php yii rbac/init
 ```

 จะพบข้อความดังนี้

```
Yii 2 Learning
```

หลังจากทดแล้วแล้วว่าใช้งานได้ ถึงเวลาที่เราจะสร้าง rbac จริงๆ ซึ่งเราได้ออกแบบไว้แล้ว ดูตามภาพ

![](/img/rbac.jpg)

ตามที่ออกแบบไว้เราจะแบ่ง role เป็น 4 ประเภท คือ

- Author สำหรับผู้ที่เขียนบทความ
- Editor สำหรับผู้ที่ตรวจสอบบทความ
- managerUser สำหรับจัดการข้อมูลผู้ใช้งาน
- Admin สำหรับผู้ดูแลระบบ

ซึ่งหากดูตามภาพแล้ว admin จะอยู่สูงสุดและให้ role ต่างๆ ไปอยู่ภายใต้ admin เพื่อทีจะให้ admin สามารถทำทุกอย่างได้ ตามที่ role ที่อยู่ภายใต้มีสิทธิ์ทำ

หลังจากนั้นเราจะทำการ assignment ค่า role ให้กับ user ต่างๆ ดังนี้

- user-a : admin
- user-b : editor
- user-c : author
- user-d : ไม่ต้องให้สิทธิ์ เอาไว้ทดสอบ

ในการสร้าง role เราจะเรียกใช้งานผ่าน `Yii::$app->authManager` เพื่อทำการสร้าง rbac โดยมีฟังก์ชันหลักๆ ที่ใช้งานดังนี้

- `removeAll()` เป็นฟังก์ชัน clear ข้อมูล rbac ทั้งหมด โปรดระวังการใช้งานด้วยครับ
- `createRole()` เป็นคำสั่งที่เอาไว้สร้าง role
- `add()` เป็นคำสั่งบันทึก role,rule,permission ตามที่กำหนด
- `addChild()` สามารถกำหนดให้ role,permission ที่ต้องการอยู่ภายใต้ role ใหนก็ได้
- `assign()` เป็นคำสั่งที่เอาไว้มอบ role ให้กับ user ที่ต้องการ

ให้ทำการสร้าง rbac ตามโค้ดด้านล่างนี้

```php
<?php
//...
public function actionInit(){

  $auth = Yii::$app->authManager;
  $auth->removeAll();
  Console::output('Removing All! RBAC.....');

  $manageUser = $auth->createRole('ManageUser');
  $manageUser->description = 'สำหรับจัดการข้อมูลผู้ใช้งาน';
  $auth->add($manageUser);

  $author = $auth->createRole('Author');
  $author->description = 'สำหรับการเขียนบทความ';
  $auth->add($author);

  $editor = $auth->createRole('Editor');
  $editor->description = 'สำหรับการตรวจสอบบทความ';
  $auth->add($editor);
  $auth->addChild($editor, $author);

  $admin = $auth->createRole('Admin');
  $admin->description = 'สำหรับการดูแลระบบ';
  $auth->add($admin);

  $auth->addChild($admin, $editor);
  $auth->addChild($admin, $manageUser);

  $auth->assign($admin, 1);
  $auth->assign($editor, 2);
  $auth->assign($author, 3);

  Console::output('Success! RBAC roles has been added.');

}
```

จากนั้นทำการรันคำสั่งเพื่อสร้าง rbac ตาที่ได้ออกแบบไว้

```
php yii rbac/init
```
จะได้ผลลัพธ์ดังนี้

```
Removing All! RBAC.....
Success! RBAC roles has been added.
```

หลังจากนั้น ให้เราทำการตรวจสอบข้อมูลตารางที่ `auth_item` จะพบว่ามีข้อมูลขึ้นมาแล้ว 4 แถวซึ่งก็คือ Role ที่เราได้สร้างไว้นั่นเอง

```sql
select name,type,description  from auth_item;
+------------+------+-----------------------------------------+
| name       | type | description                             |
+------------+------+-----------------------------------------+
| Admin      |    1 | สำหรับการดูแลระบบ                         |
| Author     |    1 | สำหรับการเขียนบทความ                      |
| Editor     |    1 | สำหรับการตรวจสอบบทความ                   |
| ManageUser |    1 | สำหรับจัดการข้อมูลผู้ใช้งาน                    |
+------------+------+-----------------------------------------+
```
ในส่วนของ auth_item_child ที่เป็นตารางที่บอกว่า role ใหนอยู่ภายใต้ role อะไร

```sql
mysql> select * from auth_item_child;
+--------+------------+
| parent | child      |
+--------+------------+
| Editor | Author     |
| Admin  | Editor     |
| Admin  | ManageUser |
+--------+------------+
3 rows in set (0.00 sec)
```

และสุดท้าย `auth_assignment` จะเป็นตารางที่บอกว่า user คนใหนมีสิทธิ์อะไรหรือได้รับ role  อะไร

```sql
mysql> select * from auth_assignment;
+-----------+---------+------------+
| item_name | user_id | created_at |
+-----------+---------+------------+
| Admin     | 1       | 1440296691 |
| Editor    | 2       | 1440296691 |
| Author    | 3       | 1440296691 |
+-----------+---------+------------+
3 rows in set (0.00 sec)
```

> สังเกตุ เราไมได้ให้สิทธิ์ user-d เพราะจะเอาไว้ทดสอบเข้าใช้งานในกรณีคนที่ไม่มีสิทธิ์

เสร็จเรียบร้อย การสร้าง RBAC มันง่ายๆ แค่นี้เอง (แต่กว่าจะเข้าใจก็เล่นเอาเหนื่อยเหมือนกัน อาจจะด้วยภาษาอังกฤษอันอ่อนด้อยของผม)  หลังจากนี้ก็เป็นการตั้งค่าให้ controller ต่างๆ และการสร้างฟอร์ม assignment role ให้กับ user เพื่อให้เราจัดการสิทธิ์ได้ง่ายๆ


# รู้จักกับ Access Control Filter

ก่อนที่เราจะทำการเรียกใช้งาน rbac ที่เราได้สร้างขึ้น เราต้องทำความเข้าใจกับ  Access Control Filter ใน Controller ก่อน ซึ่งปกติค่า role default อยู่ 2 ตัว ที่เราสามารถเรียกใช้ได้เลยคือ

- ผู้ใช้งานที่ล็อกอิน (@)
- ผู้ใช้งานที่ไม่ได้ล็อกอิน (?)

 โดยทั้ง 2 ตัวนี้เราสามารถกำหนดค่าในส่วน Access Control ของ Controller ได้ลองดูตัวอย่าง

```php
<?php
use yii\web\Controller;
use yii\filters\AccessControl;

class SiteController extends Controller
{
    public function behaviors()
    {
        return [
            'access' => [
                'class' => AccessControl::className(),
                'only' => ['login', 'logout', 'signup'],
                'rules' => [
                    [
                        'allow' => true,
                        'actions' => ['login', 'signup'],
                        'roles' => ['?'],
                    ],
                    [
                        'allow' => true,
                        'actions' => ['logout'],
                        'roles' => ['@'],
                    ],
                ],
            ],
        ];
    }
    // ...
}
```

Access Control Filter เป็นส่วนของการกำหนดค่าให้แต่ละ action ว่า Role ใหนสามารถเข้าใช้งาน action อะไรได้บ้าง เราสามารถกำหนดค่าเพื่อเปิดใช้งานผ่าน `function behaviors()` ของ Controller ได้เลยโดยกำหนดภายใน property `access` หลังจากนั้นก็สามารถระบุได้เลยว่า action ใหนให้เข้าใช้งานยังไง ตัวอย่างเช่น action  `logout` ต้อง login ก่อนถึงจะทำการเรียกใช้งาน logout ได้ จะสังเกตว่าเราจะใช้ roles เป็นเครื่องหมาย `@` แปลว่าต้องเป็นคนที่ login มาแล้วเท่านั้น ซึ่งในส่วนของ `actions` เราจะระบุกี่ action ก็ได้

```php
[
    'allow' => true,
    'actions' => ['logout'],
    'roles' => ['@'],
],
```

ส่วนถ้ายังไม่ login ก็ใช้งานได้ 2 action คือ 'login', 'signup' และตรง roles ก็กำหนดค่าเป็น `?` เพื่อระบุว่าเป็นใครก็ได้ที่ยังไม่ได้ login

```php
[
    'allow' => true,
    'actions' => ['login', 'signup'],
    'roles' => ['?'],
],
```

นี่เป็นการใช้งาน Access Control Filter แบบปกติทั่วไป เราจะสามารถควบคุมการเข้าใช้งานได้แค่ ล็อกอิน,ไม่ล็อกอิน  แต่ถ้าหากเราจะควบคุมการเข้าใช้งานหลังจากล็อกอินแล้ว เช่น ผู้ใช้งานบางคนอาจมีสิทธิ์การเข้าใช้งานที่แตกต่างกัน จะไม่สามารถทำได้ ซึ่ง RBAC จะมาช่วยตรงนี้

RBAC ที่เราจะใช้งานก็แค่เปลี่ยน role default `@`,`?` ให้เป็นชื่อ Role ตามที่เราได้ตั้งขึ้น

# สร้าง Blog และเรียกใช้งาน RBAC

เราจะสร้าง Controller เพื่อใช้ในการทดสอบ RBAC ที่เราสร้างขึ้นโดยจะทำระบบ blog ง่ายๆ ให้ทำการสร้างตาราง blog ดังนี้

```sql
CREATE TABLE `blog` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `title` varchar(255) DEFAULT NULL COMMENT 'ชื่อเรื่อง',
  `content` text COMMENT 'เนื้อหา',
  `category` int(11) DEFAULT NULL COMMENT 'หมวดหมู่',
  `tag` varchar(255) DEFAULT NULL COMMENT 'คำค้น',
  `created_at` int(11) DEFAULT NULL COMMENT 'สร้างวันที่',
  `created_by` int(11) DEFAULT NULL COMMENT 'สร้างโดย',
  `updated_at` int(11) DEFAULT NULL COMMENT 'แก้ไขวันที่',
  `updated_by` int(11) DEFAULT NULL COMMENT 'แก้ไขโดย',
  PRIMARY KEY (`id`),
  KEY `title` (`title`),
  FULLTEXT KEY `content` (`content`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

จากนั้นทำการ Gii model ไว้ที่ common/models เพื่อให้สามารถใช้งานได้ทั้ง backend,frontend

![](/img/rbac-db-gii-model.png)


เมื่อ Gii model Blog เสร็จเรียบร้อยให้เราทำการ Gii CRUD Blog โดยให้ gii ไว้ที่ frontend

![](/img/rbac-db-gii-crud.png)

> สังเกตผมจะไม่เอา BlogSearch ไว้ที่ common นะครับ เพราะในการใช้งาน search มันอาจมีการตั้งค่าหรือมีการเรียกใช้ที่แตกต่างกัน เงื่อนไขในการค้นหาต่างๆ ไม่เหมือนกัน  ถ้าใช้ที่ frontend ก็ gii ไว้ที่ frontend เลย ถ้า backend ก็ gii ไว้ที่ backend เลยจะให้เราทำงานง่ายกว่า เพราะไม่ต้องใช้ไฟล์เดียวกัน

ปรับแต่ง model ที่ common/models/Blog.php เพื่อให้บันทึกรหัส created_by,updated_by โดยจะใช้ BlameableBehavior[ ดูรายละเดียดเพิ่มเติมที่นี่](https://dixonsatit.github.io/2015/06/17/blameable-behavior.html) ส่วน created_at,updated_at ใช้ TimestampBehavior [ดูรายละเอียดได้ที่นี่](https://dixonsatit.github.io/2015/06/14/model-behaviors.html) ทั้ง 2 ตัวนี้ระบบจะทำให้อัตโนมัติเราไม่ต้องทำอะไรเลยแค่ตั้งค่าเรียกใช้งานที่ model เลย

เปิดไฟล์ Blog.php ขึ้นมา แล้วทำการเรียกใช้งาน BlameableBehavior,TimestampBehavior จากนั้นตั้งค่าเปิดใช้งานที่ฟังก์ชัน `behaviors()`

```php
<?php

use yii\behaviors\BlameableBehavior;
use yii\behaviors\TimestampBehavior;

// .....

class Blog extends \yii\db\ActiveRecord
{

    public function behaviors(){
      return [
        BlameableBehavior::className(),
        TimestampBehavior::className()
      ];
    }

  //...
}

```

จากนั้นไปที่ /frontend/views/blog/\_blog.php ลบฟิวด์ที่ไม่ได้ใช้ออกไปคือ created_at,created_by,updated_at,updated_by เพราะเราใช้ behaviors เป็นตัวกรอกข้อมูลให้เราเอง ฟอร์มก็จะเหลือ 4 ฟิวด์ดังนี้

```php
<?php

use yii\helpers\Html;
use yii\widgets\ActiveForm;

/* @var $this yii\web\View */
/* @var $model common\models\Blog */
/* @var $form yii\widgets\ActiveForm */
?>

<div class="blog-form">

    <?php $form = ActiveForm::begin(); ?>

    <?= $form->field($model, 'title')->textInput(['maxlength' => true]) ?>

    <?= $form->field($model, 'content')->textarea(['rows' => 6]) ?>

    <?= $form->field($model, 'category')->textInput() ?>

    <?= $form->field($model, 'tag')->textInput(['maxlength' => true]) ?>

    <div class="form-group">
        <?= Html::submitButton($model->isNewRecord ? Yii::t('app', 'Create') : Yii::t('app', 'Update'), ['class' => $model->isNewRecord ? 'btn btn-success' : 'btn btn-primary']) ?>
    </div>

    <?php ActiveForm::end(); ?>

</div>

```

ให้ทดลอง login แล้วเข้าใช้งานที่ blog ว่าเข้าใช้งานได้หรือไม่ ทดลอง เพิ่ม,ลบ,แก้ไขข้อมูลดู หากสังเกตุจะมีข้อมูล created_at,created_by ขึ้นมาแล้ว

```sql
mysql> select * from blog;
+----+-------+---------+----------+------+------------+------------+------------+------------+
| id | title | content | category | tag  | created_at | created_by | updated_at | updated_by |
+----+-------+---------+----------+------+------------+------------+------------+------------+
|  1 | sdf   | sdf     |        1 | 1    | 1440302545 |          3 | 1440302545 |          3 |
+----+-------+---------+----------+------+------------+------------+------------+------------+
1 row in set (0.00 sec)
```

ให้ไปที่ไฟล์ BlogCongroller.php เราจะตั้งค่า rbac โดยใช้ข้อมูลจากที่เราได้ออกแบบไว้ข้างต้นคือ

![](/img/rbac.jpg)

เปิดไฟล์ BlogCongroller.php ซึ่งของเดิมจะยังไม่มีการเรียกใช้งาน Access Control Filter

```php
<?php
//...

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

    //...

}
```

ให้ทำการ use yii\filters\AccessControl;  เพิ่มใช้กำหนดค่าในการเข้าใช้งาน action ต่างๆ ว่า role ใหนให้เค้า action อะไรบ้าง

```php
<?php

namespace frontend\controllers;

use Yii;
use common\models\Blog;
use frontend\models\BlogSearch;
use yii\web\Controller;
use yii\web\NotFoundHttpException;
use yii\filters\VerbFilter;
use yii\filters\AccessControl; //<<------ Use

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
          'access'=>[
            'class'=>AccessControl::className(),
            'rules'=>[
              [
                'allow'=>true,
                'actions'=>['index','create','update'],
                'roles'=>['Author']
              ],
              [
                'allow'=>true,
                'actions'=>['delete'],
                'roles'=>['Editor']
              ]
            ]
          ]
      ];
  }

    //...

}
```

ถ้าดูตามโค้ดจะเห็นว่า เรากำหนดให้ index,create,update ให้เฉพาะ `Author` ส่วน action delete ให้เฉพาะ role `Editor` ขึ้นไป ซึ่งก็จะเป็นลำดับตามภาพที่เราได้กำหนดค่าไว้

- user-a : admin
- user-b : editor
- user-c : author
- user-d : ไม่ได้กำหนด


## ลองใช้งาน user-d
ลอง login ด้วย user-d และลองเข้า blog/index ดูจะพบว่าเข้าไม่ได้ เพราะว่าไม่ได้รับสิทธิ์ในการใช้งาน ไม่ว่าจะเข้า action อะไรก็ไม่สามารถเข้าได้ แต่อย่าลืม rbac จะรู้จักแค่ action ที่เรากำหนดไว้ใน actions=>['xxx','xxx'] เท่านั้นนะครับ

> อย่าลืมกำหนด property `'action'=>[....]` ใน rules ให้ครบทุกตัวด้วยนะครับ มี action กี่ตัวก็ระบุให้ครบ

![](/img/rbac-db-forbidden.png)

## ลองใช้งาน user-c

ต่อไปให้ลอง login ด้วย user-c จะเข้าได้ปกติ action index,create,view,update  ตามที่เรากำหนดไว้ใน rules ของ controller

![](/img/rbac-db-blog.png)

แต่เราจะไม่สามารถเข้าใช้งาน action Delete ได้ เพราะเรากำหนดว่าต้องเป็น role Editor ขึ้นไปถึงจะสามารถลบได้

![](/img/rbac-db-forbidden.png)

## ลองใช้งาน user-b,user-a

ส่วน user-a,user-b ก็จะสามารถเข้าได้เช่นกันและยังสามารถลบได้อีกด้วย เพราะเราระบุว่า user ที่จะลบได้ต้องเป็น สิทธิ์ editor ขึ้นไป ถึงแม้ว่าในส่วน access ของ Controller จะไม่ได้ระบุ role admin,editor ไว้ เพราะ author นั้นอยู่ภายใต้ editor และ admin อีกทีตามลำดับมันจึงสามารถเข้าใช้งานได้นั่นเอง

![](/img/rbac-db-blog.png)

สามารถลบข้อมูลได้

![](/img/rbac-db-confirm-delete.png)

![](/img/rbac-db-delete.png)

ในการตั้งค่า Access Control Filter และ RBAC กับ Controller ก็จะมีประมาณนี้

# สร้าง Rule เพื่อเช็คว่าเป็นเจ้าของบทความหรือไม่

ในการใช้งานจริงๆ ถ้าหากเราไม่ใช่คนสร้างบทความ ก็ไม่ควรจะไปแก้ไขบทความหรือลบบทความคนอื่นๆ ได้ ควรจะทำได้เฉพาะของตนเอง  ดังนั้นเราจะสร้าง Rule ขึ้นมา 1 ตัวเพื่อเอาไว้เช็คว่าเป็นเจ้าของบทความจริงหรือไม่ โดยตรวจสอบ user_id ที่ login เข้ามากับ created_by ที่อยู่ในตาราง blog หากเป็นคนๆ เดียวกันก็อนุญาติให้แก้ไขได้

Rule นั้นจะทำงานร่วมกัน Role ถ้าจะให้เข้าใจง่ายๆ Rule เป็นตัวช่วยที่ทำให้ Role ปกติสามารถเขียนฟังก์ชันและรับค่าเพื่อนำไปตรวจสอบตามเงื่อนไขของเราได้

> ไม่รู้ว่าผมเข้าใจถูกหรือปล่าวนะ แต่เท่าที่ทำมามันประมาณนี้ ผิดถูกยังไงแนะนำด้วยนะครับ

เราจะทำการสร้าง Rule ชื่อ AuthorRule.php เพื่อทำการเช็คว่าผู้ใช้งานเป็นเจ้าของบทความหรือไม่ ให้ทำการสร้างไฟล์ไว้ที่  common/rbac/AuthorRule.php

> หากไม่มีโฟลเดอร์ rbac ให้สร้างได้เลย

จากนั้นทำการสร้าง Rule ตามโค้ดด้านล่าง

```php
<?php
namespace common\rbac;

use yii\rbac\Rule;

class AuthorRule extends Rule
{
    public $name = 'isAuthor';

    public function execute($userId, $item, $params)
    {
        return isset($params['blog']) ? $params['blog']->created_by == $userId : false;
    }
}
 ?>

```

Rule นี้มีหน้าที่รับค่าเข้า model Blog เข้ามาเพื่อตรวจสอบกับฟิวด์ created_by ตรงกับ $userId ที่ login ในตอนนี้หรือไม่ ถ้าใช้ก็ return เป็น true ซึ่งเท่ากับยอมให้เข้าใช้งาน แต่ถ้าหากเป็น false ก็จะ แสดง error forbidden ออกไป ส่วนค่า $userId นั้นมันส่งเข้ามาให้อัตโนมัติเราแค่ตั้งชื่อแล้วก็เรียกใช้งานได้เลย

ในการเรียกใช้งานง่ายมากๆ  แค่ใช้ Yii::$app->user->can() เป็นตัวตรวจสอบให้ว่ามีสิทธ์หรือไม่

- ถ้าใช่ จะ return เป็น true  
- ถ้าไม่จะ  return เป็น false

และตัวฟังชันจะทำการรับค่า params 2 ตัวคือ

- ลำดับแรก เป็นชื่อ Rule
- ลำดับที่สอง เป็นค่า parameter ที่เราจะส่งเข้าไปเพื่อตรวจสอบ

ต่อไปทำการแก้ไข RbacController ใหม่ดังนี้

```php
<?php
namespace console\controllers;

use Yii;
use yii\helpers\Console;

class RbacController extends \yii\console\Controller {

  public function actionInit(){

      $auth = Yii::$app->authManager;
      $auth->removeAll();
      Console::output('Removing All! RBAC.....');

      $updateBlog = $auth->createPermission('UpdateBlog');
      $updateBlog->description = 'แก้ไขบทความ';
      $auth->add($updateBlog);

      $authorRole =  new \common\rbac\AuthorRule;
      $auth->add($authorRole);

      $updateOwnBlog = $auth->createPermission('UpdateOwnBlog');
      $updateOwnBlog->description = 'แก้ไขเฉพาะบทความของตัวเอง';
      $updateOwnBlog->ruleName = $authorRole->name;
      $auth->add($updateOwnBlog);
      $auth->addChild($updateOwnBlog,$updateBlog);

      $manageUser = $auth->createRole('ManageUser');
      $manageUser->description = 'สำหรับจัดการข้อมูลผู้ใช้งาน';
      $auth->add($manageUser);

      $author = $auth->createRole('Author');
      $author->description = 'สำหรับการเขียนบทความ';
      $auth->add($author);
      $auth->addChild($author, $updateOwnBlog);

      $editor = $auth->createRole('Editor');
      $editor->description = 'สำหรับการตรวจสอบบทความ';
      $auth->add($editor);
      $auth->addChild($editor, $author);

      $admin = $auth->createRole('Admin');
      $admin->description = 'สำหรับการดูแลระบบ';
      $auth->add($admin);

      $auth->addChild($admin, $editor);
      $auth->addChild($admin, $manageUser);

      $auth->assign($admin, 1);
      $auth->assign($editor, 2);
      $auth->assign($author, 3);

      Console::output('Success! RBAC roles has been added.');
  }

}
?>

 ```

```php
<?php
if (\Yii::$app->user->can('updatePost', ['post' => $post])) {
    // update post
}
```
