# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

# 背景
最近遇到一个问题，把一个 6 个月前在git ci上重新打包后，发现打包后的文件名的 hash 值跟之前不一样了，明明代码也没改啊，怎么回事？

# 分析1

分析一下，肯定是 webpack 打包的问题，我们的项目中用的是 contenthash，是不是换成 chunkhash 就好了呢？

## hash, chunkhash 和 contenthash 的区别

网上搜了一下 chunkhash 和 contenthash 的区别，写的挺模糊的。我们动手试验一下：

### contenthash

创建一个 create-react-app，yarn eject 后，可以看到 config/webpack.config.js 中，默认配置的 hash 是 contenthash

```js
output: {
  // The build folder.
  path: isEnvProduction ? paths.appBuild : undefined,
  // Add /* filename */ comments to generated require()s in the output.
  pathinfo: isEnvDevelopment,
  // There will be one main bundle, and one file per asynchronous chunk.
  // In development, it does not produce real files.
  filename: isEnvProduction
    ? 'static/js/[name].[contenthash:8].js'
    : isEnvDevelopment && 'static/js/bundle.js',
  // TODO: remove this when upgrading to webpack 5
  futureEmitAssets: true,
  // There are also additional JS chunk files if you use code splitting.
  chunkFilename: isEnvProduction
    ? 'static/js/[name].[contenthash:8].chunk.js'
    : isEnvDevelopment && 'static/js/[name].chunk.js',
  // webpack uses `publicPath` to determine where the app is being served from.
  // It requires a trailing slash, or the file assets will get an incorrect path.
  // We inferred the "public path" (such as / or /my-project) from homepage.
  publicPath: paths.publicUrlOrPath,
```

1. 执行 yarn build 进行打包，可以得到打包后的文件名。main.chunk.js 和 main.chunk.css 的 hash 值不同：

```
  41.21 KB  build/static/js/2.0b773d50.chunk.js
  1.4 KB    build/static/js/3.113de9f7.chunk.js
  1.18 KB   build/static/js/runtime-main.1f83336f.js
  603 B     build/static/js/main.de68dd7b.chunk.js
  531 B     build/static/css/main.8c8b27cf.chunk.css
```

2. 再次打包，在内容无修改的情况下，打包后的文件名是不变的：

```
  41.21 KB  build/static/js/2.0b773d50.chunk.js
  1.4 KB    build/static/js/3.113de9f7.chunk.js
  1.18 KB   build/static/js/runtime-main.1f83336f.js
  603 B     build/static/js/main.de68dd7b.chunk.js
  531 B     build/static/css/main.8c8b27cf.chunk.css
```

3. 随意修改一下 App.js 的内容，然后再次打包，main.xxx.chunk.js 的 hash 值变了，但是 main.xxx.chunk.css 的 hash 没有变化：

```
  41.21 KB      build/static/js/2.0b773d50.chunk.js
  1.4 KB        build/static/js/3.113de9f7.chunk.js
  1.18 KB       build/static/js/runtime-main.1f83336f.js
  604 B (+1 B)  build/static/js/main.65d37fe9.chunk.js
  531 B         build/static/css/main.8c8b27cf.chunk.css
```

4. 将 App.js 的内容复原，再次打包，main.xxx.chunk.js 又与第 1 步、第 2 步的 hash 值相同了：

```
  41.21 KB      build/static/js/2.0b773d50.chunk.js
  1.4 KB        build/static/js/3.113de9f7.chunk.js
  1.18 KB       build/static/js/runtime-main.1f83336f.js
  603 B (-1 B)  build/static/js/main.de68dd7b.chunk.js
  531 B         build/static/css/main.8c8b27cf.chunk.css
```

5. 随意修改一下 App.css 的内容，然后再次打包，main.xxx.chunk.css 的 hash 值变了，但是 main.xxx.chunk.js 的 hash 没有变化：

```
  41.21 KB      build/static/js/2.0b773d50.chunk.js
  1.4 KB        build/static/js/3.113de9f7.chunk.js
  1.18 KB       build/static/js/runtime-main.1f83336f.js
  603 B         build/static/js/main.de68dd7b.chunk.js
  532 B (+1 B)  build/static/css/main.3b619909.chunk.css
```

6. 将 App.css 的内容复原，再次打包，main.xxx.chunk.css 又与第 1 步、第 2 步的 hash 值相同了：

