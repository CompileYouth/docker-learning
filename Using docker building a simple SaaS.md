# 利用 Docker 创建一个简单的 SaaS

这是一个简单的 Demo，意在使用 Docker 和 Node Web 创建一个 SaaS 应用。在这个 SaaS 应用中，我将会使用 Node 创建一个再简单不过的 Web 程序，然后将其放入到 Docker 容器中。然后假设有两位客户想要使用这个软件服务，所以需要为这两位客户分配相应的资源，从而让他们可以使用我的 Node 网页应用。

希望大家跟着步骤一起实践一下。

本文默认你已经安装好了 Docker。

## 创建一个 Node web 应用

这个不是文章的重点，简略说一下。

创建一个新的文件夹，使用 `npm init` 初始化，一路回车，得到一个 `package.json` 文件；`npm install --save express`，安装 express。

在根目录中创建 `server.js` 用来定义 Web 程序。

```javascript
// server.js
const express = require('express');
const app = express();

const COMPANY_NAME = process.env.COMPANY_NAME ? process.env.COMPANY_NAME : 'Stranger';

app.get('/', (req, res) => {
    res.send(`Hello, ${COMPANY_NAME}`);
});

app.listen(8080, () => {
    console.log(`The server is running at 8080...`);
});
```

然后在 `package.json` 中的 script 属性中添加 `"start": node server.js`。至此，`package.json` 文件里的内容为：

```json
{
    "name": "test",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "start": "node server.js",
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
        "express": "^4.14.0"
    }
}
```

现在，在项目根目录里启动命令行，输入 `npm start` 命令；在浏览器中输入 `localhost:8080`，如果能看到 "Hello, Stranger"，那么就说明这个用 Node 写的服务器已经可以使用了。

## 利用 Docker 创建应用镜像

在将我们的应用放入 Docker 容器之前，我们首先需要为我们的应用创建一个 Docker 镜像。

**创建 Dockerfile**

在项目的根目录下面创建一个名为 `Dockerfile` 的空文件，注意没有任何后缀。

下面就是 `Dockerfile` 内需要写的内容，每一行的命令都以注释的形式标注了出来。值得提一下的是，根据我的学习经验，在跟着教程做 Demo 时，中间不懂的不需要太纠结，“硬着头皮”做下去，做到最后，看到了结果之后就能对整个过程有了一个高屋建瓴的认识，那个时候，回过头来慢慢抠细节将变得很高效。好吧，其实我是想说，下面的 `Dockerfile` 文件中，你不一定看得懂每一行注释，但跟着做下去，回过头来看肯定有不一样的认识。

```Dockerfile
# 定义我们这个镜像文件的父镜像，因为我们的应用是个用 Node 写的 web 应用，所以这里的父镜像是 node，具体讲是 node 的长期维护的版本：boron
# 还需要注意的是，Dockerfile 中的所有命令都是大写的
FROM node:boron

# 为应用在镜像中创建一个工作目录
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# 将应用中的 package.json 拷到工作目录中，并且在工作目录中执行 "npm install" 命令
COPY package.json /usr/src/app/
RUN npm install

# 将应用中的代码等文件全部拷到工作目录中
COPY . /usr/src/app

# 因为应用监听的是 8080 端口，所以在镜像中也需要绑定 8080 端口
EXPOSE 8080

# 至此，我们还需要在镜像中启动应用，这里只需要调用 "npm start" 命令即可启动应用
CMD [ "npm", "start" ]
```

习惯上，我们还需要创建一个 `.dockerignore` 文件，主要为了防止本地的模块和调试信息被拷贝到 Docker 镜像中。

```.dockerignore
# .dockerignore
node_modules
npm-debug.log
```

**创建镜像**

”创建镜像“这个词说的有点高大上，其实说白了与前些年在小地摊刻录光盘差不多一回事，这么一对比，相信你对下面的过程应该会更好理解一些。

在项目根目录启动命令行工具，输入以下命令：

