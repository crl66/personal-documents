# 本地Dockerfile打包镜像上传阿里云容器服务最佳实践 

本文旨在寻找一种方便快捷的方法通过本地dockerfile上传到阿里云镜像服务并上线 如有优化可添加

环境：win10 + vscode + docker

## 一、docker安装

#### 1.1 下载安装

[官网](https://docs.docker.com/desktop/windows/install/)docker windows安装包 下载之后直接一步next安装即可

#### 1.2 问题解决

如果出现docker没有成功启动 输入命令出现以下报错 一般是linux内核版本太低 可以下载并安装Linux 内核更新包之后重启解决

```
docker ps
error during connect: This error may indicate that the docker daemon is not running.: 
Get http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/containers/json: open //./pipe/docker_engine: The system cannot find the file specified. 
```

##### 1.3 docker启动成功界面
![logo](/images/docker允许成功截图.png)

### 二、编写Dockerfile文件

##### 2.1 文件编写

在根目录下创建Dockerfile文件 一般的配置就是如下

```
FROM node:lts-alpine as build-stage  
WORKDIR /app 
COPY . /app
RUN npm install
RUN npm run build:prod --mode ${ENV}

FROM nginx:stable-alpine as production-stage
WORKDIR /usr/share/nginx/html
COPY --from=build-stage /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 三.本地制作镜像推送到阿里云服务（银行后台为例）

#### 3.1 环境配置

通过配置环境来决定打包命令

```
.env.development  //开发环境
.env.production // 生产环境
.env.test //测试环境
```

#### 3.2 阿里云容器登录

1.登录阿里云 查看命名空间 dev为开发和环境 pro为生产环境 在这两个空间下创建项目

2.找到容器地址 如：这里镜像版本号dev以v开头 v1.1 or v1.0,0都可 pro以p开头 p1.1

```
registry.cn-shenzhen.aliyuncs.com/meta-houselai-dev/bank.admin:[镜像版本号]
```

#### 3.3 本地制作

1.在项目package.json文件中修改scripts命令 添加 docker:run-dev docker:push-dev docker:push-pro三个命令

 docker:run-dev：本地制作docker用于本地跑

docker:push-dev：推送到测试环境（用的是.env.test的环境依赖）

 docker:push-pro：推送到生产环境

<font color='red'>注意：每次上传版本号都要+1 方便服务器更新镜像</font>

```
  "scripts": {
    "dev": "vue-cli-service serve",
    "build:prod": "vue-cli-service build",
    "build:stage": "vue-cli-service build --mode staging",
    "docker:run-dev": "docker build -t bank.admin . && docker ps ",
    "docker:push-dev": "docker build --build-arg REQUEST_ENV=test -t registry.cn-shenzhen.aliyuncs.com/meta-houselai-dev/bank.admin:v1.2 . && docker push registry.cn-shenzhen.aliyuncs.com/meta-houselai-dev/bank.admin:v1.2",
    "docker:push-pro": "docker build -t registry.cn-shenzhen.aliyuncs.com/meta-houselai-pro/bank.admin:p1.8 . && docker push registry.cn-shenzhen.aliyuncs.com/meta-houselai-pro/bank.admin:p1.8"
  },
```

#### 3.4 开始推送

<font color='red'>注意：第一次推送一定要先打开终端登录一下 登录一次之后后续就不需要了, 如：</font>

```
docker login --username=guohua@1226846912747238 registry.cn-shenzhen.aliyuncs.com
```

修改 package.json scripts命令的版本号

运行想要推送的命令。ide直接点击即可



### 四、服务器上推送docker

1、登录 sudo su  -> 在输入密码 houselai2020

2、查找目录 cd /home  直到找到镜像位置

3、ls 查看目录清单

4、进入linux 编辑版本  vim docker-compose.yml

① esc键 换到命令模式 shift+冒号

② i输入 q 退出 w保存 q! 强制退出

5、检查版本是否编辑成功  cat docker-compose.yml

6、部署 docker-compose up -d