```
  41.21 KB      build/static/js/2.0b773d50.chunk.js
  1.4 KB        build/static/js/3.113de9f7.chunk.js
  1.18 KB       build/static/js/runtime-main.1f83336f.js
  603 B (-1 B)  build/static/js/main.de68dd7b.chunk.js
  531 B         build/static/css/main.8c8b27cf.chunk.css
```

### 将 webpack.config.js 中的 contenthash 改为 chunkhash，重复上面 6 个步骤

1. 执行 yarn build 进行打包，可以得到打包后的文件名。main.chunk.js 和 main.chunk.css 的 hash 值是相同的：

```
  41.21 KB       build/static/js/2.8308284b.chunk.js
  1.4 KB (+1 B)  build/static/js/3.69da51f9.chunk.js
  1.18 KB        build/static/js/runtime-main.5c8ad717.js
  604 B (+1 B)   build/static/js/main.23b897ad.chunk.js
  531 B          build/static/css/main.23b897ad.chunk.css
```

2. 再次打包，在内容无修改的情况下，打包后的文件名是不变的：

```
  41.21 KB  build/static/js/2.8308284b.chunk.js
  1.4 KB    build/static/js/3.69da51f9.chunk.js
  1.18 KB   build/static/js/runtime-main.5c8ad717.js
  604 B     build/static/js/main.23b897ad.chunk.js
  531 B     build/static/css/main.23b897ad.chunk.css
```

3. 随意修改一下 App.js 的内容，然后再次打包，main.xxx.chunk.js 和 main.xxx.chunk.css 的 hash 值都变了：

```
  41.21 KB      build/static/js/2.8308284b.chunk.js
  1.4 KB        build/static/js/3.69da51f9.chunk.js
  1.18 KB       build/static/js/runtime-main.5c8ad717.js
  602 B (-2 B)  build/static/js/main.dedc4bbe.chunk.js
  529 B (-2 B)  build/static/css/main.dedc4bbe.chunk.css
```

4. 将 App.js 的内容复原，再次打包，main.xxx.chunk.js 和 main.xxx.chunk.css 又与第 1 步、第 2 步的 hash 值相同了：

```
  41.21 KB      build/static/js/2.8308284b.chunk.js
  1.4 KB        build/static/js/3.69da51f9.chunk.js
  1.18 KB       build/static/js/runtime-main.5c8ad717.js
  604 B (+2 B)  build/static/js/main.23b897ad.chunk.js
  531 B (+2 B)  build/static/css/main.23b897ad.chunk.css
```

5. 随意修改一下 App.css 的内容，然后再次打包，main.xxx.chunk.js 和 main.xxx.chunk.css 的 hash 值都变了：

```
  41.21 KB      build/static/js/2.8308284b.chunk.js
  1.4 KB        build/static/js/3.69da51f9.chunk.js
  1.18 KB       build/static/js/runtime-main.5c8ad717.js
  603 B (-1 B)  build/static/js/main.acc05875.chunk.js
  531 B         build/static/css/main.acc05875.chunk.css
```

6. 将 App.css 的内容复原，再次打包，main.xxx.chunk.css 又与第 1 步、第 2 步的 hash 值相同了：

```
  41.21 KB      build/static/js/2.8308284b.chunk.js
  1.4 KB        build/static/js/3.69da51f9.chunk.js
  1.18 KB       build/static/js/runtime-main.5c8ad717.js
  604 B (+1 B)  build/static/js/main.23b897ad.chunk.js
  531 B         build/static/css/main.23b897ad.chunk.css
```

### 将 webpack.config.js 中的 contenthash 改为 hash，重复上面 6 个步骤

1. 执行 yarn build 进行打包，可以得到打包后的文件名。所有文件的 hash 值都是相同的：

```
  41.21 KB (-2 B)  build/static/js/2.c17725a0.chunk.js
  1.4 KB           build/static/js/3.c17725a0.chunk.js
  1.16 KB (-16 B)  build/static/js/runtime-main.c17725a0.js
  603 B (-1 B)     build/static/js/main.c17725a0.chunk.js
  531 B            build/static/css/main.c17725a0.chunk.css
```

2. 再次打包，在内容无修改的情况下，打包后的文件名是不变的：

