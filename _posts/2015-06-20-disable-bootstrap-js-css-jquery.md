---
layout : post
title : การใช้ assetManager และปิดใช้งาน Bootstrap css,js,JQuery 
---

หากเราไม่ต้องการใช้งาน Boostrap css,js หรือ JQuery ที่มากับ Yii 2 เราสามารถปิดการใช้งานได้ เพราะบางทีอาจใช้ css อื่นๆ ที่ไม่ใช่ Bootstrap เช่น JqueryMobile, ExtJs เป็นต้น

การปิดการใช้งานจะใช้คอมโพเน็นส์ `assetManager` ซึ่งเป็นตัวเอาไว้จัดการ Assets ต่างๆ เพื่อเปลี่ยนค่าตัวเดิมที่มันใช้อยู่เช่น อยากใช้ jquery เวอร์ชั่นอืนๆ ที่เราโหลดมาเอง ซึ่งตัวอย่างนี้ผมเก็บไฟล์ jquery ไว้ที่โฟลเดอร์ web/js/jquery.min.js

```css
return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => [
                'sourcePath' => null,
                'basePath' => '@webroot',
                'baseUrl' => '@web',
                'js' => [
                    	'jquery.min.js',
                	]
                ],
            ],
        ],
    ],
];
```

หรือในกรณีถ้าอยากใช้ไฟล์จาก CDN Google เลยก็ได้

```css
return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
                'yii\web\JqueryAsset' => [
                    'sourcePath' => null, 
                    'js' => [
                        '//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js',
                    ]
                ],
            ],
        ],
    ],
];
```

หากเราต้องการปิดการใช้งานก็ง่ายมากๆ  ก็แค่ใช่เป็น array ว่างๆ เข้าไป

```css
return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
				'yii\web\JqueryAsset' => [
				    'js'=>[] // disable JQuery
		        ],
		        'yii\bootstrap\BootstrapPluginAsset' => [
		            'js'=>[] // disable JS Bootstrap
		        ],
		        'yii\bootstrap\BootstrapAsset' => [
		            'css' => [], // disable Css Bootstrap
		        ],
            ],
        ],
    ],
];
```

หรือสั้นๆ ก็ได้

```css
return [
    // ...
    'components' => [
        'assetManager' => [
            'bundles' => [
               'yii\web\JqueryAsset' => false,
               'yii\bootstrap\BootstrapPluginAsset' => false,
               'yii\bootstrap\BootstrapAsset' => false,
            ],
        ],
    ],
];
```

อ่านเพิ่มเติมได้ที่นี่ [http://www.yiiframework.com/doc-2.0/guide-structure-assets.html#customizing-asset-bundles](http://www.yiiframework.com/doc-2.0/guide-structure-assets.html#customizing-asset-bundles)
