#### Docker安装及基本操作
* Docker (CE)小企业或个人,此处是CE
* Docker (EE)企业级

* 安装及启动
>   
    参考官方文档
    前置需要
        yum install -y yum-utils \
          device-mapper-persistent-data \
          lvm2
          
    配置url,不是必须,多一些镜像应该是.
        yum-config-manager \
            --add-repo \
            https://download.docker.com/linux/centos/docker-ce.repo
        开启若干其他测试库等 yum-config-manager--disable docker-ce-edge 这样可以禁用
        yum-config-manager --enable docker-ce-edge 
        yum-config-manager --enable docker-ce-test
        
    安装
        yum install docker-ce
    
    启动docker(配置文件 /etc/sysconfig/docker)
        systemctl start docker
    
    开机启动等
        systemctl enable docker
        systemctl disable docker
        chkconfig docker on
    
    测试(这个命令下载一个测试图像并在容器中运行。容器运行时，会打印一条信息消息并退出。)
        docker run hello-world
    
    卸载
        yum remove docker-ce
        rm -rf /var/lib/docker # 删除镜像容器等
>

* 常用命令
>   
    docker version # 查看docker的版本号，包括客户端、服务端、依赖的Go等
    
    xx表示不同的命令如，pull、run等。可以查看该命令的帮助，所有参数
    docker xx --help
    
    获取镜像 name：镜像名  [:tag]：版本，默认为最新的（也就是会自己加上一个参数:latest）
    docker pull [options] name[:tag]
    
    上面的需要翻墙，下面的是指定 下载地址 。网易蜂巢不错。
    docker pull hub.c.163.com/public/redis:2.8.4 
    
    查看本机的镜像  
    docker images [options] [repository[:tag]]
    
    运行    image：镜像名字;  command:命令;  arg:命令的参数
    docker run [options] image[:tag][command][arg...]
    docker run -d image 后台运行，并打印出id
    -p 8080:80  进行端口映射，将nginx的80端口映射到主机的8080端口上，也就是别人访问8080，可以访问到自己的80
    -P 同上，不过是80端口映射到随机端口。
    查看目前正在运行的容器
    docker ps
    
    查看所有容器
    docker ps -a
    
    启动一个运行(run)过的容器
    docker start <容器id>
    
    在运行的容器中执行命令 
    docker exec [options] container command [arg...]
    例如:   f：id中的字符
    docker exec -it f bash
    可以进入一个容器，和虚拟机中一样。相当于一个虚拟的linux。也就是容器内部
    
    停止运行容器 如果只有一个，f就是任意字符
    docker stop f
    
    删除容器
    docker rm <容器id> 
    
    制作镜像
    以下就是 打包镜像tomcat和jpress.war
    在某个目录创建文件 Dockerfile 使用vim，编辑输入如下内容：
        from images(镜像名)   （继承自哪个镜像）
        MAINTAINER ZX 970389745@qq.com  (维护人员信息)
        COPY jpress.war  /usr/local/tomcat/webapps    (同一目录下要打包成镜像的文件,拷贝到tomcat的运行目录下)
    
    然后在目录下使用  
    即可创建镜像，注意， . 是当前目录的意思
    docker build .   
    下面这句 -t是创建镜像并命名，因为上面的镜像没有命名
    docker build -t jpress:latest .
    
    运行容器
    -d表示后台运行 -p表示设置端口映射， jpress是镜像名
    docker run -d -p  8888:8080 jpress
    
    删除镜像
    docker rmi xx(名字或id)
    
    运行mysql镜像
    docker run -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=xxx  images(镜像名)
>

* 使用阿里云的Docker镜像加速器
>
    管理控制台 -> 容器镜像服务 -> 镜像加速器 -> 获取到其分配的加速地址
    修改 /etc/docker/daemon.json 文件,增加如下,没有时创建
      {
        "registry-mirrors": ["加速地址"]
      }
    
>

* 共享宿主机目录给容器
>
    docker run -d --name=test -v /opt/test:/usr/databases docker-test 
    test是容器的名字，需唯一；
    -v表示创建一个数据卷并挂载到容器里，
    示例表示把宿主机的/opt/test目录挂载到容器的/usr/databases目录下；
    docker-test是镜像的名字
>

#### Jenkins 持续集成 
* 前置
    * JDK8
    * Docker
* 使用docker下载镜像,并启动运行Jenkins容器
>
    docker run \
          -u root \
          -d \
          -p 8080:8080 \
          -v jenkins-data:/var/jenkins_home \
          -v /var/run/docker.sock:/var/run/docker.sock \
          jenkinsci/blueocean
    
    官方文档中还有一个 --rm . 但是提示 -d 和 --rm 相互冲突
>
* 进入Jenkins容器执行命令
> docker exec -it <容器id> bash
* 查看容器输出的日志
> docker logs <容器id> [-f(滚动的)]
* 或者自行下载war包运行
> java -jar jenkins.war --httpPort=8080

* 日志中输出了一串密码 a355a069e1e5410ea5dc0dfa1b21eb50
* 访问http://106.14.7.29:8080,并输入密码
* 可选.在 系统管理 -> 全局工具配置 中 设置maven和jdk目录等

