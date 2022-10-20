# webpack-rsa
js逆向webpack+rsa实战2
#登陆https://account.ppdai.com/pc/login?returnUrl=https://tz.ppdai.com/account/indexV2
1、登陆抓包,需要扣phone的加密方法
<img width="1717" alt="image" src="https://user-images.githubusercontent.com/85286900/196977257-9eb1f0ba-7c05-4e3d-9ee9-22b5036efca7.png">
2、搜索关键字：phone，定位到js
<img width="1622" alt="image" src="https://user-images.githubusercontent.com/85286900/196977496-f55762f1-23e3-4896-9c48-421786d15bb1.png">
3、最终定位到这里；因为前面还是明文，经过n.$encrypt.encrypt(n.reg_mobile)这个代码之后加密，所以确定需要扣的函数是：n.$encrypt.encrypt：
<img width="1464" alt="image" src="https://user-images.githubusercontent.com/85286900/196977975-a1869f29-cac4-4c7b-8bf5-91bb65c12160.png">
4、点击：n.$encrypt.encrypt进入：
<img width="601" alt="image" src="https://user-images.githubusercontent.com/85286900/196978463-5f6cc6f3-8848-4b13-bc52-23f1ca745541.png">

5、发现最终返回的是：this.getKey().encrypt(t)方法调用，这个方法就不要跟进去了，里面是具体的加密逻辑

6、确定需要扣：this.getKey().encrypt(t)所在模块

7、这里记录另一种webpack扣法，之前一直是用自执行，这次用变种

8、先把它的js头部webpack加载器扣下来
```javascript
//这是目标网站的加载器
!function(t) {
    function e(r) {
        if (n[r])
            return n[r].exports;
        var i = n[r] = {
            i: r,
            l: !1,
            exports: {}
        };
        return t[r].call(i.exports, i, i.exports, e),
        i.l = !0,
        i.exports
    }
    var n = {};
    e.m = t,
    e.c = n,
    e.i = function(t) {
        return t
    }
    ,
    e.d = function(t, n, r) {
        e.o(t, n) || Object.defineProperty(t, n, {
            configurable: !1,
            enumerable: !0,
            get: r
        })
    }
    ,
    e.n = function(t) {
        var n = t && t.__esModule ? function() {
            return t.default
        }
        : function() {
            return t
        }
        ;
        return e.d(n, "a", n),
        n
    }
    ,
    e.o = function(t, e) {
        return Object.prototype.hasOwnProperty.call(t, e)
    }
    ,
    e.p = "",
    e(e.s = 277)
}([

```

9、以前自执行方式修改后的代码
```javascript
!function(t) {
    function e(r) {
        if (n[r])//这行也可以删除
            return n[r].exports;//这行也可以删除
        var i = n[r] = {//这行n删除后，可以改成var i = {
            i: r,
            l: !1,
            exports: {}
        };
        return t[r].call(i.exports, i, i.exports, e),
        i.l = !0,
        i.exports
    }
    var n = {};//这行也可以删除
}([])
```
10、现在另一种方式直接函数调用
```javascript

var fangfa1 = function(t,e){...}//定义个变量fangfa1用来保存扣下来的方法，function(t,e){}就是扣下来的方法

var t = [fangfa1]//定义t数组，把fangfa1添加到数组，注意这个t和return t[r].call(i.exports, i, i.exports, e),中的t是同一个

function e(r) {
        var i =  {
            i: r,
            l: !1,
            exports: {}
        };
        return t[r].call(i.exports, i, i.exports, e),
        i.l = !0,
        i.exports
    }
e(0);//调用模块，这个意思是把0传给e方法，e方法实际返回：return t[r].call(i.exports, i, i.exports, e)；就是返回t数组第0个方法的调用
```


11、扣到的加密函数所在完整模块：kou.js

