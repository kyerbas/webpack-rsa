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
