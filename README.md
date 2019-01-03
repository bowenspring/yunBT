# YunBT
***
基于ThinkCMS的YunBT的多用户下载程序，支持Magnet和HTTP下载。每个单独用户支持10个任务，默认下载文件最大为10GB，可以在后台修改。下载完成后用户可以直接查看下载的文件,系统将使用ffmpeg将视频转码为720P的mp4格式的视频，用户可以分享下载完的任务。管理员可以添加用户的下载量及查看管理下载任务。  
  

测试站点:[www.yunbt.net](http://www.yunbt.net)  
测试站点默认下载限制单个下载小于1GB

## 截图  
分享密码    
![分享密码](https://i.loli.net/2018/12/23/5c1f95ae39ffb.png)
分享  
![分享](https://i.loli.net/2018/12/23/5c1f95ae983c4.png)
视频文件  
![视频文件](https://i.loli.net/2018/12/23/5c1f95aeb8f78.png)
下载列表  
![下载列表](https://i.loli.net/2018/12/23/5c1f95af2e1f8.png)  
后台转码页面  
![后台转码](https://i.loli.net/2018/12/24/5c2060774d0f1.png)
水印及切片设置  
![水印设置](https://i.loli.net/2019/01/03/5c2d7041dce4e.png)  
Dplayer播放器  
![Dplayer播放器](https://i.loli.net/2019/01/03/5c2d704219f57.png)  
## 安装

### 服务器环境
Linux Nginx Mysql Php Python 

#### LNMP安装
[lnmp](https://lnmp.org/install.html) 一键安装环境 
`wget http://soft.vpser.net/lnmp/lnmp1.5.tar.gz -cO lnmp1.5.tar.gz && tar zxf lnmp1.5.tar.gz && cd lnmp1.5 && ./install.sh lnmp`  
[详尽步骤](https://lnmp.org/install.html)

#### Redis

lnmp安装redis  
`./addons.sh install redis` 
[详尽步骤](https://lnmp.org/faq/addons.html)

#### Aria2
安装Aria2  
`apt-get update && apt-get install -y aria2 `

`cd /root`  

`mkdir .aria2 && cd .aria2`  

`wget http://webdir.cc/aria2.conf`  

`wget http://webdir.cc/dht.dat`  

`echo '' > /root/aria2.session`  

`screen -dmS aria2 aria2c --enable-rpc --rpc-listen-all=true --rpc-allow-origin-all -c `  

 
#### ffmpeg
`apt-get install ffmpeg`


#### PHP
php>7   
lnmp下php安装fileinfo插件  
lnmp1.4 安装php fileinfo扩展 方法  
- 第一步：在lnmp1.4找到php安装的版本  
使用命令 `tar   -jxvf   php-7.1.7.tar.bz2` 解压  
- 第二步： 在解压的php-7.1.7文件夹里找到fileinfo文件夹，然后使用命令 `cd  /home/xxx/lnmp1.4/src/php-7.1.7/ext/fileinfo`进入到fileinfo文件夹  
- 第三步：输入`/usr/local/php/bin/phpize` 得到数据  
- 第四步： 使用如下命令编译安装  
`./configure -with-php-config=/usr/local/php/bin/php-config`  
`make && make install`
   
- 第五步：再修改/usr/local/php/etc/php.ini  查找：extension = 再最后一个extension= 后面添加上extension = "fileinfo.so"   保存，执行`/etc/init.d/php-fpm restart` 重启。

#### Nginx  

nginx修改fastcgi.conf配置  
>lnmp下该文件在 `/usr/local/nginx/conf/fastcgi.conf`  

把其中的  
```
#fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/";
fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/../:/tmp/:/proc/";
```


Nginx 配置
>下文件为lnmp下的配置    

```
server
    {
        listen 80;
        #listen [::]:80;
        server_name www.yunbt.net yunbt.net;
        index index.html index.htm index.php default.html default.htm default.php;
        root  /home/wwwroot/www.yunbt.net/public;

	location / {
        rewrite ^/file/(.*) /file.php?file=$1 last;
   	if (!-e $request_filename) {
        	rewrite ^(.*)$ /index.php?s=/$1 last;
    	}
	}
	location /afile{
                internal;
                alias /home/wwwroot/www.yunbt.net/public/file;
        }
        #error_page   404   /404.html;

        # Deny access to PHP files in specific directory
        #location ~ /(wp-content|uploads|wp-includes|images)/.*\.php$ { deny all; }
	location ~* ^/(file|upload)/.*\.(php|php5)$ {  
		deny all; 
	}  
        include enable-php.conf;

        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
        {
            expires      30d;
        }

        location ~ .*\.(js|css)?$
        {
            expires      12h;
        }

        location ~ /.well-known {
            allow all;
        }

        location ~ /\.
        {
            deny all;
        }

        access_log  /home/wwwlogs/www.yunbt.net.log;
    }

```
#### Mysql
创建数据库名yunbt  
用户名yunbt  
密码a123456  
数据库导入yunbt.sql  

数据库配置 data/conf/database.php


#### python
python3  
pymysql  
`pip3 install pymysql`  
cron_ffmpeg.py
+ 44行:数据库配置
+ 50行:设置你的web路径  

cron_move.py  
+ 14行:设置你的web路径
+ 15行:视频文件[无须修改]
+ 74行:数据库配置  

cron_download.py  
+ 3行:限制下载最大值[单位GB]默认10GB   

cron_m3u8.py  
+ 10行:数据库配置
+ 17行:设置你的web路径[/public]


#### 权限修改
修改data文件夹下的权限  

`chmod -R 777 data/`  
`chmod -R 777 public/`  

#### cron

添加定时任务  
`crontab -e`
```
*/1 * * * * curl http://www.yunbt.net/portal/cron/download
*/3 * * * * python3 /home/wwwroot/www.yunbt.net/python/cron_move.py
*/1 * * * * python3 /home/wwwroot/www.yunbt.net/python/cron_ffmpeg.py
*/1 * * * * python3 /home/wwwroot/www.yunbt.net/python/cron_m3u8.py
*/30 * * * * python3 /home/wwwroot/www.yunbt.net/python/cron_download.py
```  
请替换其中www.yunbt.net 为你自己的域名  

### 管理

管理后台 your_domain.com/admin  
用户名 admin  
密码 a123456  





##未来计划
目前功能：
- 添加用户下载量
- 下载管理
- 修改当前最大下载文件量［默认10GB］
- 分享功能
- 视频转码  
- 工具下载  
- 后台转码控制
- Dplayer播放器
- 水印添加
- 视频切片

未来计划:  
1. ~~分享功能~~
2. ~~用户转码功能~~
3. 积分功能
4. 邀请功能
5. ~~工具下载~~
6. ~~水印添加~~
7. ~~M3U8添加~~
8. ~~自动播放转码~~



