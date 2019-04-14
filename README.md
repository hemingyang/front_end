# front_end
:a:

```linux
Angular 脚手架工具  

npm install -g @angular/cli  脚手架

ng  new my-app  使用angular官方模板 

ng  new  my_app 会出现找不到路不建议加 "_" 

ng  serve --open  启动项目 4200端口  

angular 新建项目时会出现python依赖问题 使用以下两个命令

npm install --global --production windows-build-tools

npm install -g node-gyp 


```



:v:

```linux
vue.js 安装加脚手架工具

npm  install vue -g  vue库

npm install -g @vue/cli   vue官方脚手架工具

vue -4058错误  原因网络代理问题换个npm源

vue create my-project 或 vue ui 创建项目

npm run serve 启动项目 8080端口

npm config set registry https://registry.npm.taobao.org

npm --registry https://registry.npm.taobao.org info underscore
```

:point_up:

```linux
react 安装

npm install -g create-react-app  react官方模板库

npm init create-react-app hello-react  创建项目

npm start  启动项目 3000端口
```

Ajax 💯

```javascript
原生Ajax

var xhr =new XMLHttpRequest();
xhr.open('GET','send-ajax.java');
//
xhr.onreadystatechnge =function(){
    var DONE =4;
    var OK =200;
    if(xhr.readState==DONE){
        if(xhr.status==OK){
            console.log(xhr.responseText);
        }else{
            console.log('Error:'+xhr.status);
        }
    }
}
xhr.send(null);

jQuery 

$.ajax({
    type:'GET',
    url:'send-ajax-data-java',
    dataType:'JSON',
    socess:function(data){
        console.log('Error:'+data);
    }
    
});


跨域解决方案
1、 通过jsonp跨域
2、 document.domain + iframe跨域
3、 location.hash + iframe
4、 window.name + iframe跨域
5、 postMessage跨域
6、 跨域资源共享（CORS）
7、 nginx代理跨域
8、 nodejs中间件代理跨域
9、 WebSocket协议跨域

一、 通过jsonp跨域
通常为了减轻web服务器的负载，我们把js、css，img等静态资源分离到另一台独立域名的服务器上，在html页面中再通过相应的标签从不同域名下加载静态资源，而被浏览器允许，基于此原理，我们可以通过动态创建script，再请求一个带参网址实现跨域通信。

1.）原生实现：

 <script>
    var script = document.createElement('script');
    script.type = 'text/javascript';

    // 传参并指定回调执行函数为onBack
    script.src = 'http://www.domain2.com:8080/login?user=admin&callback=onBack';
    document.head.appendChild(script);

    // 回调执行函数
    function onBack(res) {
        alert(JSON.stringify(res));
    }
 </script>
服务端返回如下（返回时即执行全局函数）：

onBack({"status": true, "user": "admin"})
2.）jquery ajax：

$.ajax({
    url: 'http://www.domain2.com:8080/login',
    type: 'get',
    dataType: 'jsonp',  // 请求方式为jsonp
    jsonpCallback: "onBack",    // 自定义回调函数名
    data: {}
});
3.）vue.js：

this.$http.jsonp('http://www.domain2.com:8080/login', {
    params: {},
    jsonp: 'onBack'
}).then((res) => {
    console.log(res); 
})
后端node.js代码示例：

var querystring = require('querystring');
var http = require('http');
var server = http.createServer();

server.on('request', function(req, res) {
    var params = qs.parse(req.url.split('?')[1]);
    var fn = params.callback;

    // jsonp返回设置
    res.writeHead(200, { 'Content-Type': 'text/javascript' });
    res.write(fn + '(' + JSON.stringify(params) + ')');

    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');
jsonp缺点：只能实现get一种请求。

二、 document.domain + iframe跨域
此方案仅限主域相同，子域不同的跨域应用场景。

实现原理：两个页面都通过js强制设置document.domain为基础主域，就实现了同域。

1.）父窗口：(http://www.domain.com/a.html)

<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>
<script>
    document.domain = 'domain.com';
    var user = 'admin';
</script>
2.）子窗口：(http://child.domain.com/b.html)

<script>
    document.domain = 'domain.com';
    // 获取父窗口中变量
    alert('get js data from parent ---> ' + window.parent.user);
</script>
三、 location.hash + iframe跨域
实现原理： a欲与b跨域相互通信，通过中间页c来实现。 三个页面，不同域之间利用iframe的location.hash传值，相同域之间直接js访问来通信。

具体实现：A域：a.html -> B域：b.html -> A域：c.html，a与b不同域只能通过hash值单向通信，b与c也不同域也只能单向通信，但c与a同域，所以c可通过parent.parent访问a页面所有对象。

1.）a.html：(http://www.domain1.com/a.html)

<iframe id="iframe" src="http://www.domain2.com/b.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');

    // 向b.html传hash值
    setTimeout(function() {
        iframe.src = iframe.src + '#user=admin';
    }, 1000);
    
    // 开放给同域c.html的回调方法
    function onCallback(res) {
        alert('data from c.html ---> ' + res);
    }
</script>
2.）b.html：(http://www.domain2.com/b.html)

<iframe id="iframe" src="http://www.domain1.com/c.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');

    // 监听a.html传来的hash值，再传给c.html
    window.onhashchange = function () {
        iframe.src = iframe.src + location.hash;
    };
</script>
3.）c.html：(http://www.domain1.com/c.html)

<script>
    // 监听b.html传来的hash值
    window.onhashchange = function () {
        // 再通过操作同域a.html的js回调，将结果传回
        window.parent.parent.onCallback('hello: ' + location.hash.replace('#user=', ''));
    };
</script>
四、 window.name + iframe跨域
window.name属性的独特之处：name值在不同的页面（甚至不同域名）加载后依旧存在，并且可以支持非常长的 name 值（2MB）。

1.）a.html：(http://www.domain1.com/a.html)

var proxy = function(url, callback) {
    var state = 0;
    var iframe = document.createElement('iframe');

    // 加载跨域页面
    iframe.src = url;

    // onload事件会触发2次，第1次加载跨域页，并留存数据于window.name
    iframe.onload = function() {
        if (state === 1) {
            // 第2次onload(同域proxy页)成功后，读取同域window.name中数据
            callback(iframe.contentWindow.name);
            destoryFrame();

        } else if (state === 0) {
            // 第1次onload(跨域页)成功后，切换到同域代理页面
            iframe.contentWindow.location = 'http://www.domain1.com/proxy.html';
            state = 1;
        }
    };

    document.body.appendChild(iframe);

    // 获取数据以后销毁这个iframe，释放内存；这也保证了安全（不被其他域frame js访问）
    function destoryFrame() {
        iframe.contentWindow.document.write('');
        iframe.contentWindow.close();
        document.body.removeChild(iframe);
    }
};

// 请求跨域b页面数据
proxy('http://www.domain2.com/b.html', function(data){
    alert(data);
});
2.）proxy.html：(http://www.domain1.com/proxy....
中间代理页，与a.html同域，内容为空即可。

3.）b.html：(http://www.domain2.com/b.html)

<script>
    window.name = 'This is domain2 data!';
</script>
总结：通过iframe的src属性由外域转向本地域，跨域数据即由iframe的window.name从外域传递到本地域。这个就巧妙地绕过了浏览器的跨域访问限制，但同时它又是安全操作。

五、 postMessage跨域
postMessage是HTML5 XMLHttpRequest Level 2中的API，且是为数不多可以跨域操作的window属性之一，它可用于解决以下方面的问题：
a.） 页面和其打开的新窗口的数据传递
b.） 多窗口之间消息传递
c.） 页面与嵌套的iframe消息传递
d.） 上面三个场景的跨域数据传递

用法：postMessage(data,origin)方法接受两个参数
data： html5规范支持任意基本类型或可复制的对象，但部分浏览器只支持字符串，所以传参时最好用JSON.stringify()序列化。
origin： 协议+主机+端口号，也可以设置为"*"，表示可以传递给任意窗口，如果要指定和当前窗口同源的话设置为"/"。

1.）a.html：(http://www.domain1.com/a.html)

<iframe id="iframe" src="http://www.domain2.com/b.html" style="display:none;"></iframe>
<script>       
    var iframe = document.getElementById('iframe');
    iframe.onload = function() {
        var data = {
            name: 'aym'
        };
        // 向domain2传送跨域数据
        iframe.contentWindow.postMessage(JSON.stringify(data), 'http://www.domain2.com');
    };

    // 接受domain2返回数据
    window.addEventListener('message', function(e) {
        alert('data from domain2 ---> ' + e.data);
    }, false);
</script>
2.）b.html：(http://www.domain2.com/b.html)

<script>
    // 接收domain1的数据
    window.addEventListener('message', function(e) {
        alert('data from domain1 ---> ' + e.data);

        var data = JSON.parse(e.data);
        if (data) {
            data.number = 16;

            // 处理后再发回domain1
            window.parent.postMessage(JSON.stringify(data), 'http://www.domain1.com');
        }
    }, false);
</script>
六、 跨域资源共享（CORS）
普通跨域请求：只服务端设置Access-Control-Allow-Origin即可，前端无须设置，若要带cookie请求：前后端都需要设置。

需注意的是：由于同源策略的限制，所读取的cookie为跨域请求接口所在域的cookie，而非当前页。如果想实现当前页cookie的写入，可参考下文：七、nginx反向代理中设置proxy_cookie_domain 和 八、NodeJs中间件代理中cookieDomainRewrite参数的设置。

目前，所有浏览器都支持该功能(IE8+：IE8/9需要使用XDomainRequest对象来支持CORS）)，CORS也已经成为主流的跨域解决方案。

1、 前端设置：
1.）原生ajax

// 前端设置是否带cookie
xhr.withCredentials = true;
示例代码：

var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容

// 前端设置是否带cookie
xhr.withCredentials = true;

xhr.open('post', 'http://www.domain2.com:8080/login', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('user=admin');

xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        alert(xhr.responseText);
    }
};
2.）jQuery ajax

$.ajax({
    ...
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
   crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    ...
});
3.）vue框架

a.) axios设置：

axios.defaults.withCredentials = true
b.) vue-resource设置：

Vue.http.options.credentials = true
2、 服务端设置：
若后端设置成功，前端浏览器控制台则不会出现跨域报错信息，反之，说明没设成功。

1.）Java后台：

/*
 * 导入包：import javax.servlet.http.HttpServletResponse;
 * 接口参数中定义：HttpServletResponse response
 */

// 允许跨域访问的域名：若有端口需写全（协议+域名+端口），若没有端口末尾不用加'/'
response.setHeader("Access-Control-Allow-Origin", "http://www.domain1.com"); 

// 允许前端带认证cookie：启用此项后，上面的域名不能为'*'，必须指定具体的域名，否则浏览器会提示
response.setHeader("Access-Control-Allow-Credentials", "true"); 

// 提示OPTIONS预检时，后端需要设置的两个常用自定义头
response.setHeader("Access-Control-Allow-Headers", "Content-Type,X-Requested-With");
2.）Nodejs后台示例：

var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var postData = '';

    // 数据块接收中
    req.addListener('data', function(chunk) {
        postData += chunk;
    });

    // 数据接收完毕
    req.addListener('end', function() {
        postData = qs.parse(postData);

        // 跨域后台设置
        res.writeHead(200, {
            'Access-Control-Allow-Credentials': 'true',     // 后端允许发送Cookie
            'Access-Control-Allow-Origin': 'http://www.domain1.com',    // 允许访问的域（协议+域名+端口）
            /* 
             * 此处设置的cookie还是domain2的而非domain1，因为后端也不能跨域写cookie(nginx反向代理可以实现)，
             * 但只要domain2中写入一次cookie认证，后面的跨域接口都能从domain2中获取cookie，从而实现所有的接口都能跨域访问
             */
            'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'  // HttpOnly的作用是让js无法读取cookie
        });

        res.write(JSON.stringify(postData));
        res.end();
    });
});

server.listen('8080');
console.log('Server is running at port 8080...');
七、 nginx代理跨域
1、 nginx配置解决iconfont跨域
浏览器跨域访问js、css、img等常规静态资源被同源策略许可，但iconfont字体文件(eot|otf|ttf|woff|svg)例外，此时可在nginx的静态资源服务器中加入以下配置。

location / {
  add_header Access-Control-Allow-Origin *;
}
2、 nginx反向代理接口跨域
跨域原理： 同源策略是浏览器的安全策略，不是HTTP协议的一部分。服务器端调用HTTP接口只是使用HTTP协议，不会执行JS脚本，不需要同源策略，也就不存在跨越问题。

实现思路：通过nginx配置一个代理服务器（域名与domain1相同，端口不同）做跳板机，反向代理访问domain2接口，并且可以顺便修改cookie中domain信息，方便当前域cookie写入，实现跨域登录。

nginx具体配置：

#proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;

    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}
1.) 前端代码示例：

var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问nginx中的代理服务器
xhr.open('get', 'http://www.domain1.com:81/?user=admin', true);
xhr.send();
2.) Nodejs后台示例：

var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var params = qs.parse(req.url.substring(2));

    // 向前台写cookie
    res.writeHead(200, {
        'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'   // HttpOnly:脚本无法读取
    });

    res.write(JSON.stringify(params));
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');
八、 Nodejs中间件代理跨域
node中间件实现跨域代理，原理大致与nginx相同，都是通过启一个代理服务器，实现数据的转发，也可以通过设置cookieDomainRewrite参数修改响应头中cookie中域名，实现当前域的cookie写入，方便接口登录认证。

1、 非vue框架的跨域（2次跨域）
利用node + express + http-proxy-middleware搭建一个proxy服务器。

1.）前端代码示例：

var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问http-proxy-middleware代理服务器
xhr.open('get', 'http://www.domain1.com:3000/login?user=admin', true);
xhr.send();
2.）中间件服务器：

var express = require('express');
var proxy = require('http-proxy-middleware');
var app = express();

app.use('/', proxy({
    // 代理跨域目标接口
    target: 'http://www.domain2.com:8080',
    changeOrigin: true,

    // 修改响应头信息，实现跨域并允许带cookie
    onProxyRes: function(proxyRes, req, res) {
        res.header('Access-Control-Allow-Origin', 'http://www.domain1.com');
        res.header('Access-Control-Allow-Credentials', 'true');
    },

    // 修改响应信息中的cookie域名
    cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
}));

app.listen(3000);
console.log('Proxy server is listen at port 3000...');
3.）Nodejs后台同（六：nginx）

2、 vue框架的跨域（1次跨域）
利用node + webpack + webpack-dev-server代理接口跨域。在开发环境下，由于vue渲染服务和接口代理服务都是webpack-dev-server同一个，所以页面与代理接口之间不再跨域，无须设置headers跨域信息了。

webpack.config.js部分配置：

module.exports = {
    entry: {},
    module: {},
    ...
    devServer: {
        historyApiFallback: true,
        proxy: [{
            context: '/login',
            target: 'http://www.domain2.com:8080',  // 代理跨域目标接口
            changeOrigin: true,
            secure: false,  // 当代理某些https服务报错时用
            cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
        }],
        noInfo: true
    }
}
九、 WebSocket协议跨域
WebSocket protocol是HTML5一种新的协议。它实现了浏览器与服务器全双工通信，同时允许跨域通讯，是server push技术的一种很好的实现。
原生WebSocket API使用起来不太方便，我们使用Socket.io，它很好地封装了webSocket接口，提供了更简单、灵活的接口，也对不支持webSocket的浏览器提供了向下兼容。

1.）前端代码：

<div>user input：<input type="text"></div>
<script src="./socket.io.js"></script>
<script>
var socket = io('http://www.domain2.com:8080');

// 连接成功处理
socket.on('connect', function() {
    // 监听服务端消息
    socket.on('message', function(msg) {
        console.log('data from server: ---> ' + msg); 
    });

    // 监听服务端关闭
    socket.on('disconnect', function() { 
        console.log('Server socket has closed.'); 
    });
});

document.getElementsByTagName('input')[0].onblur = function() {
    socket.send(this.value);
};
</script>
2.）Nodejs socket后台：

var http = require('http');
var socket = require('socket.io');

// 启http服务
var server = http.createServer(function(req, res) {
    res.writeHead(200, {
        'Content-type': 'text/html'
    });
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');

// 监听socket连接
socket.listen(server).on('connection', function(client) {
    // 接收信息
    client.on('message', function(msg) {
        client.send('hello：' + msg);
        console.log('data from client: ---> ' + msg);
    });

    // 断开处理
    client.on('disconnect', function() {
        console.log('Client socket has closed.'); 
    });
});
```





