1. 若依微服务项目部署流程

   用docker部署RuoYi-Cloud

   https://gitee.com/y_project/RuoYi-Cloud

   # 一、服务器说明

   服务器

   | 序号 | 域名             | ip                  | 作用                                                         |
   | ---- | ---------------- | ------------------- | ------------------------------------------------------------ |
   | 1    | ruoyi.config.com | 192.168.66.30(前端) | 提供redis,nacos,mysql ,nginx服务，前端代码也是部署到该服务器 |
   | 2    | ruoyi.server.com | 192.168.66.31(后端) | 服务部署的机器，supervisor 进行项目线程统一管理              |

   # 二、成品展示

   1.页面展示

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172038766.png)

   2.config机器前端代码部署路径
   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172036919.png)

   3.config机器中间件安装路径(只用到了redis，[nginx](https://so.csdn.net/so/search?q=nginx&spm=1001.2101.3001.7020)，nacos其余的请忽略,start.sh是统一启动的shell脚本)
   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172043280.png)
   4.后端代码部署路径

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172225686.png)

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172228275.png)
   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172229772.png)
   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172235723.png)
   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172237531.png)

   # 三、前期准备

   ## 3.1、修改ip和域名

   由于我是在虚拟机上部署的，并没有真正的域名所以需要修改本机的C:\Windows\System32\drivers\etc下的hosts文件

   **windows下：**

   ```
   C:\Windows\System32\drivers\etc
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171350638.png)

   添加

   ```
   #若依前后台服务器ip和域名
   192.168.66.30 ruoyi.config.com
   192.168.66.31 ruoyi.server.com
   ```

   2.两台服务器的/etc下的hosts文件也需要修改

   **linux下：**

   ```
   vim /etc/hosts
   ```

   ```
   192.168.66.30 ruoyi.config.com
   192.168.66.31 ruoyi.server.com
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171347097.png)

   ## 3.2、创建项目文件夹

   1、在  192.168.66.30服务器上

   ```sh
   #1、创建项目文件夹
   mkdir -p /home/projects/ruoyi-cloud/ruoyi-ui
   mkdir -p /root/docker-compose-service
   ```

   2、在  192.168.66.31服务器上

   **create-file.sh**

   ```sh
   #! usr/bin/bash
   
   #1、创建项目所需文件夹
   mkdir -p /home/projects/ruoyi-cloud/logs
   mkdir -p /home/projects/ruoyi-cloud/servers
   mkdir -p /home/projects/ruoyi-cloud/script
   mkdir -p /home/projects/ruoyi-cloud/upload_files
   mkdir -p /home/projects/ruoyi-cloud/script/supervisor/logs
   mkdir -p /home/projects/ruoyi-cloud/script/shell_script/logs
   
   #2、在servers下创建项目文件夹
   #auth、file、gateway、gen、job、monitor、system
   
   mkdir -p /home/projects/ruoyi-cloud/servers/auth/lib
   mkdir -p /home/projects/ruoyi-cloud/servers/file/lib
   mkdir -p /home/projects/ruoyi-cloud/servers/gateway/lib
   mkdir -p /home/projects/ruoyi-cloud/servers/gen/lib
   mkdir -p /home/projects/ruoyi-cloud/servers/job/lib
   mkdir -p /home/projects/ruoyi-cloud/servers/monitor/lib
   mkdir -p /home/projects/ruoyi-cloud/servers/system/lib
   #创建lib和config文件夹
   mkdir -p /home/projects/ruoyi-cloud/servers/auth/config
   mkdir -p /home/projects/ruoyi-cloud/servers/file/config
   mkdir -p /home/projects/ruoyi-cloud/servers/gateway/config
   mkdir -p /home/projects/ruoyi-cloud/servers/gen/config
   mkdir -p /home/projects/ruoyi-cloud/servers/job/config
   mkdir -p /home/projects/ruoyi-cloud/servers/monitor/config
   mkdir -p /home/projects/ruoyi-cloud/servers/system/config
   ```

   # 四、config机器前端部署

   ## 4.1、安装环境

   要求：在config机器上安装mysql、redis、[nacos](https://so.csdn.net/so/search?q=nacos&spm=1001.2101.3001.7020)，nginx

   ### nginx

   nginx.sh

   ```sh
   #! /usr/bin/bash
   
   #安装nginx镜像
   docker pull nginx
   
   # 1.删除 nginx 相应文件
   rm -rf /root/docker-compose-service/nginx/conf
   rm -rf /root/docker-compose-service/nginx/conf.d
   rm -rf /root/docker-compose-service/nginx/html
   rm -rf /root/docker-compose-service/nginx/logs
   
   # 2.运行 nginx 容器
   docker run -p 80:80 --name nginx -d nginx
   
   # 3.创建 nginx 相应文件
   mkdir -p /root/docker-compose-service/nginx/conf && chown -R 200 /root/docker-compose-service/nginx/conf
   
   mkdir -p /root/docker-compose-service/nginx/conf.d && chown -R 200 /root/docker-compose-service/nginx/conf.d
   
   mkdir -p /root/docker-compose-service/nginx/logs && chown -R 200 /root/docker-compose-service/nginx/logs
   
   # 4.把nginx容器中的文件复制出来
   docker cp -a nginx:/usr/share/nginx/html /root/docker-compose-service/nginx && chown -R 200 /root/docker-compose-service/nginx/html
   
   docker cp -a nginx:/etc/nginx/nginx.conf /root/docker-compose-service/nginx/conf
   
   docker cp -a nginx:/etc/nginx/conf.d/default.conf /root/docker-compose-service/nginx/conf.d
   
   docker cp -a nginx:/var/log/nginx /root/docker-compose-service/nginx/log
   
   # 5.删除 nginx 容器
   docker rm -f nginx
   
   # 6.创建并运行nginx容器
   docker-compose -f ./docker-compose.yaml up -d
   docker ps -a
   docker logs -f nginx
   ```

   docker-compose.yaml

   ```yaml
   version: "3"
   services:
   
     nginx:
       image: nginx
       container_name: nginx
       restart: always
       ports:
         - "80:80"
         - "8080:8080"
         - "443:443"
       volumes:
          - ./html:/usr/share/nginx/html
          - ./conf/nginx.conf:/etc/nginx/nginx.conf
          - ./conf.d:/etc/nginx/conf.d
          - ./logs:/var/log/nginx
   ```

   ```
   
   ```

   ```
   docker update nginx --restart=always
   ```

   ## 4.2、修改RuoYi-Vue

   ```
   # 克隆项目
   git clone https://gitee.com/y_project/RuoYi-Vue
   ```

   下载nodejs

   打开

   ```
   C:\Users\Administrator
   ```

   修改.npmrc文件

   ```ini
   registry=https://registry.npm.taobao.org/
   prefix=E:\java\03_Environment\07-Nodejs\nodejs_install\projects\ruoyi-cloud\ruoyi-ui\node-v12.10.0-win-x64\node_global
   cache=E:\java\03_Environment\07-Nodejs\nodejs_install\projects\ruoyi-cloud\ruoyi-ui\node-v12.10.0-win-x64\node_cache
   sass_binary_site=https://npm.taobao.org/mirrors/node-sass/
   chromedriver_cdnurl=https://npm.taobao.org/mirrors/chromedriver
   phantomjs_cdnurl=https://npm.taobao.org/mirrors/phantomjs
   electron_mirror=https://npm.taobao.org/mirrors/electron
   ```

   验证nodejs

   ```
   node -v
   
   v12.10.0
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152134896.png)

   设置nodejs缓存位置和淘宝加速，见其他文章

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152142657.png)

   修改前端的vue.config.js和index.js的配置
   配置修改：前端ui文件中的index.js文件、vue.config.js文件
   如下图：

   index.js文件

   1. 把history改为hash

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203161806869.png)

   vue.config.js文件

   1. 添加一个点
   2. 把target的地址改为前端服务器的ip或者域名，我的前端服务器的ip为

   ```
   192.168.66.30
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203161812376.png)

   3.修改后在前端执行

   **打开终端**

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152138083.png)

   **开发**

   ```bash
   # 进入项目目录
   cd ruoyi-ui
   
   # 安装依赖
   # 建议不要直接使用 cnpm 安装依赖，会有各种诡异的 bug。可以通过如下操作解决 npm 下载速度慢的问题
   npm install --registry=https://registry.npm.taobao.org
   
   # 启动服务
   npm run dev
   ```

   浏览器访问 http://localhost:80

   **发布**

   ```bash
   # 构建测试环境
   npm run build:stage
   
   # 构建生产环境
   npm run build:prod
   ```

    npm run build:prod 生成dist包文件，放到config服务器的/home/projects/ruoyi-cloud/ruoyi-ui目录下

   我的是放在nginx下的html目录下

   ```
   /root/docker-compose-service/nginx/html/dist
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171117090.png)

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172102970.png)

   ## 4.3、修改nginx配置

   ### default.conf

   ```
   vim /root/docker-compose-service/nginx/conf.d/default.conf
   ```

   ```ini
   server {
       listen       80;
       listen  [::]:80;
       server_name  localhost;
   
       #access_log  /var/log/nginx/host.access.log  main;
   
       location / {
           #1、前端项目dist放到nginx的html目录下所在位置
           root   /usr/share/nginx/html/dist;
           #root   /home/projects/ruoyi/ruoyi-ui/dist;
           try_files $uri $uri/ /index.html last;
           index  index.html index.htm;
       }
   
       #error_page  404              /404.html;
   
       # redirect server error pages to the static page /50x.html
       #
       error_page   500 502 503 504  /50x.html;
       location = /50x.html {
           root   /usr/share/nginx/html;
       }
   
   
       #2、后代理端地址
       location /prod-api/{
                   proxy_set_header Host $http_host;
                   proxy_set_header X-Real-IP $remote_addr;
                   proxy_set_header REMOTE-HOST $remote_addr;
                   proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                   proxy_pass http://192.168.66.31:8080/;
       }
   
   
   
       # proxy the PHP scripts to Apache listening on 127.0.0.1:80
       #
       #location ~ \.php$ {
       #    proxy_pass   http://127.0.0.1;
       #}
   
       # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
       #
       #location ~ \.php$ {
       #    root           html;
       #    fastcgi_pass   127.0.0.1:9000;
       #    fastcgi_index  index.php;
       #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
       #    include        fastcgi_params;
       #}
   
       # deny access to .htaccess files, if Apache's document root
       # concurs with nginx's one
       #
       #location ~ /\.ht {
       #    deny  all;
       #}
   }
   ```

   ### nginx.conf

   nginx.conf不用修改

   ```
   cat /root/docker-compose-service/nginx/conf/nginx.conf
   ```

   ```ini
   user  root;
   worker_processes  auto;
   
   error_log  /var/log/nginx/error.log notice;
   pid        /var/run/nginx.pid;
   
   
   events {
       worker_connections  1024;
   }
   
   
   http {
       include       /etc/nginx/mime.types;
       default_type  application/octet-stream;
   
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
   
       access_log  /var/log/nginx/access.log  main;
   
       sendfile        on;
       #tcp_nopush     on;
   
       keepalive_timeout  65;
   
       #gzip  on;
       
       include /etc/nginx/conf.d/*.conf;
   }
   ```

   ### 参考

   ```
   后端跨域配置如下：
   
   复制代码
   // nginx.conf
   server{
        listen 80; // 前台项目端口号
        server_name 140.143.133.253; // 前台项目ip或域名
        index index.html index.htm index.php;
        root  /www/wwwroot/lynn_z.com/dist; // 前台项目根目录
           
        location /api {
             proxy_pass http://140.143.133.253:4000; // 后端域名或IP
             rewrite ^/api/(.*)$ /$1 break; // 重写URL
             proxy_cookie_domain 140.143.133.253:4000 140.143.133.253:81; // 修改cookie中域名
             add_header Access-Control-Allow-Origin 140.143.133.253:80;
             add_header Access-Control-Allow-Credentials true;
   
        }
   }
   ```

   ```
   后端跨域配置如下：
   
   复制代码
   // nginx.conf
   server{
        listen 80; // 前台项目端口号
        server_name 192.168.66.30; // 前台项目ip或域名
        index index.html index.htm index.php;
        root  /usr/share/nginx/html/dist; // 前台项目根目录
           
        location /api {
             proxy_pass http://192.168.66.31:8080; // 后端域名或IP
             rewrite ^/api/(.*)$ /$1 break; // 重写URL
             proxy_cookie_domain 192.168.66.31:8080 192.168.66.31:81; // 修改cookie中域名
             add_header Access-Control-Allow-Origin 192.168.66.31:80;
             add_header Access-Control-Allow-Credentials true;
   
        }
   }
   ```

   5.启动nginx

   ```
   docker restart nginx
   docker ps
   
   #docker ps 后出现up状态表示成功，restarting状态表示上面的nginx配置文件有问题，重新修改nginx配置文件后重动nginx即可!
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171343369.png)

   浏览器访问192.168.66.30出现下图的效果则前端代码部署好了
   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172255839.png)

   9.修改nacos配置信息

   - 访问http://192.168.66.30:8848/

   # 五、server机器后端部署

   1.在server服务器上安装jdk1.8

   ## 5.1、jdk安装

   https://www.oracle.com/java/technologies/downloads/#java8

   <font color='orange'>从官网上下载linux版本的JDK</font>（<font color='cornflowerblue'>jdk-8u311-linux-x64.tar.gz</</font>）

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202112281532980.png)

   <font color='orange'>通过ftp或者rz命令上传到linux</font>

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202112281530240.png)

   我这里需要把JDK安装在/usr/local目录下，在/usr/local下新建文件加java

   ```css
   mkdir /usr/local/java
   ```

   tar命令解压JDK包，解压地址是   <font color='orange'>/usr/local/java</font>

   ```css
   tar -zxvf jdk-8u311-linux-x64.tar.gz 
   
   得到文件：jdk1.8.0_311 
   ```

   <font color='orange'>接下来配置环境变量</font>

   vim命令打开/etc/profile  

   ```css
   vim /etc/profile
   ```

   ### 添加下面内容

   ```csharp
   #Set Java_Environment
   export JAVA_HOME=/usr/local/java/jdk1.8.0_311
   export JRE_HOME=/usr/local/java/jdk1.8.0_311/jre
   export CLASSPATH=.:$JAVA_HOME/lib$:JRE_HOME/lib:$CLASSPATH
   export PATH=$JAVA_HOME/bin:$JRE_HOME/bin/$JAVA_HOME:$PATH
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202112281534527.png)

   <font color='orange'>加载环境变量</font>

   ```csharp
   source /etc/profile
   ```

   <font color='orange'>验证是否安装成功</font>

   ```csharp
   java -version
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202112281534060.png)

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202112281535999.png)

   <font color='cornflowerblue'>成功！</font>

   

   ## 5.2、nohup安装

   **解决：Linux下-bash: nohup: command not found或者bash: nohup: 未找到命令的问题**

   首先，没有发现nohup，先安装

   ```
   yum install coreutils
   ```

   其次，如果已经安装 ，查看本地是否有，查看nohup具体位置（**注意：nohup的字母不要打错，之前就是因为打成了“nohub”而浪费了很多时间的**）

   ```
   which nohup
   ```

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190806083920122.png)
   再次，将具体位置进行配置

   ```
   vim ~/.bash_profile 
   ```

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190806084138598.png)
   然后如下图所示，在[环境变量](https://so.csdn.net/so/search?q=环境变量&spm=1001.2101.3001.7020)PATH后面加上`：usr/bin`
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190806084118415.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwMjQxOTU3,size_16,color_FFFFFF,t_70)
   然后，保存，刷新刷新生效

   ```
   :wq
   
   source ~/.bash_profile
   ```

   最后，进行验证

   ```
   nohup --version
   ```

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190806084350463.png)

   

   

   ## 5.3、执行sql文件

   ### 1、创建ry-config数据库

   ```
   字符集:utf8
   utf8_bin
   ```

   **创建数据库**

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152120088.png)

   运行**ry_config_20220114.sql**文件

   ### 2、创建ry-cloud数据库

   ```
   ry-cloud
   
   字符集：utf8
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152102036.png)

   1. 运行ry_20210908.sql文件
   2. 运行quartz.sql(任务调度)文件

   ### 3、执行分布式事务sql

   选中mysql-master，重新new一个查询，在整个库执行sql

   运行ry_seata_20210128.sql文件

   ## 6.1、修改nacos中的信息

   1. 启动nacos
   2. 修改mysql、redis、nacos信息

   修改mysql、redis、nacos信息

   修改数据库连接的url和账号信息以及redis的ip地址

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152131952.png)

   **修改ruoyi-job-dev.yml**

   ```
    url: jdbc:mysql://192.168.66.30:3307/ry-cloud?useUnicode=true&characterEncoding=utf8&
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172249937.png)

   **修改ruoyi-system-dev.yml**

   ```
    url: jdbc:mysql://192.168.66.30:3307/ry-cloud?useUnicode=true&characterEncoding=utf8&
   ```

   每个服务依次检查

   修改ruoyi-gateway-dev.yml

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152151286.png)

   ```
   192.168.66.30:6379
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203152150624.png)

   ## 6.2、修改项目的配置信息

   1. 启动idea
   2. 修改mysql、redis、nacos信息

   以ruoyi-auth为例

   1、修改每个bootstrap.yml文件中的配置信息：mysql、redis、nacos

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172306875.png)

   2、修改logback.xml文件中的日志存放路径和文件编码格式

   ruoyi-auth

   ```
   <!-- 日志存放路径 -->
   <property name="log.path" value="/home/projects/ruoyi-cloud/logs/ruoyi-auth" />
   ```

   控制台输出添加：<charset>UTF-8</charset>

   ```
   <!-- 控制台输出 -->
   	<appender name="console" class="ch.qos.logback.core.ConsoleAppender">
   		<encoder>
   			<pattern>${log.pattern}</pattern>
               <charset>UTF-8</charset>
   		</encoder>
   	</appender>
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171716243.png)

   下面的几个模块的控制台输出同样都添加：<charset>UTF-8</charset>

   ruoyi-gateway

   ```
   <!-- 日志存放路径 -->
   <property name="log.path" value="/home/projects/ruoyi-cloud/logs/ruoyi-gateway" />
   ```

   ruoyi-modules

   ```
   <!-- 日志存放路径 -->
   	<property name="log.path" value="/home/projects/ruoyi-cloud/logs/ruoyi-file" />
   ```

   ruoyi-gen

   ```
   <!-- 日志存放路径 -->
   	<property name="log.path" value="/home/projects/ruoyi-cloud/logs/ruoyi-gen" />
   ```

   ruoyi-job

   ```
   <!-- 日志存放路径 -->
   	<property name="log.path" value="/home/projects/ruoyi-cloud/logs/ruoyi-job" />
   ```

   ruoyi-system

   ```
   <!-- 日志存放路径 -->
   	<property name="log.path" value="/home/projects/ruoyi-cloud/logs/ruoyi-system" />
   ```

   

   ## 7.1、本地运行项目

   修改项目mmaven配置

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203282351404.png)

   

   ## idea打包项目

   修改pom.xml文件，将build替换成下面的内容

   ```powershell
       <build>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <executions>
                       <execution>
                           <goals>
                               <goal>repackage</goal>
                           </goals>
                       </execution>
                   </executions>
               </plugin>
               <plugin>
                   <!--打包时去除第三方依赖-->
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <layout>ZIP</layout>
                       <includes>
                           <include>
                               <groupId>non-exists</groupId>
                               <artifactId>non-exists</artifactId>
                           </include>
                       </includes>
                   </configuration>
               </plugin>
   <!--            拷贝第三方依赖文件到指定目录-->
               <plugin>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-dependency-plugin</artifactId>
                   <executions>
                       <execution>
                           <id>copy-dependencies</id>
                           <phase>package</phase>
                           <goals>
                               <goal>copy-dependencies</goal>
                           </goals>
                           <configuration>
                               <!--target/lib是依赖jar包的输出目录，根据自己喜好配置-->
                               <outputDirectory>target/lib</outputDirectory>
                               <excludeTransitive>false</excludeTransitive>
                               <stripVersion>false</stripVersion>
                               <includeScope>runtime</includeScope>
                           </configuration>
                       </execution>
                   </executions>
               </plugin>
           </plugins>
       </build>
   ```

   其余的服务也是一样，依次检查修改

   1、

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171655231.png)

   

   2、

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172309316.png)

   4.按照成品展示的第四步将文件夹创建好后开始上传jar包和依赖包，以及配置文件

   配置文件我上传了这些

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172320435.png)

   5.shell脚本管理jar包开启关闭(不推荐,shell脚本学习的话可以试一试)，test服务是我自己创建的一个服务，请忽略

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172316679.png)

   

   ### 注意

   只提交左边这些目录下的jar和lib包

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171824591.png)

   **以auth为例**

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171804328.png)

   **file、gen、job、system的jar和lib都在ruoyi-modules包下**

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171806878.png)

   **monitor在\ruoyi-visual\ruoyi-monitor模块下**

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171821710.png)

   ## 我的脚本

   ### servers.sh

   ```sh
   #! /usr/bin/bash
   
   CURRENT_PATH="/home/projects/ruoyi-cloud/servers/"
   LOG_PATH="/home/projects/ruoyi-cloud/script/shell_script/logs/"
   
   SERVER_NAMES=("auth" "gateway" "file" "gen" "job" "system" "monitor")
   JAR=""
   PID=""
   
   #根据不同的服务定义项目路径和日志路径
   if [[ "${SERVER_NAMES[@]}"  =~ "${2}" ]]; then
       CURRENT_PATH=${CURRENT_PATH}${2}/
       LOG_PATH=${LOG_PATH}${2}.log
       JAR=$(find $CURRENT_PATH -maxdepth 1 -name "*.jar")
       PID=$(ps -ef | grep $JAR | grep -v grep | awk '{ print $2 }')
   elif [[ "${2}" != "all" ]]; then
       echo -e "Usage: ./servers.sh ${1}  {auth|gateway|file|gen|job|system|test|monitor|all} \n" >&2
       exit 1
   fi
   
   #服务操作的通用方法
   function call_servers(){
       for server_name in ${SERVER_NAMES[@]}
       do
            $1 $2 ${server_name}
       done
   }
   
   case "${1}" in
       "start")
           if [ "${2}" == "all" ];then
              call_servers ${0} ${1}
           elif [ ! -z "$PID" ]; then
               echo  "$JAR 已经启动，进程号: $PID "
           else
               echo -e "启动 $JAR ... \n"
               cd $CURRENT_PATH
           nohup java -Dloader.path=$CURRENT_PATH,resources,lib -Dfile.encoding=UTF-8 -jar $JAR >$LOG_PATH 2>&1 &
               if [ "$?"="0" ]; then
                   echo -e "$JAR 启动完成，请查看日志确保成功 \n"
               else
                   echo -e "$JAR 启动失败 \n"
               fi
           fi
           ;;
       "stop")
           if [ "${2}" == "all" ];then
              call_servers ${0} ${1}
           elif [ -z "$PID" ]; then
               echo -e "$JAR 没有在运行，无需关闭 \n"
           else
               echo -e "关闭 $JAR ..."
                 kill -9 $PID
               if [ "$?"="0" ]; then
                   echo -e "服务已关闭 \n"
               else
                   echo -e "服务关闭失败 \n"
               fi
           fi
           ;;
       "restart")
           if [ "${2}" == "all" ];then
              call_servers ${0} ${1}
           else
               ${0} stop ${2}
               ${0} start ${2}
           fi
           ;;
       "kill")
           if [ "${2}" == "all" ];then
              call_servers ${0} ${1}
           else
               echo -e "强制关闭 $JAR \n"
               killall $JAR
               if [ "$?"="0" ]; then
                   echo -e "成功 \n"
               else
                   echo -e "失败 \n"
               fi
           fi
           ;;
       "status")
           if [ "${2}" == "all" ];then
              call_servers ${0} ${1}
           elif [ ! -z "$PID" ]; then
               echo -e "$JAR 正在运行 \n"
           else
               echo -e "$JAR 未在运行 \n"
           fi
           ;;
     *)
       echo -e "Usage: ./springboot.sh {start|stop|restart|status|kill} ${2} \n" >&2
           exit 1
   esac
   ```

   

   6.脚本效果展示

   ```powershell
   [root@xuniji shell_script]# ./servers.sh status auth
   /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar 未在运行
   
   [root@xuniji shell_script]# ./servers.sh status all
   /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/gateway/ruoyi-gateway-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/file/ruoyi-modules-file-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/gen/ruoyi-modules-gen-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/job/ruoyi-modules-job-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/system/ruoyi-modules-system-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/test/ruoyi-modules-test-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/monitor/ruoyi-visual-monitor-2.3.0.jar 未在运行
   
   [root@xuniji shell_script]# ./servers.sh start auth
   启动 /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar ...
   
   /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar 启动完成，请查看日志确保成功
   
   [root@xuniji shell_script]# ./servers.sh stop auth
   关闭 /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar ...
   服务已关闭
   
   [root@xuniji shell_script]# ./servers.sh start all
   启动 /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar ...
   
   /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/gateway/ruoyi-gateway-2.3.0.jar ...
   
   /home/ruoyi_project/servers/gateway/ruoyi-gateway-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/file/ruoyi-modules-file-2.3.0.jar ...
   
   /home/ruoyi_project/servers/file/ruoyi-modules-file-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/gen/ruoyi-modules-gen-2.3.0.jar ...
   
   /home/ruoyi_project/servers/gen/ruoyi-modules-gen-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/job/ruoyi-modules-job-2.3.0.jar ...
   
   /home/ruoyi_project/servers/job/ruoyi-modules-job-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/system/ruoyi-modules-system-2.3.0.jar ...
   
   /home/ruoyi_project/servers/system/ruoyi-modules-system-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/test/ruoyi-modules-test-2.3.0.jar ...
   
   /home/ruoyi_project/servers/test/ruoyi-modules-test-2.3.0.jar 启动完成，请查看日志确保成功
   
   启动 /home/ruoyi_project/servers/monitor/ruoyi-visual-monitor-2.3.0.jar ...
   
   /home/ruoyi_project/servers/monitor/ruoyi-visual-monitor-2.3.0.jar 启动完成，请查看日志确保成功
   
   [root@xuniji shell_script]# ./servers.sh status all
   /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/gateway/ruoyi-gateway-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/file/ruoyi-modules-file-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/gen/ruoyi-modules-gen-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/job/ruoyi-modules-job-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/system/ruoyi-modules-system-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/test/ruoyi-modules-test-2.3.0.jar 正在运行
   
   /home/ruoyi_project/servers/monitor/ruoyi-visual-monitor-2.3.0.jar 正在运行
   
   [root@xuniji shell_script]# ./servers.sh stop all
   关闭 /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/gateway/ruoyi-gateway-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/file/ruoyi-modules-file-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/gen/ruoyi-modules-gen-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/job/ruoyi-modules-job-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/system/ruoyi-modules-system-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/test/ruoyi-modules-test-2.3.0.jar ...
   服务已关闭
   
   关闭 /home/ruoyi_project/servers/monitor/ruoyi-visual-monitor-2.3.0.jar ...
   服务已关闭
   
   [root@xuniji shell_script]# ./servers.sh status all
   /home/ruoyi_project/servers/auth/ruoyi-auth-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/gateway/ruoyi-gateway-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/file/ruoyi-modules-file-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/gen/ruoyi-modules-gen-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/job/ruoyi-modules-job-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/system/ruoyi-modules-system-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/test/ruoyi-modules-test-2.3.0.jar 未在运行
   
   /home/ruoyi_project/servers/monitor/ruoyi-visual-monitor-2.3.0.jar 未在运行
   ```

   7.通过[supervisor](https://so.csdn.net/so/search?q=supervisor&spm=1001.2101.3001.7020)来管理服务状态
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201224133056378.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Mjc5ODMz,size_16,color_FFFFFF,t_70)

   ## supervisord

   （1）**安装守护程序supervisord(推荐)**

   ```sh
   #服务安装前，建议更新Python3版本，使用较新的版本有利于服务拓展。若被管理的服务依赖于较新的Python版本，需要再次重新安装服务。
   yum install -y epel-release
   yum install -y supervisor
   #查看版本号
   supervisord -v
   #配置文件路径为/etc/supervisord.conf，其中用英文;表示注释。将配置文件备份，过滤注释配置后形成新的配置文件。
   # 备份配置文件
   cp /etc/supervisord.conf /etc/supervisord.conf.old
   mv /etc/supervisord.conf /etc/supervisord.example.conf
   # 保留非注释配置，初始化为新的配置文件
   cat /etc/supervisord.example.conf | grep -v '^;' | tr -s "\n" > /etc/supervisord.conf
   
   #使用命令echo_supervisord_conf查看默认配置
   echo_supervisord_conf
   
   #通过如下命令查看版本号
   supervisord -v
   ```

   （2）**修改配置文件**

   ```powershell
   vim /etc/supervisord.conf
   ```

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201224132832539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM4Mjc5ODMz,size_16,color_FFFFFF,t_70)

   ### 我的

   ```
   [include]
   files=//home/projects/ruoyi-cloud/script/supervisor/servers.conf
   ```

   ### supervisord.conf

   ```ini
   [unix_http_server]
   file=/var/run/supervisor/supervisor.sock   ; (the path to the socket file)
   [supervisord]
   logfile=/var/log/supervisor/supervisord.log  ; (main log file;default $CWD/supervisord.log)
   logfile_maxbytes=50MB       ; (max main logfile bytes b4 rotation;default 50MB)
   logfile_backups=10          ; (num of main logfile rotation backups;default 10)
   loglevel=info               ; (log level;default info; others: debug,warn,trace)
   pidfile=/var/run/supervisord.pid ; (supervisord pidfile;default supervisord.pid)
   nodaemon=false              ; (start in foreground if true;default false)
   minfds=1024                 ; (min. avail startup file descriptors;default 1024)
   minprocs=200                ; (min. avail process descriptors;default 200)
   [rpcinterface:supervisor]
   supervisor.rpcinterface_factory = supervisor.rpcinterface:make_main_rpcinterface
   [supervisorctl]
   serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket
   #[include]
   #files = supervisord.d/*.ini
   [include]
   files=//home/projects/ruoyi-cloud/script/supervisor/servers.conf
   ```

   (3)添加需要守护的程序到servers.conf

   ```
   mkdir -p /home/projects/ruoyi/script/supervisor/logs/
   ```

   ```
   vim /home/projects/ruoyi-cloud/script/supervisor/servers.conf
   ```

   ### servers.conf

   ```ini
   [program:auth]
   ;需要执行的命令
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/auth/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/auth/ruoyi-auth-3.4.0.jar
   ;命令执行的目录
   directory=/home/projects/ruoyi-cloud/servers/auth
   ;用户
   user=root
   stopsignal=INT
   ;是否自启动
   autostart=true
   ;是否自动重启
   autorestart=true
   ;自动重启时间间隔（s）
   startsecs=3
   ;错误日志文件
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/auth_err.log
   ;输出日志文件
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/auth_info.log
   
   [program:gateway]
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/gateway/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/gateway/ruoyi-gateway-3.4.0.jar
   directory=/home/projects/ruoyi-cloud/servers/gateway
   user=root
   stopsignal=INT
   autostart=true
   autorestart=true
   startsecs=3
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/gateway_err.log
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/gateway_info.log
   
   
   [program:file]
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/file/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/file/ruoyi-modules-file-3.4.0.jar
   directory=/home/projects/ruoyi-cloud/servers/file
   user=root
   stopsignal=INT
   autostart=true
   autorestart=true
   startsecs=3
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/file_err.log
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/file_info.log
   
   [program:gen]
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/gen/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/gen/ruoyi-modules-gen-3.4.0.jar
   directory=/home/projects/ruoyi-cloud/servers/gen
   user=root
   stopsignal=INT
   autostart=true
   autorestart=true
   startsecs=3
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/gen_err.log
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/gen_info.log
   
   
   [program:job]
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/job/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/job/ruoyi-modules-job-3.4.0.jar
   directory=/home/projects/ruoyi-cloud/servers/job
   user=root
   stopsignal=INT
   autostart=true
   autorestart=true
   startsecs=3
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/job_err.log
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/job_info.log
   
   
   [program:system]
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/system/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/system/ruoyi-modules-system-3.4.0.jar
   directory=/home/projects/ruoyi-cloud/servers/system
   user=root
   stopsignal=INT
   autostart=true
   autorestart=true
   startsecs=3
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/system_err.log
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/system_info.log
   
   [program:monitor]
   command=java -Dloader.path=java -Dloader.path=/home/projects/ruoyi-cloud/servers/monitor/lib,resources,lib -Dfile.encoding=UTF-8 -jar /home/projects/ruoyi-cloud/servers/monitor/ruoyi-visual-monitor-3.4.0.jar
   directory=/home/projects/ruoyi-cloud/servers/monitor
   user=root
   stopsignal=INT
   autostart=true
   autorestart=true
   startsecs=3
   stderr_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/monitor_err.log
   stdout_logfile=/home/projects/ruoyi-cloud/script/shell_script/logs/monitor_info.log
   ```

   

   (4)设置supervisor开机启动,进入目录/usr/lib/systemd/system/新增文件supervisord.service，内容如下：

   ```
   vim /usr/lib/systemd/system/supervisord.service
   ```

   ```powershell
   [Unit]
   Description=Process Monitoring and Control Daemon
   After=rc-local.service nss-user-lookup.target
   
   [Service]
   Type=forking
   ExecStart=/usr/bin/supervisord -c /etc/supervisord.conf
   ExecStop=/usr/bin/supervisorctl shutdown
   ExecReload=/usr/bin/supervisorctl reload
   KillMode=process
   Restart=on-failure
   RestartSec=42s
   
   [Install]
   WantedBy=multi-user.target
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203170020817.png)

   

   ## 启动supervisor

   ### 后台启动

   ```powershell
   #重新加载supervisor配置
   supervisorctl reload
   #设置supervisor开机自启动
   systemctl enable supervisord.service
   # 设置开机自启
   systemctl enable supervisord
   # 启动主服务
   systemctl start supervisord
   ```

   ### 前台启动

   在编写Docker镜像，需要在一个镜像中同时管理多个服务，需要使用`前台启动`。supervisord的默认启动方式是`daemon`，若要配置为前台启动需修改配置文件`/etc/supervisord.conf`中`nodaemon`属性值为`true`。

   ```shell
   # 使用脚本替换
   sed -i 's/nodaemon=false/nodaemon=true/g' /etc/supervisord.conf
   ```

   前台启动命令如下

   ```
   supervisord -c /etc/supervisord.conf
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172156245.png)

   报错

   ```
   Error: Another program is already listening on a port that one of our HTTP servers is configured to use.  Shut this program down first before starting supervisord.
   For help, use /usr/bin/supervisord -h
   ```

   解决

   ```
   ps -ef | grep supervisord
   
   结果：
   root        884      1  0 21:42 ?        00:00:00 /usr/bin/python /usr/bin/supervisord -c /etc/supervisord.conf
   root       2733   1290  0 21:48 pts/0    00:00:00 grep --color=auto supervisord
   ```

   杀掉这个进程

   ```
   kill -9 884
   ```

   或者直接

   ```
   kill `ps -ef |grep supervisord`
   ```

   重新启动supervisor

   ```
   supervisord -c /etc/supervisord.conf
   ```

   8.supervisor效果展示

   ```
   supervisorctl start all
   ```

   ```powershell
   [root@xuniji script]# supervisorctl status
   auth                             RUNNING   pid 33892, uptime 0:00:12
   file                             RUNNING   pid 33283, uptime 0:04:16
   gateway                          RUNNING   pid 33281, uptime 0:04:16
   gen                              RUNNING   pid 33292, uptime 0:04:16
   job                              RUNNING   pid 33282, uptime 0:04:16
   monitor                          RUNNING   pid 33278, uptime 0:04:16
   system                           RUNNING   pid 33279, uptime 0:04:16
   test                             RUNNING   pid 33284, uptime 0:04:16
   [root@xuniji script]# supervisorctl stop auth
   auth: stopped
   [root@xuniji script]# supervisorctl start auth
   auth: started
   [root@xuniji script]# supervisorctl stop all
   monitor: stopped
   auth: stopped
   gateway: stopped
   test: stopped
   gen: stopped
   job: stopped
   file: stopped
   system: stopped
   [root@k8s-test2 servers]# supervisorctl restart all
   auth: stopped
   gateway: stopped
   file: stopped
   job: stopped
   gen: stopped
   system: stopped
   monitor: stopped
   monitor: started
   system: started
   auth: started
   gateway: started
   job: started
   file: started
   gen: started
   ```

   **注意：重启后端服务后必须重启前端的nginx，否则会报错**

   ```
   docker restart nginx
   
   docker ps
   ```

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172204038.png)

   ### supervisorctl常用命令

   ```
   supervisorctl stop all 停止全部
   supervisorctl stop program_name # 停止某一个进程，program_name 为 [program:x] 里的x
   
   supervisorctl start program_name # 启动个进程
   supervisorctl restart program_name # 重启某个进程
   supervisorctl restart all # 重启所有进程
   supervisorctl update 修改配置文件后更新配置
   ```

   # 部署结果

   ## 本地运行结果

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203282343443.png)

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203282343750.png)

   ## docker部署结果

   **通过域名或者ip访问项目**

   ruoyi.config.com

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203172038766.png)

   192.168.66.30

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203171353998.png)

   # 问题

   ## IDEA创建Maven项目时无法解析插件

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203282327027.png)

   ## 解决

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203282327761.png)

   

   ![](https://gitee.com/xiebiao99/images/raw/master/img/202203282327445.png)
