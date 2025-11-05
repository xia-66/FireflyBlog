---
title: "Koa2入门"
published: 2024-01-02
updated: 2024-04-12
description: ""
tags: ["Koa2"]
category: "后端相关"
draft: false
---

##一、Koa2安装
创建一个空白目录，然后进入终端，并在终端对koa进行安装：

    # 项目初始化
    npm init -y
    
    # 安装koa2
    npm i koa2 -S

## 二、入口文件
在项目根目录创建 `app.js` 文件，并在上一步操作中生成的 `package.json` 里配置：

    {
      \"scripts\": {
        \"test\": \"echo \\\"Error: no test specified\\\" && exit 1\",
        \"start\": \"node app.js\"
      },
    }

在 `app.js` 中：

    const Koa = require(\'koa2\');
    const app = new Koa();
    const port = 9000;
    
    /* 
        解释下面这段代码：
        app.use()方法是：将给定的中间件方法添加到此应用程序。简单说就是调用中间件
        app.use() 返回 this, 因此可以链式表达
    */
    app.use(async (ctx)=>{
        ctx.body = \"Hello, Koa\";
          // ctx.body是ctx.response.body的简写
    })
    
    app.listen(port, ()=>{
        console.log(\'Server is running at http://localhost:\'+port);
    })

然后运行 `npm start` ，并在浏览器输入 `http://localhost:9000/` 即可看到页面效果。
## 三、洋葱模型
学Koa必须要了解 `洋葱模型` :

![2024-01-02T03:53:58.png][1]

`Koa` 和 `Express` 都会使用到中间件，Express的中间件是顺序执行，从第一个中间件执行到最后一个中间件，发出响应：

![2024-01-02T03:54:24.png][2]

Koa是从第一个中间件开始执行，遇到 `next` 进入下一个中间件，一直执行到最后一个中间件，在逆序，执行上一个中间件 `next` 之后的代码，一直到第一个中间件执行结束才发出响应。

![2024-01-02T04:08:59.png][3]

对于这个洋葱模型，我们用代码来解释一下。假如把上面的代码改写成：

    const Koa = require(\'koa2\');
    const app = new Koa();
    const port = 9000;
    
    app.use(async (ctx, next)=>{
        console.log(1)
        await next();
        console.log(1)
    })
    
    app.use(async (ctx, next)=>{
        console.log(2)
        await next();
        console.log(2)
    })
    
    app.use(async (ctx)=>{
        console.log(3)
    })
    
    app.listen(port, ()=>{
        console.log(\'Server is running at http://localhost:\'+port);
    })

那么在浏览器刷新后，控制台得到的顺序是：

    1
    2
    3
    2
    1

现在可以看到，我们通过 `next`可以先运行下个中间件，等中间件结束后，再继续运行当前 `next()` 之后的代码。
## 四、路由安装
当需要匹配不同路由时，可以安装：

    npm i koa-router

将 `app.js` 修改：

    const Koa = require(\'koa2\');
    const Router = require(\'koa-router\');
    const app = new Koa();
    const router = new Router();
    const port = 9000;
    
    router.get(\'/\', async (ctx)=>{
        ctx.body = \"首页\";
    })
    
    router.get(\'/list\', async (ctx)=>{
        ctx.body = \"列表页\";
    })
    
    app.use(router.routes(), router.allowedMethods());
    
    app.listen(port, ()=>{
        console.log(\'Server is running at http://localhost:\'+port);
    })

此时，到浏览器刷新并在地址栏最后添加 `/list` 即可得到首页和列表页。
备注：

    // 调用router.routes()来组装匹配好的路由，返回一个合并好的中间件
    // 调用router.allowedMethods()获得一个中间件，当发送了不符合的请求时，会返回 `405 Method Not Allowed` 或 `501 Not Implemented`
    
    allowedMethods方法可以做以下配置：
    app.use(router.allowedMethods({ 
        // throw: true, // 抛出错误，代替设置响应头状态
        // notImplemented: () => \'不支持当前请求所需要的功能\',
        // methodNotAllowed: () => \'不支持的请求方式\'
    }))

## 五、路由拆分
有时候我们需要拆分路由，比如：
列表页下所有的子路由（即前端请求的api）与首页所有的子路由想分开处理，那么就需要拆分路由。
### 1、创建 `router` 文件夹
创建router文件夹，并在其中创建：`index.js` （路由总入口文件）、`home.js` （首页总路由文件）、`list.js` （列表页总路由文件）：

    # app.js
    const router = require(\'./router/index\')
    app.use(router.routes(), router.allowedMethods());
    
    # index.js
    const Router = require(\'koa-router\');
    const router = new Router();
    const home = require(\'./home\')
    const list = require(\'./list\')
    
    router.use(\'/home\', home.routes(), home.allowedMethods());
    router.use(\'/list\', list.routes(), list.allowedMethods());
    
    module.exports = router;
    
    # home.js
    const Router = require(\'koa-router\');
    const home = new Router();
    
    // 这里的 \'/\' 就是指向 index.js 中的 /home
    home.get(\'/\', async (ctx) => {
            ctx.body = \"首页\";
        })
    
    module.exports = home;
    
    
    # list.js
    const Router = require(\'koa-router\');
    const list = new Router();
    
    list.get(\'/\', async (ctx)=>{
        ctx.body = \"列表页\";
    })
    
    module.exports = list;

到浏览器刷新 `localhost:9000/home` 与 `localhost:9000/list` 即可得到首页与列表页。
### 2、路由重定向
那么有同学会问了，如果我想直接从 `localhost:9000` 重定向到 `localhost:9000/home` 该怎么办？
我们可以在 `router/index.js` 中做如下配置：

    router.use(\'/home\', home.routes(), home.allowedMethods());
    ...
    router.redirect(\'/\', \'/home\');

### 3、404无效路由
如果被访问到无效路由，那么我们可以统一返回404页面：
在 `router` 下 `errorPage.js` :

    const Router = require(\'koa-router\');
    const errorPage = new Router();
    
    errorPage.get(\'/\', async (ctx) => {
        ctx.body = \"访问页面不存在\";
    })
    
    module.exports = errorPage;

在 `app.js` 中引用：

    // 匹配不到页面的全部跳转去404
    app.use(async (ctx, next) => {
        await next();
        if (parseInt(ctx.status) === 404) {
            ctx.response.redirect(\"/404\")
        }
    })
    app.use(router.routes(), router.allowedMethods());

## 六、统一异常处理（可选）
作为后端开发，我们经常需要统一异常处理，避免每次都要自己手写404或200进行返回，因此我们可以创建 `utils/errorHandler.js` ：

    // 统一异常处理
    module.exports = (app) => {
        app.use(async (ctx, next) => {
            let status = 0;
            let fileName = \"\";
            try{
                await next();
                status = ctx.status;
            }catch(err){
                //console.log(err);
                status = 500;
            }
            if(status >= 400){
                switch(status){
                    case 400:
                    case 404:
                    case 500:
                        fileName = status;
                        break;
                    default:
                        fileName = \"other\";
                        break;
                }
            }
            ctx.response.status = status;
            console.log(fileName);
        });
    }

然后在 `app.js` 中引入：

    const errorHandler = require(\'./utils/errorHandler.js\');
    
    app.use(router.routes(), router.allowedMethods());
    ...
    errorHandler(app);

其实这一块不写关系也不大，但最好还是加上。
## 七、操作mysql函数封装
这里已经给大家直接封装好了一个库，专门用来操作mysql的。至于mysql的学习，将在独立的mysql教程中呈现。
首先，项目内安装 `mysql`：

    npm i mysql -S

现在，我们在 `utils` 目录下创建一个 `db.js` 文件：

    var mysql = require(\'mysql\')
    
    var pool = mysql.createPool({
        host: \'localhost\', // 连接的服务器(代码托管到线上后，需改为内网IP，而非外网)
        port: 3306, // mysql服务运行的端口
        database: \'xxx\', // 选择的库
        user: \'root\', // 用户名
        password: \'123456\' // 用户密码   
    })
    
    //对数据库进行增删改查操作的基础
    function query(sql,callback){
        pool.getConnection(function(err,connection){
            connection.query(sql, function (err,rows) {
                callback(err,rows)
                connection.release()
            })
        })
    }
    
    exports.query = query;

调用方式：
假设要访问首页(\'/home\')时，要查询表users中所有的数据，可以如下操作：

    const db = require(\'../utils/db.js\');
    
    home
        // 页面底部外链
        .get(\'/\', async (ctx) => {
            let data = await new Promise((resolve, reject)=>{
                let sqlLang = `select * from users`;
                db.query(sqlLang, (err, data)=>{
                    if(err) reject(err);
                    resolve(data);    // 返回拿到的数据
                })
            })
            ctx.body = data;
        })

## 八、后端允许跨域
前端想跨域，可以设置proxy。如果后端允许跨域，可以如下操作：

    // 安装koa2-cors
    npm i koa2-cors
    
    // 这里cors中间件一定要写在路由之前
    app.use(cors());
    app.use(router.routes(), router.allowedMethods())

## 九、读取静态资源文件
首先，在项目的根目录下创建 `assets` 后，将图片资源文件夹 `images` 放到其中，并且执行以下操作：

    // 安装koa-static
    npm install koa-static
    
    // 引入
    const path = require(\'path\')
    const static = require(\'koa-static\')
    
    // 获取静态资源文件夹
    app.use(static(path.join(__dirname+\'/assets\')));
    ...
    app.use(router.routes(), router.allowedMethods())

假设其中有一张图片叫做 `banner1.png`，那么我们打开浏览器，访问：`http://localhost:5050/images/banner1.png` 即可得到图片。这里注意：
路径上不需要写assets，因为我们已经指定了访问资源时， http://localhost:5050 自动指向 assets 文件夹。

由此，我们知道数据库中图片的地址只需要填写 `/images/banner1.png` 即可。
## 十、mysql录入数据

## 十一、POST请求
我们以登录举例讲post请求。
这里规定：前端发送 账号+密码 到后端，如果账号不存在于数据库，则注册账号。
如果账号存在于数据库中，则验证密码。
验证密码通过或注册账号成功，都返回token给前端。
### 1、建表
设定字段为account和pwd

    create table users (
        id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
        account VARCHAR(20) NOT NULL COMMENT \'账号\',
        pwd VARCHAR(20) NOT NULL COMMENT \'密码\',
      token LONGTEXT NOT NULL COMMENT \'令牌\'
    );

### 2、form表单页面

自己写个前端表单页面

在浏览器直接访问 `http://localhost:5050/index.html` 即可进入表单页。
### 3、安装中间件
安装 `koa-bodyparser` 与 `jsonwebtoken` 中间件:

    // koa-bodyparser用于获取post请求数据
    npm install koa-bodyparser --save
    
    // jsonwebtoken用于生成token
    npm install jsonwebtoken --save

#### * JWT
在用户登录的路由中使用 jwt.sign 来生成token，一共定义三个参数，第一个是存入token的信息，第二个是token的钥匙，和config/passport.js的配置的钥匙相同，第三个是保存的时间，3600即一个小时，最后返回token，要在前面加Bearer:

    const jwt = require(\'jsonwebtoken\')
    
    const token = jwt.sign({ myaccount: myaccount, mypwd: mypwd }, \'secret\', { expiresIn: 3600 })
    let obj = {
      token,
      msg: \'登录成功\'
    }
    resolve(obj)

### 4、添加post接口
在 `router/login.js` 中加入：

    const bodyParser = require(\'koa-bodyparser\')
    
    login.use(bodyParser());
    
    login.post(\'/register\', async (ctx)=>{
      console.log(ctx.request.body);        // 可以打印得到数据
        ctx.response.body = \"登录或注册\"
    })

### 5、登录与自动注册

    const Router = require(\'koa-router\')
    const login = new Router()
    const bodyParser = require(\'koa-bodyparser\')
    const db = require(\'../utils/db\')
    const jwt = require(\'jsonwebtoken\')
    
    login.get(\'/\', async (ctx) => {
        ctx.response.body = \"登录页面\"
    })
    
    login.use(bodyParser());
    
    login.post(\'/register\', async (ctx) => {
        let myaccount = ctx.request.body.account;
        let mypwd = ctx.request.body.pwd;
        let sql = `SELECT * FROM users WHERE account=\'${myaccount}\'`;
        let result = await new Promise((resolve, reject) => {
            return db.query(sql, (err, data) => {
                if (err) throw err;
                if (data.length > 0) {
                    resolve(data);
                } else {
                    resolve(false);
                }
            })
        })
        if (result) {
            // 能找到对应的账号
            if (result[0].pwd == mypwd) {
                // 账号密码正确，返回token
                ctx.body = {
                    token: result[0],
                    msg: \'登录成功\',
                    account: myaccount
                };
            } else {
                // 密码错误
                ctx.body = {
                    msg: \'密码错误\',
                    account: myaccount
                };
            }
        } else {
            // 找不到对应的账号，直接插入一个
            let result1 = await new Promise((resolve, reject) => {
                // 生成token
                const token = jwt.sign({ myaccount: myaccount, mypwd: mypwd }, \'secret\', { expiresIn: 3600 })
                return db.query(`INSERT INTO users (account, pwd, token) values (\'${myaccount}\', \'${mypwd}\', \'${token}\')`, (error, datas) => {
                    if (error) throw error;
                    // 已插入数据，返回用户名与token
                    let obj = {
                        token,
                        msg: \'登录成功\',
                        account: myaccount
                    }
                    resolve(obj)
                })
            })
            if (result1) {
                ctx.body = result1;
            }
        }
    })
    
    module.exports = login;

此时，前端做这个post请求后，就会得到相应的数据。

参考文档：https://chenshenhai.github.io/koa2-note/
  [1]: https://blog.heiyu.fun/usr/uploads/2024/01/743863323.png
  [2]: https://blog.heiyu.fun/usr/uploads/2024/01/4266463759.png
  [3]: https://blog.heiyu.fun/usr/uploads/2024/01/686229537.png