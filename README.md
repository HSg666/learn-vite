# 技术栈： Vue 3.3 / TS 5.2 / Vite5.0 / Pinia2.1.7 / router4.2.5 
# 环境： Node:16.20.1    pnpm:8.14.0
# 代码管理工具：sourceTree

# 命令
```js
cd learn-vite    // 切换
pnpm i           // 装依赖
pnpm start       // 启动
pnpm run build   // 打包
rm -rf node_modules  // 强行删除依赖包
```
准备打包上线时请看搭建二期的 <a href="#requestURL">新增环境变量  优化请求地址</a>，检查完配置后再执行pnpm run build 打包

这个项目有什么东西呢？
# 搭建一期

## 1、务必装上安装pnpm 
没装的看这篇文章  https://blog.csdn.net/Steven_Son/article/details/135151622 
## 2、新增 .nvmrc
每次切换项目都手动切换 node 版本
## 3、CSS预处理器用的是SCSS
## 4、公共样式表
common.scss  需要初始化全局样式可以在此文件编辑
## 5、autoprefiexer 自动增加css浏览器兼容前缀
## 6、windicss  第三方便捷性css库
## 7、封装Antd / Vant-ui按需导入 
## 8、pinia 封装统一管理存储库 
## 9、router 封装 
## 10、配置路径别名 alias
示例：@/store   只要在src下的都能这样简写

总共分为4步：

1、vite.config.ts

```js
import path from 'path'
export default defineConfig({
	//新增
	resolve: {
		alias: {
			'@/assets': path.resolve(__dirname, './src/assets'),
		},
	},
})
```
2、tsconfig.json
```js
 "paths": {
      "@/assets/*": ["src/assets/*"],
    }
```
3、配置好页面使用
例如main.ts引入
```js
// 引入全局样式
import '@/assets/scss/index.scss'
```

4、更改完vite.config.ts和tsconfig.json记得重启项目。
## 11、polyfill web项目兼容低版本浏览器插件
core-js 和 @vitejs/plugin-legacy
## 12、prettier + eslint 配置了代码规范插件
## 13、husky + lint-staged  git提交规范
## 14、browserslist 配置了浏览器兼容性


### 备份
#### package.json

去掉模块化

```js
 "type": "module",
```

### git commit 提交规范

- feat：新功能（feature）
- fix/to：修复 bug，可以是 QA 发现的 BUG，也可以是研发自己发现的 
- fix：产生 diff 并自动修复此问题。适合于一次提交直接修复问题
- to：只产生 diff 不自动修复此问题。适合于多次提交。最终修复问题提交时使用 fix
- docs：文档（documentation）。
- style：格式（不影响代码运行的变动）【比如说加注释就是这个？】
- refactor：重构（即不是新增功能，也不是修改 bug 的代码变动）。
- perf：优化相关，比如提升性能、体验。
- test：增加测试。
- chore：构建过程或辅助工具的变动。
- revert：回滚到上一个版本。
- merge：代码合并。
- sync：同步主线或分支的 Bug。

参考文章：https://blog.csdn.net/qq_17335549/article/details/128480583

# 搭建二期
## 1、新增axios并封装,还新增了自定义请求错误处理函数，请求类
## 2、封装api列表  apiList   
封装的axios配合api接口使用模板

(1)、先把接口添加进接口列表
```js
export const APIs = {
	GET_SHOPLIST: '/h5/getShopList', // 获取商品列表
}
```
(2)、页面使用
```js
// account.vue
import AxiosRequestError from '@/service/error' // 引入自定义错误处理函数
import $api from '@/service/webRequest' // 封装好的axios请求函数
import { APIs } from '@/service/apiList' // 接口列表

// 二选一即可

// async await 写法
const getShop = async () => {
	try {
		const res = await $api.getShopList()
		console.dir(res, 'res')
	} catch (error: AxiosRequestError) {
		console.dir(error, 'error')
	}
}

// 原生Primise  .then  .catch
$api.get(APIs.GET_SHOPLIST)
	.then(() => {})
	.catch((err: AxiosRequestError) => {
		console.dir(err, 'err')
})

```
(3)、用console.dir可以捕获到详细的错误信息，还能看到我们封装的错误处理函数

data: undefined,  // 接口返回值为undefined

isServerError: false, // 是否为服务器出错