```
  41.21 KB  build/static/js/2.c17725a0.chunk.js
  1.4 KB    build/static/js/3.c17725a0.chunk.js
  1.16 KB   build/static/js/runtime-main.c17725a0.js
  603 B     build/static/js/main.c17725a0.chunk.js
  531 B     build/static/css/main.c17725a0.chunk.css
```

3. 随意修改一下 App.js 的内容，然后再次打包，所有文件的 hash 值都变了：

```
  41.21 KB (+2 B)  build/static/js/2.375a482c.chunk.js
  1.4 KB (+1 B)    build/static/js/3.375a482c.chunk.js
  1.16 KB (+1 B)   build/static/js/runtime-main.375a482c.js
  604 B (+1 B)     build/static/js/main.375a482c.chunk.js
  531 B            build/static/css/main.375a482c.chunk.css
```

4. 将 App.js 的内容复原，再次打包，所有文件的 hash 值又与第 1 步、第 2 步的 hash 值相同了：

```
  41.21 KB (-2 B)  build/static/js/2.c17725a0.chunk.js
  1.4 KB (-1 B)    build/static/js/3.c17725a0.chunk.js
  1.16 KB (-1 B)   build/static/js/runtime-main.c17725a0.js
  603 B (-1 B)     build/static/js/main.c17725a0.chunk.js
  531 B            build/static/css/main.c17725a0.chunk.css
```

5. 随意修改一下 App.css 的内容，然后再次打包，所有文件的 hash 值都变了：

```
  41.21 KB (-1 B)  build/static/js/2.e437e80e.chunk.js
  1.4 KB (-1 B)    build/static/js/3.e437e80e.chunk.js
  1.16 KB (-1 B)   build/static/js/runtime-main.e437e80e.js
  602 B (-1 B)     build/static/js/main.e437e80e.chunk.js
  531 B            build/static/css/main.e437e80e.chunk.css
```

6. 将 App.css 的内容复原，再次打包，所有文件的 hash 值又与第 1 步、第 2 步的 hash 值相同了：

```
  41.21 KB (+1 B)  build/static/js/2.c17725a0.chunk.js
  1.4 KB (+1 B)    build/static/js/3.c17725a0.chunk.js
  1.16 KB (+1 B)   build/static/js/runtime-main.c17725a0.js
  603 B (+1 B)     build/static/js/main.c17725a0.chunk.js
  531 B            build/static/css/main.c17725a0.chunk.css
```

## 总结一下

1. 三者的区别：
- hash 计算与整个项目的构建相关；

整个项目公用一个 hash 值。

- chunkhash 计算与同一 chunk 内容相关；

同一个 chunk 的 js 和 css 文件，公用一个 hash 值。任意一个文件内容改变，公用 hash 值改变。

- contenthash 计算与文件内容本身相关。

当文件内容不变时，hash 值不变。

2. 试验结论：

- 当代码内容不变的时候，contenthash和chunkhash都不会造成打包后文件hash值变化。
- 当代码变化时，contenthash比chunkhash引发变化的hash值更少。
- 所以，能用contenthash就用contenthash。


# 分析2

拉下来的代码是没有变化的，但项目hash值确实变了，那会是打包机器不同造成的吗？

经过试验发现：

- 打包的机器不同，打包后的hash值是不同的
- 同一台机器，在不同的文件位置，打包后的hash值也是不同的

但我这个问题是在同一个git ci上执行的打包，机器肯定是相同的。

# 分析3

如果都没变化，那会不会是依赖的npm包变了呢？

## 如果项目代码没变，但引用的 npm 包的版本变了，打包后的文件 hash 会变化吗？

我们来动手试验一下，webpack 的 hash 设置为 contenthash。

### 在 package.json 中引入 query-string，但不 import

1. yarn add query-string@6.13.8，执行打包：

```
  41.22 KB  build/static/js/2.0b773d50.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.113de9f7.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.b370ca9f.filename.js
  610 B     build/static/js/main.de68dd7b.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

2. 只改变 package.json 中版本号，但不重新 install，打包后 hash 值不变。(废话，执行代码内容都没变)

```
  41.22 KB  build/static/js/2.0b773d50.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.113de9f7.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.b370ca9f.filename.js
  610 B     build/static/js/main.de68dd7b.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

3. yarn add query-string@6.12.0，再次打包。(肯定不变啊，代码又没引用这个包)

