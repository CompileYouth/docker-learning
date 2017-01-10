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

app.get('/', (req, res) => {
    res.send('Hello, Docker');
});

app.listen(8080, () => {
    console.log(`The server is running at 8080...`);
})
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

现在，在项目根目录里启动命令行，输入 `npm start` 命令；在浏览器中输入 `localhost:8080`，如果能看到 "Hello, Docker"，那么就说明这个用 Node 写的服务器已经可以使用了。

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