```bash
docker build -t compileyouth/node-app .
```

`-t` 的中的 "t" 的意思是 tag，当你刻录好一个光盘后，因为光盘长得都一样，所以你得用笔在光盘上标个记号。这里也是一样，为即将创建的镜像做个标记。习惯上这个标记是由用户名和应用名组成，中间用 "/" 隔开。

当创建完成后，输入 `docker images` 来查看本地有哪些镜像，看看你刚刚创建的 "compileyouth/node-app" 镜像在不在。

```
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
compileyouth/node-app   latest              fc0fd7c50222        9 hours ago         659.6 MB
node                    boron               06b984afb149        4 days ago          655.5 MB
```

大家看下 SIZE 部分，你会发现自己刚刚创建的镜像比 node 的镜像大那么一点。那是因为如果一个子镜像是基于那个父镜像的，那么这个子镜像最小也得是父镜像的大小。

**启动镜像**

在项目根目录启动命令行工具，输入以下命令来运行应用镜像：

```bash
docker run -p 8081:8080 -d compileyouth/node-app
```

`-d` 的含义是以分离模式（detached mode）来运行容器，从而使得容器可以在后台运行。大家可以试着把 `-d` 去掉再运行一遍，就会发现容器运行的进程在前台执行，从而阻止你继续下面的操作（如果玩过 Linux 的话，对这个应该很熟悉），在这种情况下，你只能关闭当前的命令行工具，然后再启动一个新的。

当容器已经启动了之后，输入 `docker ps` 来查看查看当前有哪些容器。

```
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS                    NAMES
d502c5d2515e        compileyouth/node-app   "npm start"         5 minutes ago       Up 5 minutes        0.0.0.0:8081->8080/tcp   amazing_carson
```

Docker 自动为容器创建了一个容器名字，一般是”形容词_名字”的形式。如果你想为容器指定名字的话，可以用 `--name` 来指定，这样命令可以改为：

```
docker run -p 8081:8080 -d --name myapp compileyouth/node-app
```

至此，你用 Node 完成的 Web 应用已经作为一个服务已经可以上线使用了。

**SaaS 模拟**

假设有两位客户 A、B，他们想要使用我们这个用 Node 写的 Web 服务，那么我们可以使用下面的命令来为他们创建相应的容器：

```
# 为 A 创建容器
docker run -p 8081:8080 -d --name a-app -e "COMPANY_NAME=A_COMPANY" compileyouth/node-app

# 为 B 创建容器
docker run -p 8082:8080 -d --name b-app -e "COMPANY_NAME=B_COMPANY" compileyouth/node-app
```

上面命令中的 `-e` 主要是用来指定环境变量的。执行 `docker ps` 可以得到：

```
CONTAINER ID        IMAGE                   COMMAND             CREATED             STATUS              PORTS                    NAMES
6170679cc043        compileyouth/node-app   "npm start"         4 seconds ago       Up 3 seconds        0.0.0.0:8082->8080/tcp   b-app
5464e6893423        compileyouth/node-app   "npm start"         23 seconds ago      Up 22 seconds       0.0.0.0:8081->8080/tcp   a-app
```

这时在浏览器中分别打开 `localhost:8081` 和 `localhost:8082`，你应该会分别看到 "Hello, A_COMPANY" 和 "Hello, B_COMPANY"。这个时候，已经为客户 A 和 B 提供了服务。

过了一段时间，发现客户 A 欠费了，那么你可以给 A 发送提醒邮件并且暂停对其的服务：

```
docker pause a-app
```

这时你再尝试打开 `localhost:8081` 时会发现网页处于 pending 状态（网页图标一直处于旋转状态）。

又过了一段时间，发现 A 仍然没有续费，那么你开始决定停止他的服务：

```
docker unpause a-app
docker stop a-app
```

这时如果你再尝试打开 `localhost:8081` 的话会发现网页已经彻底打不开了。
