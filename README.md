# DataX-UI

## 说明

[前段项目github地址](https://github.com/deerlili/datax-ui)
[后端项目github地址](https://github.com/deerlili/datax-backend)

该项目修改而来 [vue-element-admin](https://github.com/PanJiaChen/vue-element-admin) 


## Build Setup

运行

```
npm install --registry https://registry.npm.taobao.org
```

修改配置

找到 `vue.config.js` 修改 `proxy` 里的属性即可

```
[process.env.VUE_APP_API]: {
        target: `http://localhost:${apiPort}/api`,
        changeOrigin: true,
        pathRewrite: {
          ['^' + process.env.VUE_APP_API]: ''
        }
```

启动 

```
 npm run dev
```

打包

```
npm run build:prod
```
