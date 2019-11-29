---
title:      "从0开始写一个简单的Express中间件"
date: 2018-05-014T01:37:56+08:00
lastmod: 2017-05-14T01:37:56+08:00
author:     "Adrian"
tags: ["node", "JavaScript"]
categories: ["Coding"]
draft: false

---

既然是从0开始，那咱就先`npm init`一个项目出来，取名为“express-midware-demo”，其他项按回车使用默认设置。

输入`npm install express --save`安装express，然后在package.json的“script”项中添加服务器启动脚本：

```json
  "scripts": {
​    "start": "node index.js",
     "test": "echo \"Error: no test specified\" && exit 1"
  }
```

创建index.js，引入并启用express，监听1024端口，生成一些数据，并实现一个简单的接收get请求并返回数据的接口。

```javascript
const express = require('express')
const app = express()

let persons = [{
        "id": 1,
        "name": "Jay"
    },
    {
        "id": 2,
        "name": "David"
    }
]

app.get("/api/persons", (request, response) => {
    response.json(persons)
})

const PORT = 1024
app.listen(PORT)
```

在命令行运行npm start，打开浏览器访问http://localhost:1024/api/persons，若一切正常页面显示persons对象的数据。

接下来引入“body-parser”，实现一个接收post请求并从请求的body中获取并添加数据的接口

```javascript
const express = require('express')
const app = express()
const bodyParser = require('body-parser')

app.use(bodyParser.json())

//省略...

app.post("/api/persons", (request, response) => {
    const person = request.body
    person.id = Math.floor(Math.random() * 100)
    persons = persons.concat(person)
    response.json(person)
})

const PORT = 1024
app.listen(PORT)
```

为求方便，将新数据的id设为一个随机数。重新运行npm start，使用postman等工具发送一个post请求：

![](/img/in-post/middleware/postman.png)

再用浏览器访问http://localhost/api/persons，可发现新的数据已经成功添加。

在实现第二个接口时，我们引入的“body-parser”就是一个Express自带的中间件，中间件实际上就是一个用于处理request和response对象的函数，上面的代码中使用body-parser从request对象中获取原始数据，并转化成一个JavaScript对象，存入request对象的body属性中，若不使用body-parser，`const person = request.body`语句中的body属性将为undefined。

接下来进入正题，我们自己来写一个中间件，其作用是记录服务器接收到的每个请求的信息。

```javascript
//...
app.use(bodyParser.json())

const requestLogger = (request, response, next) => {
    console.log('===========请求信息==========')
    console.log('请求种类: ', request.method)
    console.log('请求地址: ', request.path)
    console.log('body: ', request.body)
    console.log('============================')
    next()
}

app.use(requestLogger)
//...
```

重新启动服务器，浏览器访问http://localhost:1024/api/persons，可以看到控制台中打印出了请求的信息：

![](/img/in-post/middleware/console.png)

中间件函数有三个参数，前两个为请求和响应对象，第三个next是Express路由器中的一个功能函数，当它被调用时，将执行当前中间件之后的中间件。如果中间件函数没有结束“请求-响应”周期（如我们的中间件仅仅是记录请求信息），则必须调用next()以将控制传递给下个中间件函数，否则请求将被挂起。我们可以注释掉next()后重启服务器，新开一个浏览器窗口访问http://localhost:1024/api/persons，可以发现虽然请求信息在控制台中被打出，但浏览器并没有显示数据，即没有获取响应。

上面用到的bodyParser和requestLogger都在路由前调用，因为路由中需要用到bodyParser获取的数据，而我们想让requestLogger记录所有请求的信息，这两种情况下我们都需要中间件在进入路由前就被调用。下面我们写一个在路由之后使用的中间件，用来处理那些到达http://localhost:1024/api/persons路径之外的请求：

```javascript
//app.get(...)
//app.post(...)

const nonmatch = (request, response) => {
    response.status(404).send({
        error: '请求地址错误'
    })
}

app.use(nonmatch)

const PORT = 1024
app.listen(PORT)
```

由于在这个中间件中我们返回404，结束了“请求-响应”周期，所以不再需要用到next()。重启服务器并打开浏览器，输入http://localhost:1024/haha，可以看到中间件返回的错误信息。



