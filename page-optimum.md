# 页面优化总结

概要：项目中页面优化经验

## 一、	使用cdn

1. 项目中的css，js，图片全部走cdn资源，但是并不是cdn的资源一定就是靠谱的，本人使用的阿里云的服务，但是偶尔也会出现到cdn资源加载不上的情况，所以需要在页面上做容错处理，保证服务的稳定。处理的方式就是静态资源不仅往cdn上的发布，同时服务器上也保留一份，当页面中发现cdn资源加载不上的时候就走服务器上的资源。

```
    document.addEventListener("DOMContentLoaded", function(event) {
        var url = window.location.href.split("?") // 获取页面带的参数
        if(url[1]){
          url = url[1].split("#/")
          if(!document.getElementById("app").offsetParent){
            window.location.href = "backup.html?" + url[0] // 判断css是否加载，页面上把app元素先隐藏然后再css文件中show出来
          }
          if (typeof $ === 'undefined'){
            window.location.href = "backup.html?" + url[0] // 判断js是否加载，主要查引入变量是否定义了
          }
        }
      });


      <style type="text/css">
      #app{
        display: none;
      }
    </style>

```

2.通过 DNS 预解析来告诉浏览器未来我们可能从某个特定的 URL 获取资源，当浏览器真正使用到该域中的某个资源时就可以尽快地完成 DNS 解析。

当我们从该 URL 请求一个资源时，就不再需要等待 DNS 的解析过程。该技术对使用第三方资源特别有用。
```
    <link rel="dns-prefetch" href="//example.com">
```


## 二、	资源压缩合并，减少 HTTP 的请求

通过webpack进行js,css的合并和压缩，还有图片的处理（默认是小于100k的图片会base64转码放入css文件中），这写的配置在各个脚手架里面都是默认存在的，只需要自己多多留意一下即可

## 三、	js的懒加载

当项目越来越庞大，一次性的加载所有的js就显得非常不必要了，这个时候可以使用webpack里本身就带有的require方式就可以解决了

比如vue的项目中：
```
    const test = resolve => {
      require.ensure(['@/components/test'], () => {
        resolve(require('@/components/test'))
      })
    }
```

webpack打包以后，这些require的component都可以实现router跳转的时候再进行加载

## 四、	非核心的代码异步加载

#### 1. 动态加载
所谓动态加载脚本就是利用 javascript 代码来加载脚本，通常是手工创建 script 元素，然后等到 HTML 文档解析完毕后插入到文档中去。这样就可以很好地控制脚本加载的时机，从而避免阻塞问题。
```
function loadJS(src) {
  const script = document.createElement('script');
  script.src = src;
  document.getElementsByTagName('head')[0].appendChild(script);
}
loadJS('http://example.com/scq000.js');
```
#### 2. 异步加载方式
1） defer
```
<script src="file.js" defer></script> 
```
**注:** defer 是在 HTML 解析完之后才会执行，如果是多个，按照加载的顺序依次执行。

2） async
```
<script src="file.js" async></script> 
```
**注:** async 属性是 HTML5 新增的。作用和 defer 类似，但是它将在下载后立即执行，**不能保证脚本会按顺序执行**。它们将在 onload 事件之前完成。

**上面介绍的异步加载脚本并不是十分完美的**, 如果你熟悉 promise 的话，就知道这是在 JS 中处理异步的一种强有力的工具。下面以 promise 技术来实现处理异步脚本加载过程中的依赖问题：
```
// 执行脚本
function exec(src) {
    const script = document.createElement('script');
    script.src = src;

      // 返回一个独立的promise
    return new Promise((resolve, reject) => {
        var done = false;

        script.onload = script.onreadystatechange = () => {
            if (!done && (!script.readyState || script.readyState === "loaded" || script.readyState === "complete")) {
              done = true;

              // 避免内存泄漏
              script.onload = script.onreadystatechange = null;
              resolve(script);
            }
        }

        script.onerror = reject;
        document.getElementsByTagName('head')[0].appendChild(script);
    });
}

function asyncLoadJS(dependencies) {
    return Promise.all(dependencies.map(exec));
}

asyncLoadJS(['https://code.jquery.com/jquery-2.2.1.js', 'https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js']).then(() => console.log('all done'));
```
可以看到，我们针对每个脚本依赖都会创建一个 promise 对象来管理其状态。采用动态插入脚本的方式来管理脚本，然后利用脚本 onload 和 onreadystatechange (兼容性处理)事件来监听脚本是否加载完成。一旦加载完毕，就会触发 promise 的 resovle 方法。最后，针对依赖的处理，是 promise 的 all 方法，这个方法只有在所有 promise 对象都 resolved 的时候才会触发 resolve 方法，这样一来，我们就可以确保在执行回调之前，所有依赖的脚本都已经加载并执行完毕。

## 五、   一些不稳定的第三方库
项目中会引入一些统计代码，同时还有针对一些页面事件的统计，如果第三方的库不稳定，然后你又正好把这些代码放在你逻辑的第一行，那么你的代码会在第三方库不稳定的时候被阻塞，然后导致使用的时候，看起来页面就失灵了。

所以除了你把这些统计事件放到最后执行的处理以外，你还可以在onload之后判断一下这个第三方库是否真的加载了，如果没有，那你就把他的方法重新定义一下。

比如使用友盟的统计代码的时候：
```
    <script src="https://s19.cnzz.com/z_stat.php?id=1231312&web_id=123132" language="JavaScript"></script>

    if (!window._czc || (window._czc && typeof(window._czc.push) != 'function')) {
        window._czc = {
          push: function (a) {
          }
        }
    }
```

同时在使用这些统计事件的时候先判断一下：
```
    if (!!_czc && !!_czc.push) { // important!
        _czc.push(['_trackEvent', 'test', 'test'])
    }
```
