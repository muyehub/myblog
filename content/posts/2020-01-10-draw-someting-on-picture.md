---
layout:     post
title:      "在图片上添加文字或图片"
date:       2020-01-10 15:43:41 +0800
author:     "max"
---
#### intervention/image
这是一个第三方扩展, 用来处理图片的, 功能特别强大, 可以 cover 我们这篇文章提到的需求

安装文档: http://image.intervention.io/getting_started/installation

我们首先在项目中引入这个包:
```
$ composer require intervention/image
```

其次, laravel 中需要发布配置文件:
```
php artisan vendor:publish --provider="Intervention\Image\ImageServiceProviderLaravel5"
```

lumen 中需要修改 bootstrap.php, 在当中手动注册
```
$app->register(\Intervention\Image\ImageServiceProviderLumen::class);
```

该扩展默认使用的是 GD 库, 如果你想用 Imagick 扩展需要修改文件, config/image.php
```
<?php
    return [
        'driver'    =>  'imagick'
    ];
```

如果你不想要每次 use 都使用全路径, 可以按照官方所述将 Image 关键字写入 alias 中

- Laravel: config/app.php

```
$providers:

Intervention\Image\ImageServiceProvider::class

$aliases:

'Image' => Intervention\Image\Facades\Image::class
```

- Lumen: bootstrap/app.php

```
$app->withAliases(array('Intervention\Image\Facades\Image' => 'Image'));
```

#### 具体使用
```
use Image;

public function deal()
{
    $image = Image::make(public_path() . '/foo.jpg');
    $text = '我是天才';
    $image->text($text, 620, 200, function($font) {
        $font->file(public_path() . '/SimHei.ttf');
        $font->size(200);
        $font->color('#fdf6e3');
        $font->align('center');
        $font->valign('top');
        $font->angle(0);
    });
    return $image->response('jpg');
}
```

插入文字的方法是 text, 插入图片的方法是 insert, 如果文字插入后显示乱码, 考虑换一种字体, 支持中文的, 默认的好像就是乱码

官方文档: http://image.intervention.io/
