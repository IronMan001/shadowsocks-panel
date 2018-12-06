Anan Yang edited this page on 31 Aug 2016 · 35 revisions
1. 前言
使用lnmp.org一键包环境，出现的任何环境问题请自行解决问题。
出现问题时，请查询 WIKI 与 issues 已关闭或开启的问题，不要重复issue
若都无法解决可发邮件或issue或者QQ交流群（可能最快速有回复） 。
2. 安装面板步骤
获取 Shadowsocks Panel 并安装

# 方式一：克隆最新版本
cd /home/wwwroot/
git clone https://github.com/sendya/shadowsocks-panel.git
cd shadowsocks-panel

# 方式二：下载稳定版本（推荐）
# 前往 https://github.com/sendya/shadowsocks-panel/releases ，下载最新的release版本（当前版本：v1.2.0.B）
wget https://github.com/sendya/shadowsocks-panel/archive/sspanel-v1.2.0.B.zip -O shadowsocks-panel.zip
# 解压到 /home/wwwroot/shadowsocks-panel/
$ unzip -o -d /home/wwwroot/shadowsocks-panel/ shadowsocks-panel.zip
$ cd /home/wwwroot/shadowsocks-panel/

# 复制一份 ./Data/Config.simple.php 为 ./Data/Config.php
cp ./Data/Config.simple.php ./Data/Config.php
# 设定 Data 目录读写权限
chmod -R 777 ./Data/
# 配置数据库名及数据库账户密码(代码最下面)
vim ./Data/Config.php

# 开始执行自动安装，如果没开放php函数权限，可用此命令 php -d disable_functions='' index.php install
php index.php install
执行安装命令后直到看到此内容，才代表安装成功

All done~ Cheers! open site: http://yourdomain.com/

任何情况出现 Permission denied 请对该文件设定权限
任何情况出现 system() 报错，请允许php函数 system

请将NGINX/Apache的 网站根目录路径指向到 Public 而不是 shadowsocks-panel 目录（程序入口在 /Public/index.php 而非 /index.php）。

3. 配置路由规则
选择1. 配置NGINX伪静态规则

if (!-e $request_filename) {
    rewrite (.*) /index.php last;
}
选择2. 配置Apache伪静态规则

RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
在确定程序能正常访问之后，在进行CRON计划任务配置

$ crontab -e
$ * * * * * /usr/bin/curl https://yourdomain.com/cron
# 保存退出
其他
4. 面板管理员设置
可以在数据库表中admin表新增记录 uid 为你的账号uid， id无需填写
在后台 “用户管理” 中编辑用户，选择 是管理员 则将此账户设定为管理员
全新安装的面板系统，默认第一个注册的账户是管理员。
本程序不支持 SendCloud 请使用标准SMTP邮件 或者 MailGun
5. 全部账户启用
UPDATE `member` SET enable=1 WHERE 1=1
6. 全部账户设定到期时间
UPDATE `member` SET expireTime=时间戳 WHERE 1=1
请将时间戳 替换为Unix时间戳（数字）

7. NGINX 配置示例
server{
    listen 80;
    server_name sscat.it www.sscat.it;
    access_log /home/wwwlogs/sscat_nginx.log combined;
    index index.html index.htm index.php;
    
    if (!-e $request_filename) {
        rewrite (.*) /index.php last;
    }
    root /home/wwwroot/shadowsocks-panel/Public;

    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        if (!-f $document_root$fastcgi_script_name) {
            return 404;
        }
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
8. 服务端支持相关
本面板推荐使用 shadowsocks-rm
若您要使用 ssr 请使用本项目内链接提供的 ssr 对接版本
(嫌版本旧请自行对接，方法看最下面)
面板不支持 ssr 在线设置混淆，请直接在服务端设定完毕，将参数写在节点信息中
请不要发任何 ssr 相关 issue 至本项目。

9. 更换 Composer 国内源
如果你面板安装在国内主机上
可在根目录文件 composer.json 内最底部的 } 之内添加下面这一段。即可在国内享受快速的composer下载

,
   "repositories": {
    "packagist": {
       "type": "composer",
        "url": "http://packagist.phpcomposer.com"
    }
   }
10. lnmp 一键环境
安装教程(CentOS7)(需要有动手能力)
LinuxEye提供的一键环境oneinstack
(oneinstack安装的环境需要修改 /usr/local/php/etc/php.d/ext-opcache.ini 文件，将值 opcache.save_comments=1 修改保存，并且重启php-fpm)

a. 未支持的服务端对接
尚未支持的shadowsocks manyuser版本，请自行修改sql查询字符串即可与面板对接

将shadowsocks查询语句(一般存在于文件dbtransfer.py 或者 db_transfer.py)
port, passwd, u, d, t, transfer_enable, enable, switch 修改对应
port, sspwd, flow_up, flow_down, lastConnTime, transfer, enable 面板已舍弃 switch
其中 user 表在shadowsocks-panel中为 member

支持人员
@Sendya
@kookxiang
@Acris
以及各位开源的 dalao 们

Telegram讨论组：Join telegram group *该作者已删除电报账户
