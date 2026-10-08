```
// MySQL 严格模式
SET GLOBAL sql_mode='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';


// composer
composer install --ignore-platform-reqs
// 官方镜像源
sudo composer config -g repo.packagist composer https://packagist.org
// 阿里云镜像
sudo composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/


// natapp
// 到 natapp 文件夹内跑以下两个命令
chmod a+x natapp  
./natapp -authtoken=********(your token)


// Python 查看当下 conda 所处的环境及切换环境
// 查看
conda env list
// 切换
conda activate <your_env_name>


// PHP本地版本切换
// brew-php-switcher 是通过 brew 安装的，安装方式如下：
brew install brew-php-switcher
// 使用方式：
brew-php-switcher <version>  /  brew-php-switcher <version> -s
// 例如
brew-php-switcher 5.6 -s


// 宝塔链接仓库
git clone sshxxxxx .
```