## 概览[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E6%A6%82%E8%A7%88)

JavaScript 是一种面向对象的动态语言，它包含类型、运算符、标准内置（ built-in）对象和方法。它的语法来源于 Java 和 C，所以这两种语言的许多语法特性同样适用于 JavaScript。需要注意的一个主要区别是 JavaScript 不支持类，类这一概念在 JavaScript 通过对象原型（object prototype）得到延续（有关 ES6 类的内容参考这里[`Classes`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)）。另一个主要区别是 JavaScript 中的函数也是对象，JavaScript 允许函数在包含可执行代码的同时，能像其他对象一样被传递。

先从任何编程语言都不可缺少的组成部分——“类型”开始。JavaScript 程序可以修改值（value），这些值都有各自的类型。JavaScript 中的类型包括：

- [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)（数字）
- [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)（字符串）
- [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)（布尔）
- [`Function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Function)（函数）
- [`Object`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)（对象）
- [`Symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol) (第六版新增)

…哦，还有看上去有些…奇怪的 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined)（未定义）类型和 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null)（空）类型。此外还有[`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array)（数组）类型，以及分别用于表示日期和正则表达式的 [`Date`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Date)（日期）和 [`RegExp`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/RegExp)（正则表达式），这三种类型都是特殊的对象。严格意义上说，Function（函数）也是一种特殊的对象。所以准确来说，JavaScript 中的类型应该包括这些：

- [`Number`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Number)（数字）

- [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)（字符串）

- [`Boolean`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Boolean)（布尔）

- [`Symbol`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)（符号）（第六版新增）

- `Object`

  （对象）

  - [`Function`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Function)（函数）
  - [`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array)（数组）
  - [`Date`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Date)（日期）
  - [`RegExp`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/RegExp)（正则表达式）