```
  41.22 KB  build/static/js/2.0b773d50.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.113de9f7.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.b370ca9f.filename.js
  610 B     build/static/js/main.de68dd7b.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

### 在 App.js 中 import query-string，但不执行任何方法

1. yarn add query-string@6.13.8，执行打包：

```
  43.79 KB  build/static/js/2.1874ca7b.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.8361a48e.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.21632db2.filename.js
  613 B     build/static/js/main.83b2031e.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

2. yarn add query-string@6.12.0，再次打包，2.xxx.chunk.chunkFilename.js 的 hash 值变了

```
  43.66 KB  build/static/js/2.02dac1eb.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.8361a48e.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.21632db2.filename.js
  613 B     build/static/js/main.83b2031e.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

### 在 App.js 中 import query-string，执行方法

```js
const parsed = queryString.parse(window.location.search)
console.log(parsed)
```

1. yarn add query-string@6.13.8，执行打包：

```
  43.78 KB  build/static/js/2.5b8a76de.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.8361a48e.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.21632db2.filename.js
  653 B     build/static/js/main.f3715165.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

2. yarn add query-string@6.12.0，再次打包

```
  43.65 KB  build/static/js/2.6d24736a.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.8361a48e.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.21632db2.filename.js
  653 B     build/static/js/main.f3715165.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

### 结论：

- 如果 npm 包只是在 package.json 中引入，但没有在代码里引入，相当于没有用这个 npm。该 npm 的版本变化不会引发打包后代码的 hash 值变化。（不用的包尽量删除）
- npm 包在代码里 import 引入后，该 npm 的版本变化会影响打包后的代码的 hash 值，即便只是 import，并没有真正使用。（这样冗余的代码，记得早点删除。可以使用 eslint 帮忙）
- 提交到 git 的代码，要带上 yarn.lock 或者 package-lock.json(二者选你常用的那个即可，不要一起提交到 git)，便于依赖包管理。

这样子，就不会引发一段时间后，明明代码没有改变，打包后的文件 hash 值却变了的问题了。

## 补充知识

- chunk 是什么？

存在依赖关系的模块，在打包时被封装为一个 chunk。webpack 会从入口文件开始检索，并将具有依赖关系的模块生成一棵依赖树，最终得到一个 chunk。
如果 webpakc 定义了多个入口文件，就会得到多个 chunk 文件。

- webpack 中 filename 和 chunkFilename 的区别

  - filename 指列在 entry 中，打包后输出的文件的名称。
  - chunkFilename 指未列在 entry 中，却又需要被打包出来的文件的名称。

改一下 webpack 的名字规则，让大家看起来更清晰：

```
  41.22 KB  build/static/js/2.0b773d50.chunk.chunkFilename.js
  1.41 KB   build/static/js/3.113de9f7.chunk.chunkFilename.js
  1.19 KB   build/static/js/runtime-main.b370ca9f.filename.js
  610 B     build/static/js/main.de68dd7b.chunk.chunkFilename.js
  537 B     build/static/css/main.8c8b27cf.chunk.chunFilename.css
```

- MiniCssExtractPlugin

本插件会将 CSS 提取到单独的文件中，为每个包含 CSS 的 JS 文件创建一个 CSS 文件，并且支持 CSS 和 SourceMaps 的按需加载。

- SplitChunksPlugin

本插件自动拆分代码到不同的 chunk 文件中。参考[官方文档](https://webpack.docschina.org/plugins/split-chunks-plugin/)

```
默认情况下，它仅影响按需块，因为更改初始块会影响HTML文件应包含的脚本标签以运行项目。

webpack将根据以下条件自动分割块：

可以共享新块，或者模块来自node_modules文件夹
新的块将大于20kb（在min + gz之前）
按需加载块时并行请求的最大数量将小于或等于30
初始页面加载时并行请求的最大数量将小于或等于30
当试图满足最后两个条件时，最好使用较大的块。
```

- runtimeChunk

单独分离出webpack的一些运行文件

## Available Scripts

In the project directory, you can run:

### `yarn start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in the browser.

The page will reload if you make edits.\
You will also see any lint errors in the console.

### `yarn test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `yarn build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `yarn eject`

**Note: this is a one-way operation. Once you `eject`, you can’t go back!**

If you aren’t satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you’re on your own.

You don’t have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn’t feel obligated to use this feature. However we understand that this tool wouldn’t be useful if you couldn’t customize it when you are ready for it.
