layout: post
title: ng2 build
date: 2016-12-12 10:03:59
tags:ng2
---
angular2 编译基础
================
## 起步
按照angular 官方社区提供的demo 运行，[quick_start](https://github.com/angular/quickstart),该demo运行方式为JIT编译方式，利用了模块加载system 来加载各个模块的代码。
## 编译方式
- GIT (Just-in-Time)及时编译，开发的时候加载比较快，缺点：引入的文件过多，不利于线上使用
- AOT (Ahead of Time)预编译，有点： 线上可以达成一个bundle 文件，较少文件请求，缺点，开发时候编译较慢. 不过我们应该去使用aot方式来保证错误更早的时候暴露出来及渲染速度。

起初按照按照官方的quick start demo 做例子，但是经常出现一个地方出错，导致编译报错，并且编译报错无法定位到具体的问题原因。于是对angular 的编译进行学习，主要讲解编译的过程及编译过程中依赖的一些模块及各个模块的作用

### tsconfig.json
tsconfig 主要typescript 编译的配置文件，编译ts 文件

```
{
  "compilerOptions": {
    "target": "es5",编译为js语法，这些是指把ts 编译为es5代码
    "module": "commonjs", 模块系统为commonjs
    "moduleResolution": "node",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "lib": [ "es2015", "dom" ],
    "noImplicitAny": true,
    "suppressImplicitAnyIndexErrors": true， 
  },
  "exclude": [
    "node_modules/*",
    "**/*-aot.ts"
  ]
}

```
noImplicitAny和suppressImplicitAnyIndexErrors 告诉tsc 编译在无法推测变量类型的报错机制，tsconfig 有很多配置的，具体的可以去查看[config](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)

### 如何进行aot 及tree shaking
首先需要使用@angular/compiler-cli 这个模块进行ngc 编译，ngc 是tsc 的一个高仿品
compilerOptions部分只修改了一个属性：**把module设置为es2015。 主要是为后面tree shaking 使用，tree shaking 只能应用于es6模块代码.
angularCompilerOptions,genDir属性告诉ngc编译器把编译结果保存在新的aot目录下,然后tsc 把总的编译结果放在outDir目录下
aot config 

```
{
  "compilerOptions": {
    "target": "es5",
    "module": "es2015",
    "moduleResolution": "node",
    "sourceMap": false,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "lib": ["es2015", "dom"],
    "noImplicitAny": true,
    "suppressImplicitAnyIndexErrors": true,
    "outDir": "./tsOut",
    "typeRoots": [
      "../node_modules/@types",
      "../node_modules"
    ]
  },
  "files": [
    "app/app.module.ts",
    "app/main.ts"
  ],
  "angularCompilerOptions": {
   "genDir": "aot",
   "skipMetadataEmit" : true
 }
}

```

###  rollup config
Rollup支持多种输出格式。因为我们是要在浏览器中使用，需要使用立即执行函数表达式(IIFE)

```
import rollup      from 'rollup'
import nodeResolve from 'rollup-plugin-node-resolve'
import commonjs    from 'rollup-plugin-commonjs';
import uglify      from 'rollup-plugin-uglify'

export default {
  entry: 'tsOut/app/main.js',
  dest: 'dist/build.js', // output a single application bundle
  sourceMap: false,
  format: 'iife',
  plugins: [
      nodeResolve({jsnext: true, module: true}),
      commonjs({
        include: 'node_modules/rxjs/**',
      }),
      uglify()
  ]
}

``` 

### 依赖库及各模块作用
- es6-shim: angular2依赖了大量ES2015的特性，这可能导致一些不支持ES2015特性的浏览器无法运行angular2程序(例如：老版本IE)。所以需要该shim来保证老浏览器的正确性
- reflect-metadata: angular2允许开发者使用Decorator，这使得程序具备更好的可读性。无奈Decorator是ES2016里的提案，需要reflect-metadata提供反射API才能使用
- rxjs: 一个Reactive Programming的JavaScript实现。这里对她的依赖是因为angular2支持多种数据更新模式，比如：flux、Rx.js
- zone.js: 用来对异步任务提供Hooks支持，使得在异步任务运行之前/之后做额外操作成为可能。在angular2里的主要应用场景是提高脏检查效率、降低性能损耗
- lite-server: 一个轻量级的静态服务器，本章节我们就用它启动程序
- concurrently: 这是一个可以让多个阻塞命令同时执行、管理的工具。我们将在后面用到
- tsd: typescript定义文件管理系统，由于angular2依赖ES2015的诸多特性，譬如：Promise等，所以需要这些API的支持以及typescript定义
- typescript: tsc 命令
- compiler-cli包中提供的ngc编译器来,ngc需要自己的带有AoT专用设置的tsconfig.json

## 例子
- 详细demo 可去这里[查看](https://github.com/shibiaoz/ng2)
- 编译，构建有比较成熟的脚手架,[link](https://cli.angular.io/)

## ToDo
- gulp + roolup
