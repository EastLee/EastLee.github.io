---
title: ESL模块处理过程
date: 2016-05-10 14:07:23
categories: 前端模块化
tags: esl 
---

## ESL是什么

[ESL](https://github.com/EastLee/esl)是一个浏览器端、符合AMD的标准加载器，适合于现代web浏览器端应用的入口与模块管理。
相比于RequireJS拥有以下优点：

* 体积更小
* 性能更高
* 更健壮
* 不支持非浏览器端使用
* 依赖模块用时定义


## ESL模块加载详解
<!--more-->
esl.js加载模块过程如下：
{% asset_img esl-run-route.png esl run route %}

1、require引入模块的入口，调用nativeRequire，在这个函数中通过tryFinishRequire绑定require的callback，并且把tryFinishRequire作为监听函数绑在入口模块上；

2、nativeRequire通过loadModule加载入口js文件即define文件，并且通过onload绑定一个监听函数loadedListener；

3、define函数生产出模块的id、依赖模块和callback；

4、等到define文件执行完成，loadedListener会通过completePreDefine将模块数据加入到modModules中；

5、最后通过modAnalyse分析依赖模块的属性以及处理url，其中会调用modAutoInvoke（核心函数）处理最后的结果，如果依赖的模块还存在没有加载的，再次统一放入nativeRequire中，循环上面的过程！

## 核心函数modAutoInvoke分析

```js
//核心函数 递归设置所有模块状态为3，递归执行所有invokeFactory
  function modAutoInvoke() {
    for (var id in autoDefineModules) {
      modUpdatePreparedState(id); //一次性设置有依赖关系的模块为状态3
      modTryInvokeFactory(id);
    }
  }
```

* autoDefineModules数组存放着作为入口的模块；
* modUpdatePreparedState的作用：采用递归遍历所有模块，检测依赖的模块是否都是状态3，如果是，autoDefineModules中对应的模块状态可以设置为3，如果不是，则不能设置为3；
* modTryInvokeFactory的作用是在autoDefineModules中模块状态达到3时，执行InvokeFactory；

```js
      //invokeFactory中的两段代码
      each(module.factoryDeps, function(dep) {
          var depId = dep.absId;
          if (!BUILDIN_MODULE[depId]) {
            modTryInvokeFactory(depId); //递归执行依赖模块的invokeFactory
            if (!modIs(depId, MODULE_DEFINED)) { //所有依赖的模块必须达到MODULE_DEFINED
              factoryReady = 0;
              return false;
            }
          }
          factoryDeps.push(depId); //达到MODULE_DEFINED状态的模块被推入factoryDeps中
        }
      );

      //所有依赖的模块必须达到MODULE_DEFINED就可以执行下面的代码
      if (factoryReady) {
        try {
          var args = modGetModulesExports( //依赖模块的输出
            factoryDeps, {
              require: module.require,
              exports: module.exports,
              module: module
            }
          );
          var factory = module.factory;
          var exports = typeof factory === 'function' ? factory.apply(global, args) : factory;
          if (exports != null) {
            module.exports = exports;
          }
          module.invokeFactory = null;
        } catch (ex) {}
        modDefined(id); //设置为状态4,执行监听函数
      }
    }
  }
```

以上是invokeFactory中的两段代码，这个函数的作用就是输出模块的执行结果，但是在输出自身之前必须通过modTryInvokeFactory递归所有依赖的模块，这些依赖的模块也都必须输出执行结果，赋值在exports中，供父模块调用！

## 核心函数modAutoInvoke调用

在整个源文件中，调用modAutoInvoke主要有两处，一个是modAnalyse最后执行，在当前模块分析完毕，状态达到2时调用，另外一处是nativeRequire中！核心思想就是在每处理一个模块，都要从入口模块递归依赖的所有模块，以达到一旦所有模块准备就绪，就可以执行require的callback函数！

> [demo](https://github.com/EastLee/JsModule/tree/master/esl)

