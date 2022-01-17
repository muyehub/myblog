---
layout:     post
title:      "Homestead完整的切换php版本"
date:       2020-01-10 15:14:46 +0800
author:     "max"
---
相信大多数人搜到的都是这篇文章
https://learnku.com/articles/16881

文章中说, 使用 homestead 自带的工具 update-alternatives 进行切换就可以了, 但是我在使用后 php 的版本是切换了, 但是 pecl 安装扩展的时候安装的还是20170817目录, 这是 php7.2 的扩展包默认目录, 使用 php-config 看的时候也是 php7.2

如何把phpize, pecl, php-config 都切换为 php7.1 呢? 请看下面

```
sudo update-alternatives --set php /usr/bin/php7.1
sudo update-alternatives --set phar /usr/bin/phar7.1
sudo update-alternatives --set phar.phar /usr/bin/phar.phar7.1
sudo update-alternatives --set phpize /usr/bin/phpize7.1
sudo update-alternatives --set php-config /usr/bin/php-config7.1
```

如果你在切换之前装了错误的扩展包到其他版本的目录下, 这样切换完之后卸载重新装, 就 ok 了
