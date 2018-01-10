#### Docker安装及基本操作
* Docker (CE)小企业或个人
* Docker (EE)企业级
>   
    参考官方文档
    前置
        yum install -y yum-utils \
          device-mapper-persistent-data \
          lvm2
          
    配置url
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
    
    开机启动
        systemctl enable docker
        systemctl disable docker
        chkconfig docker on
        禁用upstart
    
    测试(这个命令下载一个测试图像并在容器中运行。容器运行时，会打印一条信息消息并退出。)
        docker run hello-world
    
    卸载
        yum remove docker-ce
        rm -rf /var/lib/docker # 删除镜像容器等
    
    docker version # 查看docker的版本号，包括客户端、服务端、依赖的Go等
    
    xx表示不同的命令如，pull、run等。可以查看该命令的帮助，所有参数
    docker xx --help
    
    获取镜像 name：镜像名  [:tag]：版本，默认为最新的（也就是会自己加上一个参数:latest）
    docker pull [options] name[:tag]
    
    上面的需要翻墙，下面的是指定 下载地址 。网易蜂巢不错。
    docker pull hub.c.163.com/public/redis:2.8.4 
    
    查看本机的镜像  
    docker images [options] [repository[:tag]]
    
    运行 image：镜像名字 command:命令 arg:命令的参数
    docker run [options] image[:tag][command][arg...]
    docker run -d image 后台运行，并打印出id
    -p 8080:80  进行端口映射，将nginx的80端口映射到主机的8080端口上，也就是别人访问8080，可以访问到自己的80
    -P 同上，不过是80端口映射到随机端口。
    查看目前正在运行的容器
    docker ps
    
    查看所有容器
    docker ps -all
    
    启动一个运行(run)过的容器
    docker start <容器id>
    
    在运行的容器中执行命令 
    docker exec [options] container command [arg...]
    例如:   f：id中的字符
    docker exec -it f bash
    可以进入一个容器，和虚拟机中一样。相当于一个虚拟的linux。也就是容器内部
    
    停止运行容器 如果只有一个，f就是任意字符
    docker stop f
    
    
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

#### Jenkins 持续集成 
* 参考官方文档.

* 前置
    * JDK8
    * Docker
* 运行
* 使用docker下载镜像,并启动运行Jenkins容器
>
    docker run \
          -u root \
          -d \
          -p 8081:8080 \
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
* 访问http://106.14.7.29:8081,并输入密码