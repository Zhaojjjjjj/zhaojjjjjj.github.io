- 安装 PHP7.4 版本不成功
> Error: php@7.4 has been disabled because it is a versioned formula!

> 因为php7.4官方已经不再维护，所以Hombrew将该php版本移出了repository，所以安装不了。

- Solving problems
```
// 从第三方仓库中安装
// 将第三方仓库加入 brew
brew tap shivammathur/php
 
// 安装 PHP
brew install shivammathur/php/php@7.4
```
- PHP version switching
```
// brew-php-switcher 是通过 brew 安装的，安装方式如下：
brew install brew-php-switcher
 
// Usage
brew-php-switcher <version>  /  brew-php-switcher <version> -s
 
// Example
brew-php-switcher 5.6 -s
```