- [`Null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Null)（空）

- [`Undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Undefined)（未定义）

JavaScript 还有一种内置[`Error`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Error)（错误）类型，这个会在之后的介绍中提到；现在我们先讨论下上面这些类型。

## 数字[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E6%95%B0%E5%AD%97)

根据语言规范，JavaScript 采用“IEEE 754 标准定义的双精度64位格式”（"double-precision 64-bit format IEEE 754 values"）表示数字。据此我们能得到一个有趣的结论，和其他编程语言（如 C 和 Java）不同，JavaScript 不区分整数值和浮点数值，所有数字在 JavaScript 中均用浮点数值表示，所以在进行数字运算的时候要特别注意。看看下面的例子：

```js
0.1 + 0.2 = 0.30000000000000004
```

在具体实现时，整数值通常被视为32位整型变量，在个别实现（如某些浏览器）中也以32位整型变量的形式进行存储，直到它被用于执行某些32位整型不支持的操作，这是为了便于进行位操作。

JavaScript 支持标准的[算术运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators)，包括加法、减法、取模（或取余）等等。还有一个之前没有提及的内置对象 [`Math`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Math)（数学对象），用以处理更多的高级数学函数和常数：

```js
Math.sin(3.5);
var d = Math.PI * (r + r);
```

你可以使用内置函数 [`parseInt()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt) 将字符串转换为整型。该函数的第二个参数表示字符串所表示数字的基（进制）：

```js
parseInt("123", 10); // 123
parseInt("010", 10); //10
```

如果调用时没有提供第二个参数（字符串所表示数字的基），2013 年以前的 JavaScript 实现会返回一个意外的结果：

```js
parseInt("010");  //  8
parseInt("0x10"); // 16
```

这是因为字符串以数字 0 开头，[`parseInt()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)函数会把这样的字符串视作八进制数字；同理，0x开头的字符串则视为十六进制数字。

如果想把一个二进制数字字符串转换成整数值，只要把第二个参数设置为 2 就可以了：

```js
parseInt("11", 2); // 3
```

JavaScript 还有一个类似的内置函数 [`parseFloat()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseFloat)，用以解析浮点数字符串，与[`parseInt()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)不同的地方是，parseFloat()只应用于解析十进制数字。

单元运算符 + 也可以把数字字符串转换成数值：

```js
+ "42";   // 42
+ "010";  // 10
+ "0x10"; // 16
```

如果给定的字符串不存在数值形式，函数会返回一个特殊的值 [`NaN`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/NaN)（Not a Number 的缩写）：

```js
parseInt("hello", 10); // NaN
```

要小心NaN：如果把 `NaN` 作为参数进行任何数学运算，结果也会是 `NaN`：

