# 从封装el-doalog了解.sync 修饰符

在二次封装el-dialog遇到一些坑

## 直接上手封装 入坑

css-dialog.vue
```
<el-dialog :visible.sync="visible"></el-dialog>

<script>
  export default {
    props: {
      visible: {
        typeof: Boolean,
        default: false
      }
    }
  }
</script>
```

在父组件调用 css-dialog这个组件 传入`visible`属性来处理显隐

```
<button @click="visible = true">打开dialog<button>

<css-dialog :visible.sync="visible"></css-dialog>
```

完美，点击按钮后 css-dialog完美打开 
但是点击右上角的 X 关闭后，报错
```
[Vue warn]: Avoid mutating a prop directly since the value will be overwritten whenever the parent component re-renders. Instead, use a data or computed property based on the prop's value. Prop being mutated: "visible"
```

然后再次点击按钮，css-dialog根本打不开

## 寻找原因 中坑

这很明显 问题就出现到`visible`后面带一串东西`.sync` 聪明机智的我 果然打开vue官网查看了一下[.sync 修饰符](https://cn.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6)

看的懵懵懂懂的,这太官方了，但是有一点我还是明白的 这是因为双向绑定prop的问题，第一时间就想到 是因为在props的问题，我不直接修改这个props，用计算函数在定义另外个属性行不行呢？
按着想着就改了下 

css-dialog.vue
```
<el-dialog :visible.sync="newVisible"></el-dialog>

<script>
  export default {
    props: {
      visible: {
        typeof: Boolean,
        default: false
      }
    },
    computed: {
      newVisible () {
        return this.visible
      }
    }
  }
</script>
```


```
<button @click="visible = true">打开dialog<button>

<css-dialog :visible.sync="visible"></css-dialog>
```

点击按钮，css-dialog能打开了，但是问题还是没那么容易绕过去，点击右上角的X 还是报错
不过是另外的报错 
`vue.runtime.esm.js:619 [Vue warn]: Computed property "newVisible" was assigned to but it has no setter`
没有设置set? 
## 看源码 晚坑

这el-dialog的右上角执行了什么逻辑呢？ 好家伙 看了源码才知道
```
hide(cancel) {
  if (cancel !== false) {
    this.$emit('update:visible', false);
    this.$emit('close');
    this.closed = true;
  }
},
```
`this.$emit('update:visible', false);` 似曾相识，我又打开了[.sync 修饰符](https://cn.vuejs.org/v2/guide/components-custom-events.html#sync-%E4%BF%AE%E9%A5%B0%E7%AC%A6)文档
这次细品就能品出味道来了，原来`.sync`来进行双向绑定 那么el-dialog内 我子组件要修改这个visible 就要通过`this.$emit('update:visible', false)` 事件来替代触发
二次封装的时候 点击右上角的按钮的时候，也要对props绑定的`visible`进行触发 
那么就是说，二次封装的时候 用 `this.$emit('update:visible')`在触发更新这个双向绑定不就完事了吗？ 机智如我

## 终 完坑

css-dialog.vue
```
<el-dialog :visible.sync="newVisible"></el-dialog>

<script>
  export default {
    props: {
      visible: {
        typeof: Boolean,
        default: false
      }
    },
    computed: {
      newVisible: {
        get () {
          return this.visible
        },
        set (val) {
          this.$emit('update:visible', val)
        }
      }
    }
  }
</script>
```


```
<button @click="visible = true">打开dialog<button>

<css-dialog :visible.sync="visible"></css-dialog>
```
、
这就完美的处理了封装的显隐，在中坑的时候 点击右上角的X报错 提示没有设置set, 那么就直接`this.$emit('update:visible', val)`时间对`visible`的双向绑定进行一次更新，最终子组件`el-dialog`也能接受到更新的变化了

