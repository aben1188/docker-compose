## 一、准备工作

### 安装docker、docker-compose

可使用[docker一键安装脚本](https://github.com/aben1188/docker-install-sh)进行安装。
  
### 下载git仓库

`git clone https://github.com/mochat-cloud/mochat`

`git clone https://github.com/mochat-cloud/docker-compose`

或从gitee克隆下载(国内速度更快)：

`git clone https://gitee.com/mochat/mochat`

`git clone https://gitee.com/mochat/docker-compose`

    
**注意：**

1、mac版`docker.desktop`版本推荐3.0以上，以避免内存飙高问题；
  
2、两个仓库必须克隆下载到同一个目录下(假设目录为`~/my-mochat`)：

　　`mochat-cloud/mochat` 下载目录为 `~/my-mochat/mochat`；
  
　　`mochat-cloud/docker-compose` 下载目录为 `~/my-mochat/docker-compose`。


### 解析域名

这里假设一级域名为`yourdomain.com`，需修改为你自己的真实域名。

```
backend.yourdomain.com  （用于后端api接口服务）
dashboard.yourdomain.com  （用于管理后台）
operation.yourdomain.com  （用于营销工具）
sidebar.yourdomain.com  （用于侧边栏）
```

**注意：**

1、如果你仅在本地浏览(即本机浏览)，则无需解析，直接在系统hosts文件中添加如下内容即可(推荐使用SwitchHosts来修改hosts)：

```  
127.0.0.1 backend.yourdomain.com  
127.0.0.1 dashboard.yourdomain.com  
127.0.0.1 operation.yourdomain.com  
127.0.0.1 sidebar.yourdomain.com
```

2、api.mo.chat为系统占用域名，请避免使用。
  
3、.dev与.app域名为chrome浏览器独家使用域名，使用chrome浏览器时请避免使用；

4、使用非本地容器MySQL时，可以设置`MYSQL_CONNECT_TYPE=cloud`，并修改`CLOUD_MYSQL_*`相应属性即可。

### 放行端口

在宿主机安全组和防火墙中放行如下端口。

```
80 （用于NGINX_HTTP_HOST_PORT）  
443 （用于NGINX_HTTPS_HOST_PORT）  
3306 （用于MYSQL_HOST_PORT）  
6379 （用于REDIS_HOST_PORT）  
9501 （用于PHP_HYPERF_PORT）
```

**注意：**

1、如果你仅在本地浏览(即本机浏览)，则无需在宿主机安全组和防火墙中放行上述端口(即便是非本地浏览，如果无需远程访问mysql和redis，3306和6379端口也无需放行)；

2、如果宿主机的相应端口已被占用，请改为未占用端口；
    
3、如果http协议默认的80端口、https协议默认的443端口改为了其他端口，访问时切记带上端口，比如：将80端口改为了8080端口，则访问时应为：`http://dashboard.yourdomain.com:8080`。


## 二、修改配置

`cd ~/my-mochat/docker-compose`

`cp .env.example .env`  #根据自己的情况，修改相应配置，详见下面的提示

**注意：**

1、必须在项目初始化之前修改`docker-compose/.env`文件，项目初始化之后再来修改可能会不生效，除非停止并删除容器、删除数据库文件（`docker-compose/data/mysql/*`、`docker-compose/data/redis/*`；注：若需要push到git仓库，则不应删除两个目录中的.gitignore文件）、删除install.lock文件（`docker-compose/services/mochat_init/install.lock`）、删除缓存（`mochat/api-server/runtime/*`），然后重新构建容器（`docker-compose build`）、启动容器（`docker-compose up`）才能生效。

2、项目初始化：在执行`docker-compose build`构建完相关容器后，第一次执行`docker-compose up`时会对项目进行一次初始化，初始化完成后，会生成`docker-compose/services/mochat_init/install.lock`文件，之后再次执行`docker-compose up`时，mochat_init容器运行时检测到存在该install.lock文件的话，则不会再进行初始化，除非删除该install.lock文件。

```
# 以下端口部分，如果使用默认端口，则无需修改

# 宿主机映射到nginx容器的http协议端口
NGINX_HTTP_HOST_PORT=80
# 宿主机映射到nginx容器的https协议端口
NGINX_HTTPS_HOST_PORT=443

# 宿主机映射到mysql容器的访问端口
MYSQL_HOST_PORT=3306

# 宿主机映射到redis容器的访问端口
REDIS_HOST_PORT=6379


# 以下登录账户与密码部分，修改为你自己的账户名和密码

# mochat超级管理员手机号(即账户名)，修改为你自己的手机号
MOCHAT_ADMIN=18888888888
# mochat超级管理员密码，修改为你自己的密码
MOCHAT_PASSWORD=123456


# 以下域名部分，需修改为前面已解析过的相应域名

# mochat.dashboard域名，不包含http://
DASHBOARD_URL=dashboard.yourdomain.com

# mochat.sidebar域名，不包含http://
SIDEBAR_URL=sidebar.yourdomain.com

# mochat.operation域名，不包含http://
OPERATION_URL=operation.yourdomain.com

# mochat.api-server域名，不包含http://
API_SERVER_URL=backend.yourdomain.com
```

`cp docker-compose.sample.yml docker-compose.yml`  #默认无需修改，尽量不要修改该文件


## 三、构建镜像

```
cd ~/my-mochat/docker-compose
docker-compose build
```


## 四、运行容器

```
cd ~/my-mochat/docker-compose
docker-compose up
```

执行上述命令后，将在宿主机前台运行；如果需要终止运行，可以使用快捷键`ctrl+c`。

如果需要在宿主机后台运行，则可以加上-d参数（在后台运行将无法实时看到运行情况，建议在刚开始时不要在后台运行，待确认运行无误后再改为后台运行）：

```
docker-compose up -d
```

执行`docker-compose up`后，dashboard、sidebar、operation、mochat_init容器运行完后会自动退出，显示如下信息：

```
dashboard exited with code 0
sidebar exited with code 0
operation exited with code 0
mochat_init exited with code 0
```

这属于正常现象，不属于错误；并且这四个容器必须都显示“exited with code 0”或“Exited (0)”之后，才能正常访问并登录MoChat系统。

这是因为，mochat_init容器仅用于初始化MoChat系统，而dashboard容器、sidebar容器、operation容器仅用于编译MoChat系统的前端文件，编译完之后这些容器即会终止运行；换言之，这些容器终止运行之后才表明前端文件编译完成了，这时候MoChat系统才可以正常接受访问，否则将会显示500错误。

成功启动运行的时间根据主机配置、网络速度等因素的不同，时间可能会比较长，请耐心等待。

成功启动运行后，执行`docker-compose ps`命令，即可查看容器运行详情。

```
docker-compose ps  

Name          Command                          State    Ports                  
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

如果初始化失败，可执行如下命令重新初始化。

```
cd ~/my-mochat/docker-compose
rm ./services/mochat_init/install.lock
docker-compose up mochat_init
```

**注意：**

`mochat/dashboard/.env`、`mochat/sidebar/.env`和`mochat/operation/.env`，这三个.env文件默认一般无需修改，会在初始化过程中使用`docker-compose/.env`中的设置自动修改。

不过需要特别注意的是，这三个.env文件中backend服务(即api-server)配置项`VUE_APP_API_BASE_URL`的域名前面默认没有加上“http:”，导致访问`dashboard.yourdomain.com`时，你将只会看到“Hello Mochat”(访问`sidebar.yourdomain.com`、`operation.yourdomain.com`时类似)，而无法看到MoChat系统的登录界面。因此你仍需要手动修改一下。

手动修改步骤如下：

1、先停止所有MoChat相关容器：如果之前是在宿主机前台运行的话，则执行快捷键`ctrl+c`即可，否则需要执行：

```
docker-compose stop backend nginx mysql redis
```

2、依次打开`mochat/dashboard/.env`、`mochat/sidebar/.env`和`mochat/operation/.env`文件，将`VUE_APP_API_BASE_URL=//backend.yourdomain.com`修改为：

```　　
VUE_APP_API_BASE_URL=http://backend.yourdomain.com
```

3、再次执行`docker-compose up`或`docker-compose up -d`即可。

期待官方早点将这个错误修正过来，这样将可以省略上述步骤。


## 五、热更新

如果你需要进行二次开发，为了实现自动监控文件而实时反映改动，可按如下方法更改。

**后端PHP热更新：** 可在`docker-compose/services/php/Dockerfile`文件内将`php /opt/www/bin/hyperf.php start`改为`php /opt/www/bin/hyperf.php server:watch`。

**前端热更新：** 建议在宿主机执行`npm run dev`，接口调试地址为：`http://backend.yourdomain.com`。

**注意：** 这里的热更新仅限于你在本地开发时的自动更新，升级更新为MoChat官方的最新版本，请参照[官方文档中的升级说明](https://mochat.wiki/upgrade/)进行。


## 六、访问

在浏览器输入：`http://dashboard.yourdomain.com`，正常的话，你应该可以看到MoChat系统的登录界面。

输入你之前在`docker-compose/.env`文件设置的用户名和密码（如果你未修改，则默认为：`18888888888` / `123456`）进行登录。

成功登录之后，在`系统设置` -> `授权管理` 中点击 `添加企业微信号`（如果您没有企业微信号，您可以到企业微信官网网站注册调试用的`企业微信号`）。