* 创建一个基于Jenkins镜像的整合maven/jdk的新镜像
>   
    下载maven 它自带了一个oepn_jdk
    wget http://download.oracle.com/otn-pub/java/jdk/8u152-b16/aa0333dd3019491ca4f6ddbe78cdb6d0/jdk-8u152-linux-x64.tar.gz
    wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.5.2/binaries/apache-maven-3.5.2-bin.tar.gz

    Dockerfile文件内容
         #继承自jenkins镜像
         FROM jenkinsci/blueocean:latest  
         #维护人员信息
         MAINTAINER ZX 970389745@qq.com  
         # 拷贝本地的jdk和maven解压包到镜像的/usr下
         ADD apache-maven-3.5.2-bin.tar.gz /usr
         # 下载并解压jdk和maven,使用RUN表示执行当前系统(应该是CentOS)命令
         RUN cd /usr \
         tar -zxvf  apache-maven-3.5.2-bin.tar.gz 
         # 设置环境变量
         ENV MAVEN_HOME /usr/apache-maven-3.5.2
         ENV PATH $MAVEN_HOME/bin:$PATH
         # 开放的端口,如果有多个,用 空格 分割
         EXPOSE 8080 
    
    构建镜像     
    docker build . -t <仓库/镜像名:tag>
    例如
    docker build . -t zzzxxx/jenkins-maven-jdk:1.0
    
    运行该镜像,端口映射可自行调整.
    docker run \
              -u root \
              -d \
              -p 8080:8080 \
              -p 80:80 \
              -v jenkins-data:/var/jenkins_home \
              -v /var/run/docker.sock:/var/run/docker.sock \
              zzzxxx/jenkins-maven-jdk:1.0
              
    bug:在RUN的最后一句contOS命令后多加了个\ ,导致后面的ENV没有有效指定.
    
    maven构建过慢,自行在setting中增加阿里云镜像
            <mirror>
              <id>nexus-aliyun</id>
              <mirrorOf>*</mirrorOf>
              <name>Nexus aliyun</name>
              <url>http://maven.aliyun.com/nexus/content/groups/public</url>
            </mirror>
>

* 在docker中jenkins的主目录在/var/jenkins_home文件夹中(可用echo $JENKINS_HOME查看).
* 其中的工作目录在./workspace中.

* jenkins中,我尝试很多次,都无法在打包完成后使用java -jar命令运行jar.(会将末尾的&省略,无法后台启动).
    * 可以在shell窗口中,增加如下(BUILD_ID=dontKillMe 是防止jenkins杀死我们的后台进程)
    >
        BUILD_ID=dontKillMe
        java -jar $JENKINS_HOME/workspace/maven测试/target/zx-test.jar &
    >
    
    * 或者使用sh脚本执行(这样可以不用手动停止上一个版本的正在运行的jar)
        * 写在jenkins要执行的shell窗口中的脚本
            先停止前一个版本的jar.然后再用最新的jar替换掉之前的jar. 然后运行最新的jar
        >
            #!/bin/bash 
            cd /usr
            sh stop.sh
            cp $JENKINS_HOME/workspace/maven测试/target/zx-test.jar /usr
            echo "开始启动"
            BUILD_ID=dontKillMe 
            java -jar /usr/zx-test.jar &
        >
        * stop.sh 停止前一个版本的jar(pid=ps -ef xxx这句的意思是,通过若干过滤找到对应jar的pid记录.$1表示输出后的记录的第一个参数)
        >
            echo "正在停止之前的jar"
            pid=`ps -ef | grep zx-test.jar | grep -v grep | awk '{print $1}'`
            if [ -n "$pid" ]
            then
               echo "kill -9 的pid:" $pid
               kill -9 $pid
            fi
        >

* jenkins + github配置,实现jenkins能在push到github后,自动进行构建
    * github个人页面 -> setting -> developer setting -> personal access tokens -> Generate new token
    >
        该token就是OAuth2协议中的access_token.第三方应用(jenkinds)可通过该令牌获取你允许它进行的一些权限.
        选择创建令牌.并勾选 repo 和 admin:repo_hook权限.并自行保存好生成的token.
        这两个权限主要就是访问你的仓库,并设置你的仓库的监听器(钩子,可以理解为监听器.监听你的push等)
    >
    * 在github上选择你要部署的项目 -> setting -> webHooks -> add webHooks
    >
        这个钩子,可以配置你要监听的事件.当该事件发生后,会请求你配置的那个url.
        此处在Payload URL处填写
            http://<你的jenkins的ip>:<端口号>:/github-webhook/
        然后使用默认的监听push事件即可.
    >
    * 在jenkins中安装GitHub Plugin插件 系统管理-->插件管理-->可选插件->搜索.安装即可
    * 系统管理->系统设置->GitHub->add Github Server
    >
        api url中输入 https://api.github.com
        然后选择增加 Credentials. 选择类型为Secret text.在secret中输入之前的token即可.
        然后测试连接.当返回	
            Credentials verified for user BrightStarry, rate limit: 4997
        表示成功
    >
    * 后续还要安装maven插件. 创建maven项目.选择项目中的pom.xml文件等...不再赘述.需要时自行百度.