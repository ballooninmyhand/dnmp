[TOC]

### 1. 环境要求

- `git`
- `docker`
- `docker-compose`



### 2. 包含的镜像

- nginx-1.16.0
- php7.3.9（包含composer，包含扩展 amqp、bcmath、Core、ctype、curl、date、dom、fileinfo、filter、ftp、gd、hash、iconv、imap、json、libxml、mbstring、memcached、mongodb、mysqlnd、openssl、pcre、PDO、pdo_mysql、pdo_sqlite、Phar、posix、readline、redis、Reflection、session、SimpleXML、SPL、sqlite3、standard、tokenizer、xml、xmlreader、xmlwriter、zip、zlib）




### 3. 操作步骤

- 克隆项目

> git clone https://github.com/ballooninmyhand/dnmp.git

- 进入目录

> cd dnmp

- 复制并修改配置文件，设置端口号和工作目录

> cp env.example .env

- 复制 docker-compose-product.yml 文件

> cp docker-compose-product.yml docker-compose.yml

- 使用 `docker-compose` 创建容器，首次运行请加上 --build 参数

> docker-compose up -d [--build]

- 停止并销毁容器

> docker-compose down

- 重启某个容器

> docker-compose restart 容器1 容器2



### 4. 安装PHP扩展

- 如需安装其他 `PHP` 扩展，请自行修改 Dockerfile 文件



### 5. 如何设置 cron 定时任务

- 推荐使用主机的 cron 实现定时任务
- 每分钟执行 `test.php` 脚本，`dnmp_php` 是容器名称，`test.php` 在工作目录 `/var/www/html` 下

```shell
*/1 * * * * /usr/bin/docker exec dnmp_php php /var/www/html/test.php
```



### 6. 如何在 php 代码中使用 curl

- 问题：本地开发两个项目 A 和 B，A 需要用到 yar 扩展调用 B 中的一个 rpc 方法，但是发现报错 `curl exec failed 'Couldn't connect to server'`

- 原因：项目A中不能解析设置的域名

- 解决方案：在 `docker-compose.yml` 中配置静态ip，并在 php 中设置 `extra_hosts`

  - 配置虚拟网卡driver和subnet：

  ```yaml
  networks:
    default:
      driver: bridge
      ipam:
        config:
        - subnet: 10.0.0.0/24
  ```

  - 设置nginx的静态ip

  ```yaml
  nginx:
      #其他配置...
      networks:
        default:
          ipv4_address: 10.0.0.10
  ```

  - 再在php中设置extra_hosts

  ```yaml
  php56:
      #其他配置...
      extra_hosts:
        - "project.com:10.0.0.10"
      networks:
        - default
  ```

  - 重启服务



### 7. 为什么 mac 上请求一个接口耗时超过 1s

- 问题：在 mac 上开发时，请求一个接口需要1800ms，而在linux上只需要300ms

- 原因：[osxfs](https://docs.docker.com/docker-for-mac/osxfs/) 文件系统效率太低，mac 和 container 的文件系统不一样，同步时需要做大量的格式转换。

- 解决方案：安装 bg-sync，基本原理就是使用 daemon 方式建立一个同步磁盘，然后在 docker 启动容器时挂载这个同步磁盘。具体实现方式可以参考文件 `docker-compose-mac.yml`。




### 8. 参考链接

- [https://github.com/yeszao/dnmp](https://github.com/yeszao/dnmp)

- [docker 中使用 cron 定时任务](https://www.awaimai.com/2615.html)
