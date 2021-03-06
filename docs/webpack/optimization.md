# optimization
optimization 是webpack中非常重要的一个属性。通常情况下webpack的默认配置不能适用所有的项目。我们需要了解该配置下的一些重要属性，针对不同项目做出优化。

## chunkIds（moduleIds）
根据官方文档可知，推荐使用下述两值。
- 开发模式下默认使用 `named`便于调试。
<img :src="$withBase('/images/webpack_config_chunkIds_1.png')" alt="webpack_config_chunkIds">

- 生产模式下默认使用`deterministic`利于缓存。
<img :src="$withBase('/images/webpack_config_chunkIds_2.png')" alt="webpack_config_chunkIds">

## concatenateModules
让webpack找到模块之间的依赖联系，把模块合并的同一个函数中，缩小代码体积。

函数关系如下：

<img :src="$withBase('/images/concatenateModules.png')" alt="concatenateModules">

开启`concatenateModules`

<img :src="$withBase('/images/concatenateModules_true.png')" alt="concatenateModules">

关闭`concatenateModules`

<img :src="$withBase('/images/concatenateModules_false.png')" alt="concatenateModules">

## removeAvailableModules
改配置会降低webpack性能，在后续正式版本会默认关闭。
## runtimeChunk
把运行时的代码提取出来生成一个公共的文件。但是任何一个运行时改动都会导致这个共文件hash闭环，导致缓存失效，所以建议关闭。
## sideEffects
待定
## splitChunks
配置webpack如何执行拆包策略。
### chunks

决定那些包会被拆分优化。

1. 设置为`initial`：

- 在入口文件`index.js`中引入`lodash`，`lodash`被拆分。

```js
// index.js
import _ from 'lodash'
console.log(_)
```

<img :src="$withBase('/images/splitChunks_1.png')" alt="splitChunks_1">

- 在`index.js`中同步引入`data1.js`，`main.js`体积增加，`data1.js`未被拆分。

```js
// index.js
import _ from 'lodash'
console.log(_)
import data1 from "./data1"
console.log(data1)
```

<img :src="$withBase('/images/splitChunks_2.png')" alt="splitChunks_2">

- 在`index.js`中异步引入`data1.js`。可以看出`data1.js`中的数据被拆分。因为是异步引入，所以在解析`data1.js`时，作为新的入口文件进行拆分。

<img :src="$withBase('/images/splitChunks_3.png')" alt="splitChunks_3">

- 在`index.js`中异步引入`a.js`和`b.js`,`a.js`和`b.js`分别同步引入`data1.js`。可以看出拆出了两个文件`src_a_js.js`和`src_b_js.js`,这两个文件均包含`data1.js`的数据，出现了重复代码。

```js
// index.js
import _ from 'lodash'
console.log(_)
import('./a.js')
import('./b.js')
```

```js
// a.js b.js
import data1 from "./data1"
console.log(data1)
```

<img :src="$withBase('/images/splitChunks_4.png')" alt="splitChunks_4">


2. 设置为`async`可以看到，`lodash`未被拆分，`data1.js`从两个异步引入的文件中被拆分。


<img :src="$withBase('/images/splitChunks_5.png')" alt="splitChunks_5">


3. 设置为`all`。可以看到`loadsh`被拆分，`data1.js`从两个异步引入的文件中被拆分。

<img :src="$withBase('/images/splitChunks_6.png')" alt="splitChunks_6">

4. 总结：
- `initial` 可以把入口文件中的node_modules代码提取出来，不能提取直接引入入口文件中的js。可以减少vendor文件的大小，但是会出现重复的代码。
- `async` 可以把所有异步的代码提取出来。
- `all` 同时具备initial和async的特性。
- 通常使用`all`会有更好的拆分效果
