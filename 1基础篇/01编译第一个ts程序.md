# 编译第一个ts程序

## 安装TypeScript

使用 npm 全局安装 TypeScript，这样就可以在任何地方使用其编译器 TSC 

```shell
npm i -g typescript
```

安装完成后，新建目录 `ts_in_action`，用 npm 初始化

```shell
mkdir ts_in_action
cd ts_in_action
# -y 表示用 yes 回答初始化过程中的所有终端提问
npm init -y
```

## 用 TSC 编译

安装好后就可以在命令行中使用 TSC 编译器，查看编译器的相关配置

```shell
tsc -h
```

创建配置文件 `tsconfig.json`

```shell
tsc --init
==========
message TS6071: Successfully created a tsconfig.json file.
```

创建文件结构如下

```
package.json
tsconfig.json
src/
	|-index.ts
```

在index.ts 文件内编写一个段代码：

```typescript
let hello:string = "Hello TypeScript"
```

然后在终端对这个文件进行编译：

```shell
tsc ./src/index.ts
```

编译完成后，发现 src 目录下新生成了一个文件 `index.js`，内容是：

```javascript
var hello = "Hello TypeScript";
```

tsc 把ES6新语法 let 编译为 var ， 并且去掉了 TypeScript 的类型注解`:string`。

## 配建构建工具 webpack 

安装三个包

```shell
npm i webpack webpack-cli webpack-dev-server -D
```

为了增加可维护性，我们可以把工程开发环境、生产环境、公共环境分开的配置分开书写，最后通过插件合并。在配置 webpack 时，我们也遵循这个思路。同时在 src 目录下添加一个模板目录 tpl，用于存放首页模板。得到构建目录如下：

```shell
build/
	|-webpack.config.js			# 所有配置文件的入口
	|-webpack.base.config.js	# 公共环境的配置
	|-webpack.dev.config.js		# 开发环境的配置
	|-webpack.pro.config.js		# 生产环境的配置
src/
	|-tpl/
		|-index.html
```

再安装 webpack 中用到的加载器和插件

```shell
# 安装加载器和编译器
npm i -D typescript ts-loader 	
# 安装各环境所需插件
npm i -D html-webpack-plugin clean-webpack-plugin webpack-merge
```

- `ts-loader`：用于加载 ts 文件。

- `typescript`：用于把 ts 文件编译成 js 文件。

- `html-webpack-plugin`：通过一个模板帮助生成网站首页，并自动把 webpack 的输出文件嵌入到首页中。

- `CleanWebpackPlugin` ：因为有的时候为了避免缓存，我们网站需要在文件后加入hash，这样在多次构建后会产生很多无用的文件。这个插件可以帮助我们在每次成功构建后清空dist目录。
- `webpack-merge`：用于合并配置文件。

至此，webpack配置工作完成。

## 修改npm的脚本

- 在 package.json 中修改入口文件

```json
 {"main": "./src/index.ts"}
```

- 添加开发环境启动脚本

使用 `webpack-dev-server`启动本地服务器，通过`--mode`传入环境参数，最后用`--config`指定配置文件。

```json
{ 
 "scripts":{
     "start":"webpack-dev-server --mode=development --config ./build/webpack.config.js"
 }
}
```

终端启动开发环境构建：

```shell
npm run start
=============
i ｢wds｣: Project is running at http://localhost:8080/
...
Built at: 2020-02-11 14:28:33
     Asset       Size  Chunks             Chunk Names
    app.js    361 KiB    main  [emitted]  main
index.html  317 bytes          [emitted]
Entrypoint main = app.js
...
i ｢wdm｣: Compiled successfully.

```

可以看到`webpack-dev-server`为我们启动了一个跑在8080端口的本地服务器。再次修改 index.ts 文件，然它在页面中显示一段文本：

```javascript
let hello: string = "Hello TypeScript";
document.querySelectorAll(".app")[0].innerHTML = hello;
```

每次更新，server 都会自动帮助我们重构代码，并刷新页面。需要注意的是，webpack 的输出文件 app.js 和模板插件生成的 index.html 都存放在内存中，需要用浏览器才能查看。

- 添加生产环境脚本

生产环境需要用到 webpack ，同样通过 `--mode` 传入环境参数，并用 `--config` 指定配置文件

```json
{ 
 "scripts":{
     "build":"webpack --mode=production --config ./build/webpack.config.js"
 }
}
```

终端启动生产环境构建

```shell
npm run build
=============
Hash: 4ccf59b5b928f628cd4b
Version: webpack 4.41.5
Time: 5698ms
Built at: 2020-02-11 15:01:12
     Asset        Size  Chunks             Chunk Names
    app.js  1010 bytes       0  [emitted]  main
index.html   317 bytes          [emitted]
Entrypoint main = app.js
[0] ./src/index.ts 105 bytes {0} [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
    [0] ./node_modules/html-webpack-plugin/lib/loader.js!./src/tpl/index.html 483 bytes 
{0} [built]
    [2] (webpack)/buildin/global.js 472 bytes {0} [built]
    [3] (webpack)/buildin/module.js 497 bytes {0} [built]
        + 1 hidden module
```

构建完成后，可以看到新生成了一个 dist 目录结构如下：

```
dist/
	|-app.js
	|-index.html
```

app.js 是 webpack 压缩好的输出文件，并且已经自动嵌入到 index.html中。