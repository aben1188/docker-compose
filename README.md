## 一、前提条件

### 安装：docker、docker-compose

　　可使用[docker一键安装脚本](https://github.com/aben1188/docker-install-sh)进行安装。
  
### 克隆下载下面两个git仓库

　　git clone https://github.com/mochat-cloud/mochat.git

　　　　或使用github国内源加速下载：git clone https://github.com.cnpmjs.org/mochat-cloud/mochat.git

　　git clone https://github.com/mochat-cloud/docker-compose.git

　　　　或使用github国内源加速下载：git clone https://github.com.cnpmjs.org/mochat-cloud/docker-compose.git
    
**注意：**

1、mac版docker.desktop版本推荐3.0以上，以避免内存飙高问题；
  
2、两个项目必须克隆下载到同一个目录下：

　　mochat-cloud/mochat 下载目录为 /path/to/mochat；
  
　　mochat-cloud/docker-compose 下载目录为 /path/to/docker-compose。


### 解析要使用到的二级域名，包括(假设一级域名为yourdomain.com，需改为你的真实域名)：

　　backend.yourdomain.com  
　　dashboard.yourdomain.com  
　　operation.yourdomain.com  
　　sidebar.yourdomain.com
  
**注意：**

1、如果你仅在本地浏览，则无需解析，直接在系统hosts文件中添加如下内容即可(推荐使用SwitchHosts来修改hosts)：
  
　　127.0.0.1 backend.yourdomain.com  
　　127.0.0.1 dashboard.yourdomain.com  
　　127.0.0.1 operation.yourdomain.com  
　　127.0.0.1 sidebar.yourdomain.com

　　hosts文件在各操作系统中的路径：
  
　　　　Windows系统：C:\Windows\System32\drivers\etc\hosts  
　　　　类Unix系统(Linux、Mac、iOS、Android等)：/etc/hosts  
　　      
2、api.mo.chat为占用域名，请避免使用。
  
3、dev与.app域名为chrome域名，使用chrome时请避免使用；

4、使用非本地容器MySQL时，可以设置MYSQL_CONNECT_TYPE=cloud，并修改CLOUD_MYSQL_*相应属性即可。

### 在宿主机安全组和防火墙中放行如下端口：

　　80（NGINX_HTTP_HOST_PORT）  
　　443（NGINX_HTTPS_HOST_PORT）  
　　3306（MYSQL_HOST_PORT）  
　　6379（REDIS_HOST_PORT）  
　　9501（PHP_HYPERF_PORT）

**注意：**

1、如果宿主机的相应端口已被占用，请修改为未占用端口；
    
2、如果http协议默认的80端口、https协议默认的443端口修改为了其他端口，访问时切记带上端口，比如：将80端口改为了8080端口，则访问时应为：http://dashboard.yourdomain.com:8080。


## 二、修改配置

`cd /path/to/docker-compose`

`cp .env.example .env`  #根据自己的情况，修改相应配置，详见下面的提示

**注意：**

1、必须在项目初始化之前修改/path/to/docker-compose/.env文件，项目初始化之后再来修改可能会不生效，除非停止并删除容器、删除数据库文件（docker-compose/data/mysql/\*、docker-compose/data/redis/\*；注：若需要push到git仓库，则不应删除两个目录中的.gitignore文件）、删除install.lock文件（docker-compose/services/mochat_init/install.lock）、删除缓存（mochat/api-server/runtime/\*），然后重新构建容器（docker-compose build）、启动容器（docker-compose up）才能生效。

2、项目初始化：在执行docker-compose build构建完相关容器后，第一次执行docker-compose up时会对项目进行一次初始化，初始化完成后，会生成docker-compose/services/mochat_init/install.lock文件，之后再次执行docker-compose up时，mochat_init容器运行时检测到存在该install.lock文件的话，则不会再进行初始化，除非删除该install.lock文件。

```
# 如下端口部分，如果使用默认端口，则无需修改

# 宿主机映射到nginx容器的http协议端口
NGINX_HTTP_HOST_PORT=80
# 宿主机映射到nginx容器的https协议端口
NGINX_HTTPS_HOST_PORT=443

# 宿主机映射到mysql容器的访问端口
MYSQL_HOST_PORT=3306

# 宿主机映射到redis容器的访问端口
REDIS_HOST_PORT=6379

# mochat超级管理员手机号，可修改为你自己的手机号
MOCHAT_ADMIN=18888888888
# mochat超级管理员密码，可修改为你自己的密码
MOCHAT_PASSWORD=123456


# 以下域名部分，如果你在本地浏览，则使用本地回环IP(127.0.0.1)即可
# 否则，需修改为前面已解析过的相应域名

# mochat.dashboard域名，不包含http://
#DASHBOARD_URL=127.0.0.1
DASHBOARD_URL=dashboard.yourdomain.com
# mochat.sidebar域名，不包含http://
#SIDEBAR_URL=127.0.0.1
SIDEBAR_URL=sidebar.yourdomain.com
# mochat.operation域名，不包含http://
#OPERATION_URL=127.0.0.1
OPERATION_URL=operation.yourdomain.com
# mochat.api-server域名，不包含http://
#API_SERVER_URL=127.0.0.1
API_SERVER_URL=backend.yourdomain.com
```

`cp docker-compose.sample.yml docker-compose.yml`  #默认无需修改，尽量不要修改该文件


## 三、构建镜像

```
cd /path/to/docker-compose
docker-compose build
```

## 四、运行容器

```
cd /path/to/docker-compose
docker-compose up
```

执行上述命令后，将在宿主机前台运行；如果需要终止运行，可以使用快捷键：ctrl+c。

如果需要在宿主机后台运行，则可以加上-d参数（在后台运行将无法实时看到运行情况，建议在刚开始时不要在后台运行，待确认运行无误后再在后台运行）：

```
docker-compose up -d
```

**注意：**

执行docker-compose up时，dashboard、sidebar、operation、mochat_init容器运行完后会自动退出，显示如下信息：
```
　　dashboard exited with code 0
　　sidebar exited with code 0
　　operation exited with code 0
　　mochat_init exited with code 0
```
这属于正常现象，不属于错误；并且这四个容器必须都显示“exited with code 0”或“Exited (0)”之后，才能正常访问并登录MoChat系统。

这是因为，mochat_init容器仅用于初始化MoChat系统，而dashboard容器、sidebar容器、operation容器仅用于编译MoChat系统的前端文件，编译完之后这些容器即会终止运行；换言之，这些容器终止运行之后才表明前端文件编译完成了，这时候MoChat系统才可以正常接受访问。

成功启动运行的时间根据主机配置、网络速度等因素的不同，时间可能会比较长，请耐心等待。

```
docker-compose ps  

   Name                  Command               State                     Ports                  
------------------------------------------------------------------------------------------------
backend       /bin/sh -c sh -c "composer ...   Up       0.0.0.0:9501->9501/tcp                  
dashboard     /bin/sh -c sh -c "yarn ins ...   Exit 0        
operation     /bin/sh -c sh -c "yarn ins ...   Exit 0                                           
mochat_init   /bin/sh -c sh -c "/tmp/wai ...   Exit 0                                           
mysql         docker-entrypoint.sh mysqld      Up       0.0.0.0:3306->3306/tcp, 33060/tcp       
nginx         /docker-entrypoint.sh ngin ...   Up       0.0.0.0:443->443/tcp, 0.0.0.0:80->80/tcp
redis         docker-entrypoint.sh redis ...   Up       0.0.0.0:6379->6379/tcp                  
sidebar       /bin/sh -c sh -c "yarn ins ...   Exit 0
```

如果初始化失败，可重新初始化。

```
cd /path/to/docker-compose
rm ./services/mochat_init/install.lock
docker-compose up mochat_init
```

**特别注意：**

mochat/dashboard/.env、mochat/sidebar/.env和mochat/operation/.env，这三个.env文件默认一般无需修改，会在初始化过程中使用docker-compose/.env中的设置自动修改。

不过需要特别注意的是，其中api-server的端口默认被错误地设置为了nginx服务的宿主机映射端口(比如80)，而非backend服务的宿主机映射端口(比如9501)，因此仍需进行手动修改该端口，否则，会等同于访问：backend.yourdomain.com，你将会只能看到“Hello Mochat”，而无法看到MoChat系统的登录界面。

手动修改步骤如下：

1、先停止所有MoChat相关容器：如果之前是在宿主机前台运行的话，则执行快捷键ctrl+c即可，否则需要执行：docker-compose stop backend nginx mysql redis；

2、依次打开mochat/dashboard/.env、mochat/sidebar/.env和mochat/operation/.env文件，将VUE_APP_API_BASE_URL的值修改为：http://backend.yourdomain.com:9501 ，如下：
　　VUE_APP_API_BASE_URL=http://backend.a1b2.top:9501
  
3、再次执行docker-compose up或docker-compose up -d即可。


## 五、热更新

如果你需要进行二次开发，为了实现自动监控文件而实时反映改动，可按如下方法更改。

后端PHP热更新，可在 `./servers/php/Dockerfile` 文件内将 `php /opt/www/bin/hyperf.php start` 改为 `php /opt/www/bin/hyperf.php server:watch`

前端热更新建议在宿主机 `npm run dev`，接口调试地址为 `http://backend.yourdomain.com`。


## 六、访问

在浏览器输入：http://dashboard.yourdomain.com

输入你之前在docker-compose/.env文件设置的用户名和密码（如果你未修改，则默认为: `18888888888` / `123456`）

进入项目，在`系统设置` -> `授权管理` 中点击 `添加企业微信号`（如果您没有企业微信号，您可以到企业微信官网网站注册调试用的`企业微信号`）
