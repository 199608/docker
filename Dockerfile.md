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
任何Dockerfile中的第一个指令必须为FORM指令。  
如果在用一个Docker中创建过个镜像，可以使用多个FORM指令（每个镜像一次）

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
每条RUN指令将在当前镜像的基础上执行指令命令，并提交为新的镜像。当命令较长时可以使用\来换行

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

8. ADD  
该命令将复制指定的<src>路径下的内容到容器中的<dest>路径下。  
格式为`ADD <src> <dest>`。  
`<src>`可以是Dockerfile所在目录的一个相对路径（文件或目录），也可以是一个URL，还可以是一个tar文件（如果为tar文件，会自动解压到<dest>路径下）。  
`<dest>`可以是镜像内的绝对路径，或者相对于工作目录（WORKDIR）的相对路径。  
路径支持正则格式 

9. COPY  
格式为`COPY <src> <dest>`  
复制本地主机的`<src>`（为Dockerfile所在目录的相对路径、文件或目录）下的内容到镜像中的<dest>下。目标路径不存在时，会自动创建。  
路径同样支持正则格式。  
当使用本地目录为源目录时，推荐使用COPY。

10. ENTRYPOINT  
指定镜像的默认入口命令，该入口命令会在启动容器时作为根命令执行，所有传入值作为该命令的参数。  
支持两种格式：
    - `ENTRYPOINT ["executable", "param1", "param2"]`（exec调用执行）  
    - `ENTRYPOINT command param1 param2`（shell中执行）
此时，CMD指令指定值将作为跟命令的参数。  
每个Dockerfile中只能有一个ENTRYPOINt，当指定多个时，只有最后一个有效。  
在运行时，可以被--entrypoint参数覆盖掉，如`docker run --entrypoint`

11. VOLUME  
创建一个数据卷挂载点。  
格式为`VOLUME ["/data"]`  
可以从本地主机或其他容器挂载数据卷，一般用来存放数据库和需要保存的数据等。

12. USER  
指定运行容器时的用户名或UID，后续的RUN等指令也会使用指定的用户身份。  
格式为`USER daemon`  
当服务不需要管理员权限时，可以通过该命令指定运行用户，并且可以在之前创建所需要要的用户。
例如：`RUN groupadd -人postgres && useradd -r -g postgres postgres`  
要获取临时管理员权限可以使用gosu或sudo。

13. WORKDIR  
为后续的RUN、CMD和ENTRYPOINT指令配置工作目录。
格式为`WORKDIR /path/to/workdir`  
可以使用多个WORKDIR指令，后续命令如果参数是相对路径，则会基于之前命令指定的路径。

14. ARG  
指定一些镜像内使用的参数（例如版本号信息等），这些参数在执行`docker build`命令时才以--build-arg<varname>=<value>格式传入。  
格式为`ARG<name>[=<default value>]`

15. ONBUILD  
配置当所创建的镜像作为其他镜像的基础镜像时，所执行的创建操作指令。  
格式为`ONBUILD [INSTRUCTION]`  
如果基于image-A创建新的镜像时，新的Dockerfile中使用FROM image-A指定基础镜像，会自动执行ONBUILD指令的内容。  
使用ONBUILD指令的镜像，推荐在标签中注明，等价于在后面添加两条指令。

16. STOPSIGNAL  
指定所创建镜像启动的容器接受退出的信号值。
例如`STOPSINGAL singnal`

17. HEALTHCHECK  
配置所启动容器如何进行健康检查（如何判断健康与否），自Docker1.12开始支持。
格式有两张：
    - `HEALTHCHECK [OPTIONS] CMD command`根据所执行命令返回值是否为0来判断。
    - `HEALTHCHECK NONE`禁止基础镜像中的健康检查。
OPTION支持：
    - `--interval=DURATION`（默认30s）：过多久检查一次
    - `--timeout=DURATION`（默认30s）：每次检查等待结果的超时
    -`--retries`（默认3）：如果失败了，重试几次才最终确定失败
    
18. SHELL  
制定其他命令使用shell时的偶人shell类型  
`SHELL ["executable", "paramters"]`  
默认值为`["/bin/sh", "-c"]`

##创建镜像
编写完成Dockerfile之后，可以通过`docker build`命令来创建镜像。  
基本格式`docker build [选项] 内容路径`  
该命令将读取指定路径下（包括子目录）的Dockerfile，并将该路径下的所有内容发送给Docker服务端，有服务端来创建镜像。  
因此除非生成镜像需要，否则一般建议防止Dockerfile的目录为空目录。
两点经验：
    - 如果要使用非内容路径下的Dockerfile，可以通过-f选项来制定其路劲。
    - 要制定生成镜像的标签信息，可以使用-t选项。
    
##使用.dockerignore文件
可以通过.dockerignore文件（每一行添加一条匹配模式）来让Docker忽略匹配模式路径下的目录和文件。

##建议
- **精简镜像用途**：尽量让每个镜像的用途比较集中、单一，避免构造大而复杂、多功能镜像；
- **选用合适的基础镜像**：过大的基础镜像会造成生成臃肿的镜像，一般推荐较为小巧的debian镜像；
- **提供足够清晰的命令注释和维护者信息**：Dockerfile也是一种代码，需要考虑方便后续扩展和他人使用；
- **正确使用版本号**：使用明确的版本号信息，如1.0，2.0，而非latest，将避免内容不一致可能引发的惨案；
- **减少镜像层数**：如果希望所生成的镜像的层数尽量少，则要尽量合并指令，例如多个RUN指令可以合并为一条；
- **及时删除临时文件和缓存文件**：特别是在执行了apt-get指令后，/var/cache/apt下面会缓存一些安装包；
- **提高生成速度**：如果合理使用缓存，减少内容目录下的文件，或使用.dockerignore文件指令等；
- **条例合理的指令顺序**：在开启缓存的情况下，内容不变的指令精良放在前面，这样可以尽量服用；
- **减少外部源的干扰**：如果确实要从外部引入数据，需要指定持久的地址，并带有版本信息，让他人可以重复而不出错；
