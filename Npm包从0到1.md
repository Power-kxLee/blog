# npm包从0到1，创建属于自己的模块

## 开始创建
到npm官网注册你的凭证 [https://www.npmjs.com/](https://www.npmjs.com/)
详细自己去注册，省略
打开cmd 输入
```
npm login
```
登陆你注册好的凭证，然后就可以上传你的包了
cmd进入你得项目路径，如果项目还没有创建配置
输入
```
npm init
```
会创建一个文件`package.json`，需要注意下几点

* `name`: 你创建包的名字，不要跟其他人的重复了。
* `version`: 当前版本号，第一次的包一般都是`1.0.0`
* `description`: 包的一些描述
* `main`: 默认的入口文件
* `author`: 作者
* `private`: 是否私有，这里要设置成`false`才可以发布哦
* `license`: 开源协议 默认`MIT`

好了，一切都准备好了
输入
`npm publish`

## 验证
发布了之后，直接登陆npm官网后台就能看到你得包信息
当然你也可以下载
`npm install testbb`
注意，如果你发的是测试包 ，最好还是撤销下来
`npm --force unpublish testbb`

## 更新 
假设你的包名叫`testbb`
可以查询这个包的一些版本
```
npm view testbb versions
```
假设查询出来的版本是`1.0.3`

修改后，我需要提交版本。
这里可以使用命令 `npm version <update_type>`
`update_type `有三个参数
* patch: 补丁的意思 比如版本是`1.0.3` ，最后一位数增加1. 变成`1.0.4`
* minor: 小改的意思 比如版本是`1.0.3` ，中间一位数增加1. 变成`1.1.3`
* major: 大改的意思 比如版本是`1.0.3` ，中间一位数增加1. 变成`2.0.3`
大概就是这样的意思，自行尝试

然后继续提交吧 (最喜欢输入这行命令了)
`npm publish`
