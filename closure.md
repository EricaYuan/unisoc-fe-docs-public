# 微前端前期调研过程
前期ipd项目提出需求：需要在同一界面切换不同系统，考虑到如果将所有项目全都写到同一个项目中会有很大的问题，于是考虑用**嵌入子项目的方式来实现js和样式的隔离**。由此，考察到**qiankun这**个微前端框架。

> qiankun是支持vue、react、jquery等多种框架进行集成开发的，但由于技术栈限制，目前提供的模板项目IPDDEMO中的所有项目都是基于vue开发的。
# 参考项目
示例: [uni-ipd](http://10.0.0.174:8104/)
示例源码svn地址: [IPDDEMO](http://shexsvn01/!/#ProcessInformation/view/head/WebCode/2020-12-Project_UniIPD/05.SourceCode/02.Frontend/01.SourceCode/qiankun-ipddemo)

## uni-ipd项目架构
![qiankun项目页面架构](https://img-blog.csdnimg.cn/20210310145403511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbGxlbll1YW4=,size_16,color_FFFFFF,t_70#pic_center)
## uni-ipd代码目录结构
```md
IPDDEMO
|--- ipd-base （基座项目）
|       |—— src
|       |    |---micro-app.js // 配置子项目
|       |    |---main.js // 引入qiankun，注册子项目 41-43行 / 兼容IE11
|       |    |---App.js // 获取用户信息登录
|       |
|       |—— shared 
|       |     |--- actions.js // 声明globalState
|       |—— Vue.config.js // qiankun中基座项目的必要配置
|
|--- ipd-vue （子项目一）
|       |—— src
|       |    |---main.js // 子项目的main.js中需要export生命周期函数
|       |—— Vue.config.js // qiankun中vue子项目的必要配置
|       |—— .env.development // 子项目开发环境的资源请求路径
|       |—— .env.production // 子项目生产环境的资源请求路径
|       |—— .env.test// 子项目测试环境的资源请求路径
|
|--- ipd-vueapp （子项目二 同ipd-vue）
|       
|--- package-lock.json
|
|--- package.json  // 配置根目录run all命令
```
# qiankun项目集成子项目的具体步骤：
以IPDDEMO为例:

1. ipd-base为基座项目，一般来讲不需要进行配置修改，只需在添加子项目时进行修改。
2. 新建子项目（建议a）：
    a.复制一个ipd-vue，修改为需要的项目名称（要同步修改package.json中的name）；
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310151547382.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbGxlbll1YW4=,size_16,color_FFFFFF,t_70)
    b.create新项目，要把vue.config.js中的配置改为与ipd-vue一样）,并且新建  .env.development 和  .env.production
    ```javascript
    // IPDDEMO>ipd-vue>vue.config.js
    const packageName = require('./package.json').name;
	module.exports = {
	  publicPath: process.env.PUBLIC_PATH,
	  transpileDependencies: ['common'],
	  chainWebpack: config => config.resolve.symlinks(false),
	  configureWebpack: {
	    output: {
	      library: 'ipd-vue',
	      libraryTarget: 'umd',
	      jsonpFunction: `webpackJsonp_${packageName}`
	    }
	  },
	  devServer: {
	    port: process.env.VUE_APP_PORT,
	    headers: {
	      'Access-Control-Allow-Origin':'*'
	    },
	    sockHost: `localhost:${process.env.VUE_APP_PORT}`,
	    disableHostCheck: true
	  }
	}

3. 修改子项目根目录下的.env文件里port参数，即为新建项目的端口号；
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310151941303.png)
6. 将新的子项目添加到ipd-base中：
    a. ipd-base的micro-app.js中配置子项目, `activeRule`是跳转到该子项目的url
	```javascript
	// IPDDEMO>ipd-base>src>micro-app.js
	export const microApps = [
	  {
	    name: 'ipd-vue',
	    entry: process.env.VUE_APP_SUB_VUE,
	    activeRule: '/ipd-vue'
	  },{
	    name: 'ipd-newproject',
	    entry: process.env.VUE_APP_SUB_NEW,
	    activeRule: '/ipd-newproject'
	  }
	]
	```
	
	b. 在ipd-base目录下的.env.development、.env.production和.env.test中添加新增子项目的环境变量。
	
	```javascript
	NODE_ENV=development // or production or test
	VUE_APP_MODE=development // or production or test
	VUE_APP_SUB_VUE=http://localhost:8105/
	VUE_APP_SUB_NEW=http://new-project-url/
	```
	`子项目的请求路径需要根据生产、测试、开发环境机动配置`
	
	c. 通过修改ipd-base项目的App.vue文件中的`tempsideNavList`给左侧菜单栏添加新项目连接，如下图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210310155701767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbGxlbll1YW4=,size_16,color_FFFFFF,t_70)
# 其他相关配置
## 修改默认启动项目 ： 
```javascript
setDefaultMountApp('/ipd-vue') // 在ipd-base的main.js最后，修改默认启动app的url即可。在ipd-base的main.js最后，修改默认启动app的url即可。
```

## 关于项目运行

1. 可以在IPDDEMO的package.json中配置run-all命令，**配置好后**可以在根目录IPDDEMO中执行命令：

```javascript
npm install   // 该命令会自动给根目录下所有项目安装依赖包
npm start // 该命令会自动启动根目录下所有项目
```
2. **如果没有配置或不懂怎么配置run-all**，就直接单独运行每个项目，每个项目都是正常的vue项目操作步骤

```javascript
// 以ipd-vue为例：
cd ipd-vue
npm install
npm run serve
```

## 关于子项目菜单配置： 

```javascript
1. 本地环境配置npm到内网，终端运行下面命令：
npm config set registry https://repo.unisoc.com/nexus/repository/npm-public/

2. 项目安装unisoc-ui：npm i unisoc-ui --save

3. main.js中引入依赖包：见ipd-vue的main.js

import ElementUI from 'element-ui';
import unisocUI from 'unisoc-ui'
import 'unisoc-ui/style-default/index'
Vue.use(unisocUI);
Vue.use(ElementUI);

```
组件具体使用方法见：[unisoc-ui用法文档](http://10.29.60.94:8282/#/ipdnavbar)

## docker发布：
### docker发布的nginx配置文件：
```javascript
client_max_body_size 500m;

server {
    listen       8080;
    listen  [::]:8080;
    server_name  localhost;
    server_name_in_redirect off;
    port_in_redirect off;
    absolute_redirect off;
    add_header Access-Control-Allow-Origin *;
    add_header Access-Control-Allow-Methods 'GET,POST';
    add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization'; 
    location / {
        root   /usr/share/nginx/html;
        index  index.html;
        try_files $uri $uri/ /index.html;
   	}
   	location /ipd-gateway/ {
        proxy_http_version 1.1;
        proxy_set_header Host ipd-gateway-test.testground.10.29.21.11.xip.io;
        proxy_set_header  X-Real-IP        $remote_addr;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_set_header X-NginX-Proxy true;
        add_header backendIP $upstream_addr;
        add_header backendCode $upstream_status;
        proxy_pass http://ipd-gateway-test.testground.10.29.21.11.xip.io/;
        error_page 401 = @error401;
    }
    error_page  404              /404.html;
}

```

### docker发布的具体流程：
见bo.hong整理的文档：[docker前端发布](http://10.0.0.130:9006/pages/viewpage.action?pageId=3440692)

> 有任何问题，可以发邮件：tonghui.yuan@unisoc.com