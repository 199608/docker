#Dockerfile

##定义：
Dockerfile是一个文本格式的配置文件，用户可以使用Dockerfile来快速创建自定义的镜像

##基本结构
Dockerfile由一行行命令语句组成，并且支持以#开头的注视行。

一般而言，Dockerfile分为四部分：
- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令

##指令说明

1. FROM  
指令所创建的基础镜像，如果本地不存在则默认回去Docker Hub下载指定镜像。  
格式为`FROM<image>`，或`FROM<image>:<tag>`，或`FROM<image>@<digest>`
*任何Dockerfile中的第一个指令必须为FORM指令。*
*如果在用一个Docker中创建过个镜像，可以使用多个FORM指令（每个镜像一次）*

2. MAINTAINER  
指定维护者信息，格式为`MAINTAINER<name>`  
例如：`MAINTAINER image_createor@docker.com`  
该信息会写入生成镜像的Author属性域中。

3. RUN  
运行指令的命令  
格式为`RUN<commond>`或`RUN ["executable", "param1", "param2"]`  
`RUN<command>`默认将在shell终端中运行命令，即/bash/sh -c  
`RUN ["executable", "param1", "param2"]`则使用exec执行，不会启动shell环境。
注意，该命令会被解析为Json数组，因此必须使用双引号。  
指定使用其他终端类型可以通过第二种方式实现，例如`RUN ["/bin/bash", "-c", "echo hello"]`  
*每条RUN指令将在当前镜像的基础上执行指令命令，并提交为新的镜像。当命令较长时可以使用\来换行*

4. CMD  
用来指定启动容器时默认执行的命令。  
每个Dockerfile只能有一条CMD命令。如果指定了多条命令，只有最后一条会被执行。  
如果用启动容器时手动指定了运行命令（作为run的参数），则会覆盖掉CMD指定的命令  
它支持三种格式：
    - `CMD ["executable", "param1", "param2"]`使用exec执行，是推荐使用的方式
    - `CMD command param1 param2`在/bin/sh中执行，提供给需要交互的应用
    - `CMD ["param1", "param2"]`提供给ENTRYPOINT的默认参数
    
5. LABEL  
用来指定生成镜像的元数据标签信息
格式为`LABEL <key>=<value> <key>=<value> <key>=<value> ...`  

6. EXPOSE  
声明镜像内服务所监听的端口。  
格式为`EXPOSE <port> [<port>...]`  
注意，该指令只是起到声明作用，并不会自动自动完成端口映射。  
在启动容器时需要使用-P，Docker主机会自动分配一个宿主机的临时端口转发到指定的端口；使用-p，则可以具体指定哪个宿主主机的本地端口会映射过来。

7. ENV  
指定环境变量，在镜像生成过程中会被后续RUN指令使用，在镜像启动的容器中也会存在。
格式为`ENV <key> <value>`或`ENV <key>=<value>...`
指令指定的环境变量在运行时可以被覆盖掉
  


