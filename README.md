# frontend-mqtt

## 主要依赖
 - [async-mqttjs](https://github.com/mqttjs/async-mqtt) >=2.5.0
 - [vue.js](https://github.com/vuejs/vue) >=2.6.11
 - [uuid](https://github.com/uuidjs/uuid) >=3.4.0

配置文件位于`/src/utils/`
```ShellSession
cp ./src/utils/config.example.js ./src/utils/config.js 
# 将config.example.js 复制为 config.js
# 然后打开config.js进行修改
vim config.js
```

### 配置项说明
|属性名|类型|描述|
|:-:|:-:|:-|
|host|String|填写你的mqtt中间件域名或ip地址|
|path|String|填写mqtt的ws path 默认为/mqtt|
|port|String or Number|填写mqtt的ws 端口，默认为8083|
|user|String|用户名|
|password|String|密码|
|subscribe|Array|订阅主题列表|

订阅主题列表为一个有若干个格式如下的数组
```js
{ 
    name: "test", //主题名称
    desc: "测试频道" // 主题描述
}
```

### 数据包格式说明

数据包是一个`JSON Object`

|属性名|类型|描述|
|:-:|:-:|:-|
|id|String|完全随机并符合RFC uuid v4的唯一且独特标识码。如：8e57bf66-8125-42b0-8992-a84915864b94|
|time|Number|Unix时间戳|
|from|String|用于标识客户端的字符，代码中使用uuid v4作为标识|
|to|String|主题名|
|text|String|发送的消息|

接收到的数据包与发送数据包一致

## 安装依赖
```
yarn install
```

### 开发
```
yarn serve
```

### 编译
```
yarn build
```
