---
title: 使用rollup对koa后端项目进行打包
cover: /images/123.png
date: '2024-02-19 10:14:30'
updated: '2024-08-18 20:14:54'
---
### 先安装依赖
```
npm i rollup rollup-plugin-delete rollup-plugin-terser @babel/core @babel/plugin-transform-runtime
@babel/preset-env @rollup/plugin-babel @rollup/plugin-commonjs @rollup/plugin-json --save-dev

npm i @babel/runtime --save
```
### 创建打包的配置文件
```
//在根目录下创建
touch rollup-build.js
```
```jsx
// /rollup-build.jsconst fs = require("fs");const rollup = require("rollup");const { babel, getBabelOutputPlugin } = require("@rollup/plugin-babel");const del = require("rollup-plugin-delete");const json = require("@rollup/plugin-json");const commonjs = require("@rollup/plugin-commonjs");const { terser } = require("rollup-plugin-terser");// 获取根目录的'package.json'const packageJSON = require("./package.json");// 读取 生成模式 下需要的依赖包const packageJSONForProduction = {  name: packageJSON.name,  dependencies: packageJSON.dependencies,};const inputOptions = {  input: "./app.js",  plugins: [    // 打包前先清空输出文件夹    del({ targets: "./dist/*" }),    // babel 相关的配置, 主要是做兼容    getBabelOutputPlugin({      presets: [["@babel/preset-env", { targets: { node: "current" } }]],      plugins: [["@babel/plugin-transform-runtime", { useESModules: false }]],    }),    babel({ babelHelpers: "bundled", exclude: "node_modules/**" }),    // 这里是把入口文件(app.js)以外的业务代码也进行打包(require进来的文件)    json(),    commonjs(),    // 代码的压缩或混淆    terser(),  ],};const outputOptions = { dir: "./dist", format: "cjs" };async function build() {  // create a bundle  const bundle = await rollup.rollup(inputOptions);  // generate code and a sourcemap  // const { code, map } = await bundle.generate(outputOptions);  // or write the bundle to disk  await bundle.write(outputOptions);  // 生成生产模式的 package.json, 在服务器上使用  const writeStream = fs.createWriteStream("./dist/package.json");  writeStream.write(JSON.stringify(packageJSONForProduction));}build();
```
在package.json配置构建命令
```json
// /package.json{  "scripts": {    "build": "node ./rollup-build.js"  }}
```
执行npm run build或者yarn build之后, dist文件夹就是打包出来的文件. 里面有业务代码和package.json. 拷贝bin和public目录到dist中，并在package.json中添加下列代码
```json
"scripts": {    "dev": "node bin/www"  }
```
把dist文件夹上传到线上服务器，并在服务器安装好node环境，在dist中执行npm install 安装依赖，就可以执行项目。
### pm2进程管理
如果不想退出ssl就终止服务，就需要使用守卫进程保持启动node服务，这里采用pm2。
在服务器全局安装
```
npm install -g pm2
```
pm2 的常用命令
```
pm2 start ./bin/www #启动koa2项目
pm2 start ./bin/www --watch pm2自动重启

#www是项目名
pm2 list           #查看所用已启动项目
pm2 start          #启动
pm2 restart www    #重启
pm2 stop www       #停止
pm2 delete www     #删除
```