```js
NaN + 5; //NaN
```

可以使用内置函数 [`isNaN()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/isNaN) 来判断一个变量是否为 `NaN`：

```js
isNaN(NaN); // true
```

JavaScript 还有两个特殊值：[`Infinity`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Infinity)（正无穷）和 `-Infinity`（负无穷）：

```js
1 / 0; //  Infinity
-1 / 0; // -Infinity
```

可以使用内置函数 [`isFinite()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/isFinite) 来判断一个变量是否是一个有穷数， 如果类型为`Infinity`, `-Infinity` 或 `NaN则返回false`：

```js
isFinite(1/0); // false 
isFinite(Infinity); // false 
isFinite(NaN); // false 
isFinite(-Infinity); // false 

isFinite(0); // true 
isFinite(2e64); // true 

isFinite("0"); // true,如果是纯数值类型的检测，则返回false：Number.isFinite("0");
```

**备注：** [`parseInt()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt) 和 [`parseFloat()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseFloat) 函数会尝试逐个解析字符串中的字符，直到遇上一个无法被解析成数字的字符，然后返回该字符前所有数字字符组成的数字。然而如果使用运算符 "+"， 只要字符串中含有无法被解析成数字的字符，该字符串都将被转换成 `NaN`。请你用这两种方法分别解析“10.2abc”这一字符串，比较得到的结果，理解这两种方法的区别。

## 字符串[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%AD%97%E7%AC%A6%E4%B8%B2)