isUnAuthorized: false, // 是否已通过鉴权，也就是常见的登录状态
## <a id="requestURL">3、新增环境变量  优化请求地址</a>
测试和正式环境地址在 global/env.ts 中配置
```js
// 正式环境
export const PROD_ENV = {
	SERVER_URL: 'http://192.168.1.193:8090/', // 服务器地址
	IS_DEV: 'false', // 是否为测试环境
}

// 测试环境
export const DEV_ENV = {
	SERVER_URL: 'http://192.168.1.193:8099/',
	IS_DEV: 'true',
}

/* 	
	isDEV：true为生产环境，false为开发环境
	假设开发环境的域名是 http://127.0.0.1:8099/api 或 https://xxx-test.com
	提示：
	本地如果要将请求地址切换为生产服务器，则将isDEV设置为false，注释掉判断开发环境的代码。代码如下
	const isDEV = false
	// if (typeof window !== 'undefined') {
	// isDEV = process.env.NODE_ENV === 'development' || ['http://192.168.1.193:8099'].includes(window.location.host)
	// }

	准备打包上线，将代码改回来。（开发环境也是这个代码）代码如下   
	let isDEV = true // 默认为开发环境
	if (typeof window !== 'undefined') {
		isDEV = process.env.NODE_ENV === 'development' || ['http://192.168.1.193:8099'].includes(window.location.host)
	}

*/

let isDEV = true // 默认为开发环境，但会根据当前环境动态更换开发或生产
if (typeof window !== 'undefined') {
	isDEV = process.env.NODE_ENV === 'development' || ['http://192.168.1.193:8099'].includes(window.location.host)
}
```
## 4、新增lodash 第三方库  
防抖和节流的使用方法,节流用到时再去查
```js
import { debounce,throttle } from 'lodash-es'

// 它返回一个带防抖的新函数
const debounceLogin = debounce(toLogin, 500)
function toLogin() {
	console.log(111)
}
```

二期以上搭建内容参考的文章：https://blog.csdn.net/qq_17335549/article/details/135022054
## 5、封装按需导入移动端NutUI组件库，plugins/nutui.ts
## 6、解决main.ts 文件引入路径的问题
1、如果引入路径正确，但是提示找不到文件，则删除'XX',重新引入

2、检查vite.config.ts的路径别名配置是否正确,正确代码如下
```js
//新增
	resolve: {
		alias: {
			'@': path.resolve(__dirname, './src'), //把 src 的别名设置为 @
		},
		extensions: ['.js', '.json', '.ts'], // 这些类型的文件后缀的不需要写
	},
```
3、检查tsconfig.json的部分属性配置
```js
  "baseUrl": ".",
  "paths": {"@/*": ["src/*"]},
  "target": "ESNext",
  "module": "ESNext",
   "lib": [
      "ESNext",
      "DOM",
      "DOM.Iterable"
    ],
   "moduleResolution": "node",
    "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue", 
  	],
```
4、检查tsconfig.node.json的部分属性配置
```js
  "compilerOptions": {
	"module": "ESNext",
	"moduleResolution": "node",
  },
  	"include": ["vite.config.ts", "src/**/*.ts", "global/*.ts"]
```
5、在src下新建vite-env.d.ts ,解决ts无法识别引入.vue后缀的文件夹
```js
/// <reference types="vite/client" />
declare module '*.vue' {
	import type { DefineComponent } from 'vue'

	// eslint-disable-next-line @typescript-eslint/no-explicit-any, @typescript-eslint/ban-types

	const component: DefineComponent<{}, {}, any>

	export default component
}

```
每次修改完都要重启项目，或者关闭项目重启VSCode、重启项目。

## 7、Vue3+TS移动端自适应  采用的是postcss-pxtorem 和 lib-flexible
1、安装
```js
pnpm install postcss-pxtorem@5.1.1 --save 
pnpm install lib-flexible --save-dev
```
2、自定义封装自适应代码,src/assets,main.ts引入
```js
// 移动端适配
import 'lib-flexible/flexible.js'
// 引入全局样式
import '@/assets/scss/index.scss'
```
3、vite.config.ts
```js
export default defineConfig({
	optimizeDeps: {
		include: ['lib-flexible/flexible.js'],
	},
})
```

4、组件使用
说明：src/assets下自定义的项目宽尺寸是750，需要375的自行去更改
div像素按UI设计稿的像素即可，页面自适应会自动转换的，无需担心。
## 8、vite性能优化：处理Vite项目首屏加载响应迟缓和二次刷新的问题   
```js
npm i -D vite-plugin-optimize-persist vite-plugin-package-config
```
参考文章：https://www.cnblogs.com/fanqshun/p/16656142.html
# 搭建三期
## 1、pinia store数据长缓存

1、安装
```js
pnpm add pinia-plugin-persistedstate@2.3.0 -S
```

2、pinia引入使用
store/index.ts
```js
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

// 使用持久化存储插件
pinia.use(piniaPluginPersistedstate)
```

3、需要长缓存的store这样配置
```js
store/modules/user.ts

export const useStore = defineStore({
	id: 'user',
	state: () => ({
		name: '很老很老的值',
	}),
	actions: {
		changeName(name: string) {
			this.name = name
		},
	},
	// 核心代码在这里 ↓
	//persist:true //存储整个对象
	// 选择性的长缓存
	persist: {
		storage: localStorage, //default localStorage
		//设置['name'] -->只会将name 这个key进行缓存
		paths: ['name'],
	},
})
```
参考文章： https://download.csdn.net/blog/column/12471199/133916576

## 2、iconfont 阿里巴巴字体图标
配置文章链接： https://blog.csdn.net/Steven_Son/article/details/128149868?csdn_share_tail=%7B%22type%22%3A%22blog%22%2C%22rType%22%3A%22article%22%2C%22rId%22%3A%22128149868%22%2C%22source%22%3A%22Steven_Son%22%7D


## 3、引入免费的阿里巴巴普惠体 2.0字体  
配置文章链接：https://blog.csdn.net/Steven_Son/article/details/128149269