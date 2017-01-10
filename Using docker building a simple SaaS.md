# 利用 Docker 创建一个简单的 SaaS

这是一个简单的 Demo，意在使用 Docker 和 Node Web 创建一个 SaaS 应用。在这个 SaaS 应用中，我将会使用 Node 创建一个再简单不过的 Web 程序，然后将其放入到 Docker 容器中。然后假设有两位客户想要使用这个软件服务，所以需要为这两位客户分配相应的资源，从而让他们可以使用我的 Node 网页应用。

希望大家跟着步骤一起实践一下。

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