JavaScript 中的字符串是一串[Unicode 字符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Values,_variables,_and_literals#Unicode.E7.BC.96.E7.A0.81)序列。这对于那些需要和多语种网页打交道的开发者来说是个好消息。更准确地说，它们是一串UTF-16编码单元的序列，每一个编码单元由一个 16 位二进制数表示。每一个Unicode字符由一个或两个编码单元来表示。

如果想表示一个单独的字符，只需使用长度为 1 的字符串。

通过访问字符串的  [`长度`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/length)（编码单元的个数）属性可以得到它的长度。

```js
"hello".length; // 5
```

这是我们第一次碰到 JavaScript 对象。我们有没有提过你可以像 [objects](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)  一样使用字符串？是的，字符串也有[methods](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String#Methods)（方法）能让你操作字符串和获取字符串的信息。

```js
"hello".charAt(0); // "h"
"hello, world".replace("hello", "goodbye"); // "goodbye, world"
"hello".toUpperCase(); // "HELLO"
```

## 其他类型[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%85%B6%E4%BB%96%E7%B1%BB%E5%9E%8B)

JavaScript 中 [`null`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null) 和 [`undefined`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined) 是不同的，前者表示一个空值（non-value），必须使用null关键字才能访问，后者是“undefined（未定义）”类型的对象，表示一个未初始化的值，也就是还没有被分配的值。我们之后再具体讨论变量，但有一点可以先简单说明一下，JavaScript 允许声明变量但不对其赋值，一个未被赋值的变量就是 `undefined` 类型。还有一点需要说明的是，`undefined` 实际上是一个不允许修改的常量。

JavaScript 包含布尔类型，这个类型的变量有两个可能的值，分别是 `true` 和 `false`（两者都是关键字）。根据具体需要，JavaScript 按照如下规则将变量转换成布尔类型：

1. `false`、`0`、空字符串(`""`)、`NaN`、`null` 和 `undefined` 被转换为 `false`
2. 所有其他值被转换为 `true`

也可以使用 `Boolean()` 函数进行显式转换：

```js
Boolean(""); // false
Boolean(234); // true
```

不过一般没必要这么做，因为 JavaScript 会在需要一个布尔变量时隐式完成这个转换操作（比如在 `if` 条件语句中）。所以，有时我们可以把转换成布尔值后的变量分别称为 真值（true values）——即值为 true  和 假值（false values）——即值为 false；也可以分别称为“真的”（truthy）和“假的”（falsy）。

JavaScript 支持包括 `&&`（逻辑与）、`||` （逻辑或）和`!`（逻辑非）在内的逻辑运算符。下面会有所提到。

## 变量[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%8F%98%E9%87%8F)

在 JavaScript 中声明一个新变量的方法是使用关键字 `let` 、`const` 和 [`var`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/var)：

`**let**` 语句声明一个块级作用域的本地变量，并且可选的将其初始化为一个值。

```js
let a;
let name = 'Simon';
```

下面是使用  `**let**` 声明变量作用域的例子：

```js
// myLetVariable is *not* visible out here

for (let myLetVariable = 0; myLetVariable < 5; myLetVariable++) {
  // myLetVariable is only visible in here
}

// myLetVariable is *not* visible out here
```

`**const**` 允许声明一个不可变的常量。这个常量在定义域内总是可见的。

 

```js
const Pi = 3.14; // 设置 Pi 的值  
Pi = 1; // 将会抛出一个错误因为你改变了一个常量的值。
```

`**var**` 是最常见的声明变量的关键字。它没有其他两个关键字的种种限制。这是因为它是传统上在 JavaScript 声明变量的唯一方法。使用 **var** 声明的变量在它所声明的整个函数都是可见的。

```js
var a;
var name = "simon";
```

一个使用  **var** 声明变量的语句块的例子：

```js
// myVarVariable *is* visible out here 

for (var myVarVariable = 0; myVarVariable < 5; myVarVariable++) { 
  // myVarVariable is visible to the whole function 
} 

// myVarVariable *is* visible out here
```

如果声明了一个变量却没有对其赋值，那么这个变量的类型就是 `undefined`。

JavaScript 与其他语言的（如 Java）的重要区别是在 JavaScript 中语句块（blocks）是没有作用域的，只有函数有作用域。因此如果在一个复合语句中（如 if 控制结构中）使用 var 声明一个变量，那么它的作用域是整个函数（复合语句在函数中）。 但是从 ECMAScript Edition 6 开始将有所不同的， `let` 和 `const` 关键字允许你创建块作用域的变量。

## 运算符[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E8%BF%90%E7%AE%97%E7%AC%A6)

JavaScript的算术操作符包括 `+`、`-`、`*`、`/` 和 `% ——`求余（[与模运算不同](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#%E6%B1%82%E4%BD%99_%28%29)）。赋值使用 `=` 运算符，此外还有一些复合运算符，如 `+=` 和 `-=`，它们等价于 `x = x *op* y`。

```js
x += 5; // 等价于 x = x + 5;
```

可以使用 `++` 和 `--` 分别实现变量的自增和自减。两者都可以作为前缀或后缀操作符使用。

[`+` 操作符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#.E5.8A.A0.E6.B3.95_(.2B))还可以用来连接字符串：

```js
"hello" + " world"; // hello world
```

如果你用一个字符串加上一个数字（或其他值），那么操作数都会被首先转换为字符串。如下所示：

```js
"3" + 4 + 5; // 345
3 + 4 + "5"; // 75
```

这里不难看出一个实用的技巧——通过与空字符串相加，可以将某个变量快速转换成字符串类型。

JavaScript 中的[比较操作](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators)使用 `<`、`>`、`<=` 和 `>=`，这些运算符对于数字和字符串都通用。相等的比较稍微复杂一些。由两个“`=`（等号）”组成的相等运算符有类型自适应的功能，具体例子如下：

```js
123 == "123" // true
1 == true; // true
```

如果在比较前不需要自动类型转换，应该使用由三个“`=`（等号）”组成的相等运算符：

```js
1 === true; //false
123 === "123"; // false
```

JavaScript 还支持 `!=` 和 `!==` 两种不等运算符，具体区别与两种相等运算符的区别类似。

JavaScript 还提供了 [位操作符](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators)。

## 控制结构[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E6%8E%A7%E5%88%B6%E7%BB%93%E6%9E%84)

JavaScript 的控制结构与其他类 C 语言类似。可以使用 `if` 和 `else` 来定义条件语句，还可以连起来使用：

```js
var name = "kittens";
if (name == "puppies") {
  name += "!";
} else if (name == "kittens") {
  name += "!!";
} else {
  name = "!" + name;
}
name == "kittens!!"; // true
```

JavaScript 支持 `while` 循环和 `do-while` 循环。前者适合常见的基本循环操作，如果需要循环体至少被执行一次则可以使用 `do-while`：

```js
while (true) {
  // 一个无限循环！
}

var input;
do {
  input = get_input();
} while (inputIsNotValid(input))
```

JavaScript 的 `for` 循环与 C 和 Java 中的相同，使用时可以在一行代码中提供控制信息。

```js
for (var i = 0; i < 5; i++) {
  // 将会执行五次
}
```

JavaScript 也还包括其他两种重要的 `**for**` 循环： [`for`...`of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)

```js
for (let value of array) {
  // do something with value
}
```

和 [`for`...`in`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) ：

```js
for (let property in object) {
  // do something with object property
}
```

`&&` 和 `||` 运算符使用短路逻辑（short-circuit logic），是否会执行第二个语句（操作数）取决于第一个操作数的结果。在需要访问某个对象的属性时，使用这个特性可以事先检测该对象是否为空：

```js
var name = o && o.getName();
```

或运算可以用来设置默认值：

```js
var name = otherName || "default";
```

类似地，JavaScript 也有一个用于条件表达式的三元操作符：

```js
var allowed = (age > 18) ? "yes" : "no";
```

在需要多重分支时可以使用  `基于一个数字或字符串的switch` 语句：

```js
switch(action) {
    case 'draw':
        drawIt();
        break;
    case 'eat':
        eatIt();
        break;
    default:
        doNothing();
}
```

如果你不使用 `break` 语句，JavaScript 解释器将会执行之后 `case` 中的代码。除非是为了调试，一般你并不需要这个特性，所以大多数时候不要忘了加上 `break。`

```js
switch(a) {
    case 1: // 继续向下
    case 2:
        eatIt();
        break;
    default:
        doNothing();
}
```

`default` 语句是可选的。`switch` 和 `case` 都可以使用需要运算才能得到结果的表达式；在 `switch` 的表达式和 `case` 的表达式是使用 `===` 严格相等运算符进行比较的：

```js
switch(1 + 3){
    case 2 + 2:
        yay();
        break;
    default:
        neverhappens();
}
```

## 对象[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%AF%B9%E8%B1%A1)

JavaScript 中的对象可以简单理解成“名称-值”对，不难联想 JavaScript 中的对象与下面这些概念类似：

- Python 中的字典
- Perl 和 Ruby 中的散列（哈希）
- C/C++ 中的散列表
- Java 中的 HashMap
- PHP 中的关联数组

这样的数据结构设计合理，能应付各类复杂需求，所以被各类编程语言广泛采用。正因为 JavaScript 中的一切（除了核心类型，core object）都是对象，所以 JavaScript 程序必然与大量的散列表查找操作有着千丝万缕的联系，而散列表擅长的正是高速查找。

“名称”部分是一个 JavaScript 字符串，“值”部分可以是任何 JavaScript 的数据类型——包括对象。这使用户可以根据具体需求，创建出相当复杂的数据结构。

有两种简单方法可以创建一个空对象：

```js
var obj = new Object();
```

和：

```js
var obj = {};
```

这两种方法在语义上是相同的。第二种更方便的方法叫作“对象字面量（object literal）”法。这种也是 JSON 格式的核心语法，一般我们优先选择第二种方法。

“对象字面量”也可以用来在对象实例中定义一个对象：

```js
var obj = {
    name: "Carrot",
    "for": "Max",//'for' 是保留字之一，使用'_for'代替
    details: {
        color: "orange",
        size: 12
    }
}
```

对象的属性可以通过链式（chain）表示方法进行访问：

```js
obj.details.color; // orange
obj["details"]["size"]; // 12
```

下面的例子创建了一个对象原型，`**Person**`，和这个原型的实例，**You**。

```
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// 定义一个对象
var You = new Person("You", 24); 
// 我们创建了一个新的 Person，名称是 "You" 
// ("You" 是第一个参数, 24 是第二个参数..)
```

完成创建后，对象属性可以通过如下两种方式进行赋值和访问：

```js
obj.name = "Simon"
var name = obj.name;
```

和：

```js
// bracket notation
obj['name'] = 'Simon';
var name = obj['name'];
// can use a variable to define a key
var user = prompt('what is your key?')
obj[user] = prompt('what is its value?')
```

这两种方法在语义上也是相同的。第二种方法的优点在于属性的名称被看作一个字符串，这就意味着它可以在运行时被计算，缺点在于这样的代码有可能无法在后期被解释器优化。它也可以被用来访问某些以[预留关键字](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Keywords)作为名称的属性的值：

```js
obj.for = "Simon"; // 语法错误，因为 for 是一个预留关键字
obj["for"] = "Simon"; // 工作正常
```

**注意：**从 EcmaScript 5 开始，预留关键字可以作为对象的属性名（reserved words may be used as object property names "in the buff"）。 这意味着当定义对象字面量时不需要用双引号了。参见 ES5 [Spec](http://es5.github.io/#x7.6.1).

关于对象和原型的详情参见： [Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype).

## 数组[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E6%95%B0%E7%BB%84)

JavaScript 中的数组是一种特殊的对象。它的工作原理与普通对象类似（以数字为属性名，但只能通过`[]` 来访问），但数组还有一个特殊的属性——`length`（长度）属性。这个属性的值通常比数组最大索引大 1。

创建数组的传统方法是：

```js
var a = new Array();
a[0] = "dog";
a[1] = "cat";
a[2] = "hen";
a.length; // 3
```

使用数组字面量（array literal）法更加方便：

```js
var a = ["dog", "cat", "hen"];
a.length; // 3
```

注意，`Array.length `并不总是等于数组中元素的个数，如下所示：

```js
var a = ["dog", "cat", "hen"];
a[100] = "fox";
a.length; // 101
```

记住：数组的长度是比数组最大索引值多一的数。

如果试图访问一个不存在的数组索引，会得到 `undefined`：

```js
typeof(a[90]); // undefined
```

可以通过如下方式遍历一个数组：

```js
for (var i = 0; i < a.length; i++) {
    // Do something with a[i]
}
```

ES2015 引入了更加简洁的[`for`...`of`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)循环，可以用它来遍历可迭代对象，例如数组：

```
for (const currentValue of a) {
  // Do something with currentValue
}
```

遍历数组的另一种方法是使用 [`for...in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in) 循环。注意，如果有人向 `Array.prototype` 添加了新的属性，使用这样的循环这些属性也同样会被遍历。所以并不推荐使用这种方法遍历数组：

```js
for (var i in a) {
  // Do something with a[i]
}
```

ECMAScript 5 增加了遍历数组的另一个方法 [forEach()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)：

```
["dog", "cat", "hen"].forEach(function(currentValue, index, array) {
  // Do something with currentValue or array[index]
});
```

如果想在数组后追加元素，只需要：

```js
a.push(item);
```

Array（数组）类自带了许多方法。查看[ array 方法的完整文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)。

| 方法名称                                           | 描述                                                         |
| -------------------------------------------------- | ------------------------------------------------------------ |
| `a.toString()`                                     | 返回一个包含数组中所有元素的字符串，每个元素通过逗号分隔。   |
| `a.toLocaleString()`                               | 根据宿主环境的区域设置，返回一个包含数组中所有元素的字符串，每个元素通过逗号分隔。 |
| a.concat(item1[, item2[, ...[, itemN]]])           | 返回一个数组，这个数组包含原先 `a` 和 `item1、item2、……、itemN` 中的所有元素。《译著：item1[, item2[, ...[, itemN]]]，参考命令行语法格式》 |
| `a.join(sep)`                                      | 返回一个包含数组中所有元素的字符串，每个元素通过指定的 `sep` 分隔。 |
| `a.pop()`                                          | 删除并返回数组中的最后一个元素。                             |
| a.push(item1, ..., itemN)                          | 将 `item1、item2、……、itemN` 追加至数组 `a`。                |
| `a.reverse()`                                      | 数组逆序（会更改原数组 `a`）。                               |
| `a.shift()`                                        | 删除并返回数组中第一个元素。                                 |
| `a.slice(start, end)`                              | 返回子数组，以 `a[start]` 开头，以 `a[end]` 前一个元素结尾。 |
| `a.sort([cmpfn])`                                  | 依据 `cmpfn` 返回的结果进行排序，如果未指定比较函数则按字符顺序比较（即使元素是数字）。 |
| a.splice(start, delcount[, item1[, ...[, itemN]]]) | 从 `start` 开始，删除 `delcount` 个元素，然后插入所有的 `item`。《译著：delcount[, item1[, ...[, itemN]]]，参考命令行语法格式》 |
| a.unshift(item1[, item2[, ...[, itemN]]])          | 将 `item` 插入数组头部，返回数组新长度（考虑 `undefined`）。《译著：item1[, item2[, ...[, itemN]]]，参考命令行语法格式》 |

## 函数[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%87%BD%E6%95%B0)

学习 JavaScript 最重要的就是要理解对象和函数两个部分。最简单的函数就像下面这个这么简单：

```js
function add(x, y) {
    var total = x + y;
    return total;
}
```

这个例子包括你需要了解的关于基本函数的所有部分。一个 JavaScript 函数可以包含 0 个或多个已命名的变量。函数体中的表达式数量也没有限制。你可以声明函数自己的局部变量。`return` 语句在返回一个值并结束函数。如果没有使用 `return` 语句，或者一个没有值的 `return` 语句，JavaScript 会返回 `undefined`。

已命名的参数更像是一个指示而没有其他作用。如果调用函数时没有提供足够的参数，缺少的参数会被 `undefined` 替代。

```js
add(); // NaN 
// 不能在 undefined 对象上进行加法操作
```

你还可以传入多于函数本身需要参数个数的参数：

```js
add(2, 3, 4); // 5
 // 将前两个值相加，4被忽略了
```

这看上去有点蠢。函数实际上是访问了函数体中一个名为 [`arguments`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments) 的内部对象，这个对象就如同一个类似于数组的对象一样，包括了所有被传入的参数。让我们重写一下上面的函数，使它可以接收任意个数的参数：

```js
function add() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum;
}

add(2, 3, 4, 5); // 14
```

这跟直接写成 `2 + 3 + 4 + 5` 也没什么区别。接下来创建一个求平均数的函数：

```js
function avg() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum / arguments.length;
}
avg(2, 3, 4, 5); // 3.5
```

这个很有用，但是却带来了新的问题。`avg()` 函数处理一个由逗号连接的变量串，但如果想得到一个数组的平均值该怎么办呢？可以这么修改函数：

```js
function avgArray(arr) {
    var sum = 0;
    for (var i = 0, j = arr.length; i < j; i++) {
        sum += arr[i];
    }
    return sum / arr.length;
}
avgArray([2, 3, 4, 5]); // 3.5
```

但如果能重用我们已经创建的那个函数不是更好吗？幸运的是 JavaScript 允许使用任意函数对象的`apply() 方法`来调用该函数，并传递给它一个包含了参数的数组。

```js
avg.apply(null, [2, 3, 4, 5]); // 3.5
```

传给 `apply()` 的第二个参数是一个数组，它将被当作 `avg()` 的参数使用，至于第一个参数 `null`，我们将在后面讨论。这也正说明一个事实——函数也是对象。

JavaScript 允许你创建匿名函数：

```js
var avg = function() {
    var sum = 0;
    for (var i = 0, j = arguments.length; i < j; i++) {
        sum += arguments[i];
    }
    return sum / arguments.length;
};
```

这个函数在语义上与 `function avg()` 相同。你可以在代码中的任何地方定义这个函数，就像写普通的表达式一样。基于这个特性，有人发明出一些有趣的技巧。与 C 中的块级作用域类似，下面这个例子隐藏了局部变量：

```js
var a = 1;
var b = 2;
(function() {
    var b = 3;
    a += b;
})();

a; // 4
b; // 2
```

JavaScript 允许以递归方式调用函数。递归在处理树形结构（比如浏览器 [DOM](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)）时非常有用。

```js
function countChars(elm) {
    if (elm.nodeType == 3) { // 文本节点
        return elm.nodeValue.length;
    }
    var count = 0;
    for (var i = 0, child; child = elm.childNodes[i]; i++) {
        count += countChars(child);
    }
    return count;
}
```

这里需要说明一个潜在问题——既然匿名函数没有名字，那该怎么递归调用它呢？在这一点上，JavaScript 允许你命名这个函数表达式。你可以命名立即调用的函数表达式（IIFES——Immediately Invoked Function Expressions），如下所示：

```js
var charsInBody = (function counter(elm) {
    if (elm.nodeType == 3) { // 文本节点
        return elm.nodeValue.length;
    }
    var count = 0;
    for (var i = 0, child; child = elm.childNodes[i]; i++) {
        count += counter(child);
    }
    return count;
})(document.body);
```

如上所提供的函数表达式的名称的作用域仅仅是该函数自身。这允许引擎去做更多的优化，并且这种实现更可读、友好。该名称也显示在调试器和一些堆栈跟踪中，节省了调试时的时间。

需要注意的是 JavaScript 函数是它们本身的对象——就和 JavaScript 其他一切一样——你可以给它们添加属性或者更改它们的属性，这与前面的对象部分一样。

## 自定义对象[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E8%87%AA%E5%AE%9A%E4%B9%89%E5%AF%B9%E8%B1%A1)

**备注：**关于 JavaScript 中面向对象编程更详细的信息，请参考 [JavaScript 面向对象简介](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Introduction_to_Object-Oriented_JavaScript)。

在经典的面向对象语言中，对象是指数据和在这些数据上进行的操作的集合。与 C++ 和 Java 不同，JavaScript 是一种基于原型的编程语言，并没有 `class` 语句，而是把函数用作类。那么让我们来定义一个人名对象，这个对象包括人的姓和名两个域（field）。名字的表示有两种方法：“名 姓（First Last）”或“姓, 名（Last, First）”。使用我们前面讨论过的函数和对象概念，可以像这样完成定义：

```js
function makePerson(first, last) {
    return {
        first: first,
        last: last
    }
}
function personFullName(person) {
    return person.first + ' ' + person.last;
}
function personFullNameReversed(person) {
    return person.last + ', ' + person.first
}
s = makePerson("Simon", "Willison");
personFullName(s); // Simon Willison
personFullNameReversed(s); // Willison, Simon
```

上面的写法虽然可以满足要求，但是看起来很麻烦，因为需要在全局命名空间中写很多函数。既然函数本身就是对象，如果需要使一个函数隶属于一个对象，那么不难得到：

```js
function makePerson(first, last) {
    return {
        first: first,
        last: last,
        fullName: function() {
            return this.first + ' ' + this.last;
        },
        fullNameReversed: function() {
            return this.last + ', ' + this.first;
        }
    }
}
s = makePerson("Simon", "Willison");
s.fullName(); // Simon Willison
s.fullNameReversed(); // Willison, Simon
```

上面的代码里有一些我们之前没有见过的东西：关键字 [`this`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/this)。当使用在函数中时，`this` 指代当前的对象，也就是调用了函数的对象。如果在一个对象上使用[点或者方括号](https://developer.mozilla.org/en/JavaScript/Reference/Operators/Member_Operators)来访问属性或方法，这个对象就成了 `this`。如果并没有使用“点”运算符调用某个对象，那么 `this` 将指向全局对象（global object）。这是一个经常出错的地方。例如：

```js
s = makePerson("Simon", "Willison");
var fullName = s.fullName;
fullName(); // undefined undefined
```

当我们调用 `fullName()` 时，`this` 实际上是指向全局对象的，并没有名为 `first` 或 `last` 的全局变量，所以它们两个的返回值都会是 `undefined`。

下面使用关键字 `this` 改进已有的 `makePerson`函数：

```js
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = function() {
        return this.first + ' ' + this.last;
    }
    this.fullNameReversed = function() {
        return this.last + ', ' + this.first;
    }
}
var s = new Person("Simon", "Willison");
```

我们引入了另外一个关键字：[`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)，它和 `this` 密切相关。它的作用是创建一个崭新的空对象，然后使用指向那个对象的 `this` 调用特定的函数。注意，含有 `this` 的特定函数不会返回任何值，只会修改 `this` 对象本身。`new` 关键字将生成的 `this` 对象返回给调用方，而被 `new` 调用的函数称为构造函数。习惯的做法是将这些函数的首字母大写，这样用 `new` 调用他们的时候就容易识别了。

不过这个改进的函数还是和上一个例子一样，单独调用`fullName()` 时会产生相同的问题。

我们的 Person 对象现在已经相当完善了，但还有一些不太好的地方。每次我们创建一个 Person 对象的时候，我们都在其中创建了两个新的函数对象——如果这个代码可以共享不是更好吗？

```js
function personFullName() {
    return this.first + ' ' + this.last;
}
function personFullNameReversed() {
    return this.last + ', ' + this.first;
}
function Person(first, last) {
    this.first = first;
    this.last = last;
    this.fullName = personFullName;
    this.fullNameReversed = personFullNameReversed;
}
```

这种写法的好处是，我们只需要创建一次方法函数，在构造函数中引用它们。那是否还有更好的方法呢？答案是肯定的。

```js
function Person(first, last) {
    this.first = first;
    this.last = last;
}
Person.prototype.fullName = function() {
    return this.first + ' ' + this.last;
}
Person.prototype.fullNameReversed = function() {
    return this.last + ', ' + this.first;
}
```

`Person.prototype` 是一个可以被`Person`的所有实例共享的对象。它是一个名叫原型链（prototype chain）的查询链的一部分：当你试图访问一个 `Person` 没有定义的属性时，解释器会首先检查这个 `Person.prototype` 来判断是否存在这样一个属性。所以，任何分配给 `Person.prototype` 的东西对通过 `this` 对象构造的实例都是可用的。

这个特性功能十分强大，JavaScript 允许你在程序中的任何时候修改原型（prototype）中的一些东西，也就是说你可以在运行时(runtime)给已存在的对象添加额外的方法：

```js
s = new Person("Simon", "Willison");
s.firstNameCaps();  // TypeError on line 1: s.firstNameCaps is not a function

Person.prototype.firstNameCaps = function() {
    return this.first.toUpperCase()
}
s.firstNameCaps(); // SIMON
```

有趣的是，你还可以给 JavaScript 的内置函数原型（prototype）添加东西。让我们给 `String`添加一个方法用来返回逆序的字符串：

```js
var s = "Simon";
s.reversed(); // TypeError on line 1: s.reversed is not a function

String.prototype.reversed = function() {
    var r = "";
    for (var i = this.length - 1; i >= 0; i--) {
        r += this[i];
    }
    return r;
}
s.reversed(); // nomiS
```

定义新方法也可以在字符串字面量上用（string literal）。

```js
"This can now be reversed".reversed(); // desrever eb won nac sihT
```

正如我前面提到的，原型组成链的一部分。那条链的根节点是 `Object.prototype`，它包括 `toString()` 方法——将对象转换成字符串时调用的方法。这对于调试我们的 `Person` 对象很有用：

```js
var s = new Person("Simon", "Willison");
s; // [object Object]

Person.prototype.toString = function() {
    return '<Person: ' + this.fullName() + '>';
}
s.toString(); // <Person: Simon Willison>
```

你是否还记得之前我们说的 `avg.apply()` 中的第一个参数 `null`？现在我们可以回头看看这个东西了。`apply()` 的第一个参数应该是一个被当作 `this` 来看待的对象。下面是一个 `new` 方法的简单实现：

```js
function trivialNew(constructor, ...args) {
    var o = {}; // 创建一个对象
    constructor.apply(o, args);
    return o;
}
```

这并不是 `new` 的完整实现，因为它没有创建原型（prototype）链。想举例说明 new 的实现有些困难，因为你不会经常用到这个，但是适当了解一下还是很有用的。在这一小段代码里，`...args`（包括省略号）叫作[剩余参数（rest arguments）](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/rest_parameters)。如名所示，这个东西包含了剩下的参数。

因此调用

```js
var bill = trivialNew(Person, "William", "Orange");
```

可认为和调用如下语句是等效的

```js
var bill = new Person("William", "Orange");
```

`apply()` 有一个姐妹函数，名叫 [`call`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)，它也可以允许你设置 `this`，但它带有一个扩展的参数列表而不是一个数组。

```js
function lastNameCaps() {
    return this.last.toUpperCase();
}
var s = new Person("Simon", "Willison");
lastNameCaps.call(s);
// 和以下方式等价
s.lastNameCaps = lastNameCaps;
s.lastNameCaps();
```

## 内部函数[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%86%85%E9%83%A8%E5%87%BD%E6%95%B0)

JavaScript 允许在一个函数内部定义函数，这一点我们在之前的 `makePerson()` 例子中也见过。关于 JavaScript 中的嵌套函数，一个很重要的细节是它们可以访问父函数作用域中的变量：

```js
function betterExampleNeeded() {
    var a = 1;
    function oneMoreThanA() {
        return a + 1;
    }
    return oneMoreThanA();
}
```

如果某个函数依赖于其他的一两个函数，而这一两个函数对你其余的代码没有用处，你可以将它们嵌套在会被调用的那个函数内部，这样做可以减少全局作用域下的函数的数量，这有利于编写易于维护的代码。

这也是一个减少使用全局变量的好方法。当编写复杂代码时，程序员往往试图使用全局变量，将值共享给多个函数，但这样做会使代码很难维护。内部函数可以共享父函数的变量，所以你可以使用这个特性把一些函数捆绑在一起，这样可以有效地防止“污染”你的全局命名空间——你可以称它为“局部全局（local global）”。虽然这种方法应该谨慎使用，但它确实很有用，应该掌握。

## 闭包[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E9%97%AD%E5%8C%85)

下面我们将看到的是 JavaScript 中必须提到的功能最强大的抽象概念之一：闭包。但它可能也会带来一些潜在的困惑。那它究竟是做什么的呢？

```js
function makeAdder(a) {
    return function(b) {
        return a + b;
    }
}
var x = makeAdder(5);
var y = makeAdder(20);
x(6); // ?
y(7); // ?
```

`makeAdder` 这个名字本身应该能说明函数是用来做什么的：它创建了一个新的 `adder` 函数，这个函数自身带有一个参数，它被调用的时候这个参数会被加在外层函数传进来的参数上。

这里发生的事情和前面介绍过的内嵌函数十分相似：一个函数被定义在了另外一个函数的内部，内部函数可以访问外部函数的变量。唯一的不同是，外部函数已经返回了，那么常识告诉我们局部变量“应该”不再存在。但是它们却仍然存在——否则 `adder` 函数将不能工作。也就是说，这里存在 `makeAdder` 的局部变量的两个不同的“副本”——一个是 `a` 等于5，另一个是 `a` 等于20。那些函数的运行结果就如下所示：

```js
x(6); // 返回 11
y(7); // 返回 27
```

下面来说说到底发生了什么。每当 JavaScript 执行一个函数时，都会创建一个作用域对象（scope object），用来保存在这个函数中创建的局部变量。它和被传入函数的变量一起被初始化。这与那些保存的所有全局变量和函数的全局对象（global object）类似，但仍有一些很重要的区别，第一，每次函数被执行的时候，就会创建一个新的，特定的作用域对象；第二，与全局对象（在浏览器里面是当做 `window` 对象来访问的）不同的是，你不能从 JavaScript 代码中直接访问作用域对象，也没有可以遍历当前的作用域对象里面属性的方法。

所以当调用 `makeAdder` 时，解释器创建了一个作用域对象，它带有一个属性：`a`，这个属性被当作参数传入 `makeAdder` 函数。然后 `makeAdder` 返回一个新创建的函数。通常 JavaScript 的垃圾回收器会在这时回收 `makeAdder` 创建的作用域对象，但是返回的函数却保留一个指向那个作用域对象的引用。结果是这个作用域对象不会被垃圾回收器回收，直到指向 `makeAdder` 返回的那个函数对象的引用计数为零。

作用域对象组成了一个名为作用域链（scope chain）的链。它类似于原型（prototype）链一样，被 JavaScript 的对象系统使用。

一个**闭包**就是一个函数和被创建的函数中的作用域对象的组合。

闭包允许你保存状态——所以它们通常可以代替对象来使用。[这里](http://stackoverflow.com/questions/111102/how-do-javascript-closures-work)有一些关于闭包的详细介绍。

### 内存泄露[节](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/A_re-introduction_to_JavaScript#%E5%86%85%E5%AD%98%E6%B3%84%E9%9C%B2)

使用闭包的一个坏处是，在 IE 浏览器中它会很容易导致内存泄露。JavaScript 是一种具有垃圾回收机制的语言——对象在被创建的时候分配内存，然后当指向这个对象的引用计数为零时，浏览器会回收内存。宿主环境提供的对象都是按照这种方法被处理的。

浏览器主机需要处理大量的对象来描绘一个正在被展现的 HTML 页面——[DOM](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model) 对象。浏览器负责管理它们的内存分配和回收。

IE 浏览器有自己的一套垃圾回收机制，这套机制与 JavaScript 提供的垃圾回收机制进行交互时，可能会发生内存泄露。

在 IE 中，每当在一个 JavaScript 对象和一个本地对象之间形成循环引用时，就会发生内存泄露。如下所示：

```js
function leakMemory() {
    var el = document.getElementById('el');
    var o = { 'el': el };
    el.o = o;
}
```

这段代码的循环引用会导致内存泄露：IE 不会释放被 `el` 和 `o` 使用的内存，直到浏览器被彻底关闭并重启后。

这个例子往往无法引起人们的重视：一般只会在长时间运行的应用程序中，或者因为巨大的数据量和循环中导致内存泄露发生时，内存泄露才会引起注意。

不过一般也很少发生如此明显的内存泄露现象——通常泄露的数据结构有多层的引用(references)，往往掩盖了循环引用的情况。

闭包很容易发生无意识的内存泄露。如下所示：

```js
function addHandler() {
    var el = document.getElementById('el');
    el.onclick = function() {
        el.style.backgroundColor = 'red';
    }
}
```

这段代码创建了一个元素，当它被点击的时候变红，但同时它也会发生内存泄露。为什么？因为对 `el` 的引用不小心被放在一个匿名内部函数中。这就在 JavaScript 对象（这个内部函数）和本地对象之间（`el`）创建了一个循环引用。

这个问题有很多种解决方法，最简单的一种是不要使用 `el` 变量：

```js
function addHandler(){
    document.getElementById('el').onclick = function(){
        this.style.backgroundColor = 'red';
    };
}
```

另外一种避免闭包的好方法是在 `window.onunload` 事件发生期间破坏循环引用。很多事件库都能完成这项工作。注意这样做将使 [Firefox 中的 bfcache](https://developer.mozilla.org/en-US/docs/Using_Firefox_1.5_caching) 无法工作。所以除非有其他必要的原因，最好不要在 Firefox 中注册一个`onunload` 的监听器。																@火狐