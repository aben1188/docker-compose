## 一、前提条件

* 已安装：docker、docker-compose

　　如果未安装docker、docker-compose，可使用[docker一键安装脚本](https://github.com/aben1188/docker-install-sh)进行安装。
  
* 克隆下载下面两个git仓库

　　git clone https://github.com/mochat-cloud/mochat.git

　　　　或使用github国内源加速下载：git clone https://github.com.cnpmjs.org/mochat-cloud/mochat.git

　　git clone https://github.com/mochat-cloud/docker-compose.git

　　　　或使用github国内源加速下载：git clone https://github.com.cnpmjs.org/mochat-cloud/docker-compose.git
    
```
注意：

　　1、mac版docker.desktop版本推荐3.0以上，以避免内存飙高问题；
  
　　2、两个项目必须克隆下载到同一个目录下：
　　　　mochat-cloud/mochat 下载目录为 /path/to/mochat；
　　　　mochat-cloud/docker-compose 下载目录为 /path/to/docker-compose。
```

* 解析要使用到的二级域名，包括(假设一级域名为yourdomain.com，需改为你的真实域名)：
　
　　backend.yourdomain.com
  
　　dashboard.yourdomain.com
  
　　operation.yourdomain.com
  
　　sidebar.yourdomain.com
  
```
注意：

　　1、如果你仅在本地浏览，则无需解析，直接在系统hosts文件中添加如下内容即可(推荐使用[SwitchHosts](https://github.com/oldj/SwitchHosts/blob/master/README_cn.md)来修改hosts)：
  
　　　　127.0.0.1 backend.yourdomain.com
　　　　127.0.0.1 dashboard.yourdomain.com
　　　　127.0.0.1 sidebar.yourdomain.com
　　　　127.0.0.1 operation.yourdomain.com

　　　　hosts文件在各操作系统中的路径：
　　　　　　Windows系统：C:\Windows\System32\drivers\etc\hosts
　　　　　　类Unix系统(Linux、Mac、iOS、Android等)：/etc/hosts
      
　　2、api.mo.chat为占用域名，请避免使用。
  
　　3、dev与.app域名为chrome域名，使用chrome时请避免使用；

　　4、使用非本地容器MySQL时，可以设置MYSQL_CONNECT_TYPE=cloud，并修改CLOUD_MYSQL_*相应属性即可。
```

* 在宿主机安全组和防火墙中放行如下端口：

　　80（NGINX_HTTP_HOST_PORT）
  
　　443（NGINX_HTTPS_HOST_PORT）
  
　　3306（MYSQL_HOST_PORT）
  
　　6379（REDIS_HOST_PORT）
  
　　9501（PHP_HYPERF_PORT）

```
注意：

　　1、如果宿主机的相应端口已被占用，请修改为未占用端口；
    
　　2、如果http协议默认的80端口、https协议默认的443端口修改为了其他端口，访问时切记带上端口，比如：将80端口改为了8080端口，则访问时应为：http://dashboard.yourdomain.com:8080　。
```


## 二、配置

### 2.1 docker-compose 配置

- `cd /path/to/docker-compose`
- `cp .env.example .env`  #根据自己的情况，修改相应配置，详见下面的提示

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

# mochat超级管理员手机号，可修改你自己的手机号
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

- `cp docker-compose.sample.yml docker-compose.yml`  #默认无需修改


## 三、运行
```
cd /path/to/docker-compose
docker-compose build
docker-compose up
```
提示：
- docker-compose up执行时，dashboard、sidebar、operation、mochat_init容器运行完后会自动退出显示

```
dashboard exited with code 0
sidebar exited with code 0
operation exited with code 0
mochat_init exited with code 0
```
这属于正常现象，不属于error

- 运行成功
运行时间根据不同主机配置、网络速度不同，可能会时间过长，请耐心等待
```
stone@localhost docker-compose % docker-compose ps            
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

- 如果初始化失败，可重新初始化
```
cd /path/to/docker-compose
rm ./services/mochat_init/install.lock
docker-compose up mochat_init
```

### 3.1 如果热更新
- 后端PHP热更新，可以 `./servers/php/Dockerfile` 内改 `php /opt/www/bin/hyperf.php start` 为 `php /opt/www/bin/hyperf.php server:watch`
- 前端热更新建议在宿主机 `npm run dev`，接口调试地址为 `http://api.mochat.com`

### 3.2 登陆
- 在浏览器输入 http://scrm.mochat.com
- 默认的用户名密码: `18888888888` / `123456`
- 进入项目，在`系统设置` -> `授权管理` 中点击 `添加企业微信号`
- 如果您没有企业微信号，您可以到企业微信官网网站注册调试用的`企业微信号`
