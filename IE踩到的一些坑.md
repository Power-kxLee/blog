# 记录开发过程中遇到的IE坑

## event.path的一些小问题
1. `event.path`很方便的获取所有路径节点。但是在`IE11`、`firefox`中就会收到错误`event.path undefined`.
解决办法:
```
function composedPath (el) {
    var path = [];
    while (el) {
        path.push(el);
        if (el.tagName === 'HTML') {
            path.push(document);
            path.push(window);
            return path;
       }
       el = el.parentElement;
    }
}
var path = event.path || (event.composedPath && event.composedPath()) || composedPath(event.target);
```
## initial的一些小问题
2. `xxx.style.display = 'initial'` 设置成`initial`会无效。 
解决办法: 
```
xxx.style.display = ''
```
##关 于iframe创建后load的问题（巨坑）

3. 关于iframe创建后load的问题（巨坑）
```
iframe = document.createElement('iframe');
document.body.appendChild(iframe);
```

动态创建iframe后，我用`onload`事件监听，原代码如下:

```
paWindow.onload = () => {
            // 省略一万行逻辑...
}
```

chorme浏览器正常。Firefox浏览器正常。IE偏偏出来搞事，没有进入load的逻辑
查阅了一些资料，找到了I（狗）E（B）浏览器专属方法，`attachEvent` 来绑定事件 ，IE11废弃了这个方法，那么就从IE9往上开始兼容
代码更改成 如下

```
paWindow.attachEvent('onload', () => {
             // 省略一万行逻辑...
        });

```
满怀希望的进入测试，还是没进入load事件，一脸蒙蔽，console打印 `paWindow.attachEvent`出来结果，告诉我已经有这个方法了啊，为毛还是不行，实在苦恼

```
function attachEvent() {    [native code]}
```

然后怀疑是不是传入的参数，ifame不对，经过很多排查也是没问题
考虑到插件的特殊情况，iframe用src其他链接，不存在异步的情况，是最后write一段字符串而已，还是为了防止意外发生，最后用定时器更改了一下。
然后代码改成，还是发生了意外

```
const _loaded = () => {
             // 省略一万行逻辑...
   };
if (iframe.attachEvent) {
    setTimeout(_loaded, 1);
} else {
    paWindow.onload = _loaded();
}
```

经过测试发生，咦，怎么IE11 也变了正常了，不应该啊？？ 因为IE11 就是没经过`onload`，这真的意外之喜啊
会不会IE9，IE10 也好了呢？？？
最终代码改成

```
const _loaded = () => {
        // 省略一万行逻辑...
    };
    paWindow.onload = _loaded();
```

卧槽。卧槽。真尼玛的好了， IE9，IE10，IE11，chrome，Firefox 也都好了
这尼玛什么鬼？？？？谁可以告诉我

```
paWindow.onload = function() {
  };
 paWindow.onload = _loaded();
```

这两个调用方法有催子不一样？？？难道下面的比较帅？ 奇葩奇葩