12、分析扣出来的代码
```javascript
//扣出来的代码，最后那个自执行方法太长，省略了代码
function(t, e, n) {
    "use strict";
    function r(t) {
        return t && t.__esModule ? t : {
            default: t
        }
    }
    var i, o, a, s = n(65), c = r(s), u = n(110), l = r(u), d = n(111), p = r(d), f = n(114), h = r(f);
    !function(n, r) {
        "object" == (0,
        h.default)(e) && void 0 !== t ? r(e) : (o = [e],
        i = r,
        void 0 !== (a = "function" == typeof i ? i.apply(e, o) : i) && (t.exports = a))
    }(0, function(t) {...})

/////////
}

//****************分析代码**********************************

function(t, e, n) {
    "use strict";
    function r(t) {
        return t && t.__esModule ? t : {
            default: t
        }
    }
    var i, o, a, s = n(65), c = r(s), u = n(110), l = r(u), d = n(111), p = r(d), f = n(114), h = r(f);
    !function(n, r) {//这里是一个自执行方法，发现这整个大方法最终目的就是运行这个自执行方法，所以上面部分调用其它模块的n(110)等等可以不用扣，直接再分析自执行代码
        "object" == (0,
        h.default)(e) && void 0 !== t ? r(e) : (o = [e],
        i = r,
        void 0 !== (a = "function" == typeof i ? i.apply(e, o) : i) && (t.exports = a))
    }(0, function(t) {...})

/////////
}
*****************分析自执行方法****************************
    //还需要结合模块加载器
 var t = [jsjiemi]//把jsjiemi方法存入t数组
 
!function(t) {
    function e(r) {
        if (n[r])//这行也可以删除
            return n[r].exports;//这行也可以删除
        var i = n[r] = {//这行n删除后，可以改成var i = {
            i: r,
            l: !1,
            exports: {}
        };
        return t[r].call(i.exports, i, i.exports, e),//e(0)运行的时候，就是调用t数组模块中的第0个也就是jsjiemi方法，(i.exports, i, i.exports, e)这几个参数是传给被调用的jsjiemi方法的，第一个参数不算(这点自己也不懂),i是个对象,i.exports是空,e是方法本身
        i.l = !0,
        i.exports
    }
    var n = {};//这行也可以删除
}(t)//把存模块的t数组传入进去
e(0);//调用第0个模块，也就是jsjiemi
/////分析
var jsjiemi = function(t, e, n) {//t是传过来的i对象,e是空i.exports是空对象
    "use strict";
    function r(t) {
        return t && t.__esModule ? t : {
            default: t
        }
    }
    var i, o, a, s = n(65), c = r(s), u = n(110), l = r(u), d = n(111), p = r(d), f = n(114), h = r(f);
    !function(n, r) {//这里是一个自执行方法，发现这整个大方法最终目的就是运行这个自执行方法，所以上面部分调用其它模块的n(110)等等可以不用扣，直接再分析自执行代码
                    //这个自执行的第一个参数是0，第二个参数是方法
        "object" == (0,
        h.default)(e) && void 0 !== t ? r(e) : (o = [e],//e是空对象，所以这里改写成&& void 0 !== t ? r({}) : (o = [e]，所以r也就是自执行的方法，只需要一个空对象就可以运行，结合上面所说，这个大方法最终目的是运行这个自执行方法，而这个自执行方法中第二个参数方法又只需要一个空对象就可以运行，这个自执行方法中的最终目的也是运行第二个参数的方法，所以只需要扣第二个方法并给它穿入空对象就行
        i = r,
        void 0 !== (a = "function" == typeof i ? i.apply(e, o) : i) && (t.exports = a))
    }(0, function(t) {...})

/////////
}
```
12、结合上面分析，最后只需要把扣好的js中的自执行第二个方法扣下来，并给它传个空对象就可以
```javascript
//并且构造下自执行
!(function(t) {...})({})

```

13、构造好后放到浏览器控制台调试，会报错：Uncaught ReferenceError: p is not defined，定位到代码：
```javascript
B = p.default || {
            __proto__: []
        }
 //分析，如果p.default为空，就会赋第二个值，所以这里的p.default直接删掉，让他每次都赋第二个值
```

14、代码扣完，发现他自己在最后已经把加密方法所在的方法导出来了

```javascript
        window.JSEncrypt = ot,
        t.JSEncrypt = ot,
        t.default = ot,
        Object.defineProperty(t, "__esModule", {
            value: !0
        })
```
15、最后按照rsa流程，运行
```javascript
//第一步 var xxx = new rsa;
var aaa = new window.JSEncrypt;
//第二步设置setPublicKey，由于这个站的plblickey没找到，就不继续找了
aaa.setPublicKey('')
//第三步调用加密方法
aaa.encrypt('密码')
```
