---
title: ESL之config配置
date: 2016-05-10 17:09:54
categories: 前端模块化
tags: esl
---

## config默认配置

```js
var requireConf = {
    baseUrl: './',
    paths: {},
    config: {},
    map: {},
    packages: [],
    waitSeconds: 0,
    noRequests: {},
    urlArgs: {}
  };
```

所有的配置初始化在createConfIndex函数中完成
<!--more-->
## baseUrl

在整个源码中只有以下两处出现过：
createConfIndex对baseUrl的处理
```js
//处理末尾有或者没有‘/’
requireConf.baseUrl = requireConf.baseUrl.replace(/\/$/, '') + '/';
```
对于baseUrl，末尾可以有“/”也可以没有！

在toUrl函数中拼接模块的url
```js
// 相对路径时，附加baseUrl(url前面不能有"/")
    if (!/^([a-z]{2,10}:\/)?\//i.test(url)) {
      url = requireConf.baseUrl + url;
    }
```
在加载模块时，会把所有的要通过script标签加载define文件的url前面拼接baseUrl

## paths

配置

```js
require.config({
      baseUrl: "http://j2.58cdn.com.cn/m58/njs/",
      paths: {
        test: "pkg/house"
      }
    });
    require(['test/ershoufang_list'], function() {
      console.log('pkg ershoufang_list callback')
    });
```

createConfIndex对paths的处理
```js
pathsIndex = createKVSortedIndex(requireConf.paths);
//处理结果
pathsIndex = {
    k: "test",
    reg: /^test(\/|$)/, //前缀匹配
    v: "pkg/house"
}
```

toUrl的处理
```js
    // paths处理和匹配
    var isPathMap;
    // 将url中的key用pathsIndex中value替换
    indexRetrieve(id, pathsIndex, function(value, key) {
      url = url.replace(key, value);
      isPathMap = 1;
    });

    // 如果pathsIndex没有匹配，就用packagesIndex处理url
    if (!isPathMap) {
      indexRetrieve(id, packagesIndex, function(value, key, item) {
        url = url.replace(item.name, item.location);
      });
    }
```
相当于在url中用paths的key作为模块id的一部分，而其value才是url真实的一部分，并且packages和paths是互斥的！

## packages

配置
```js
require.config({
      baseUrl: "http://j2.58cdn.com.cn/m58/njs/",
      packages: [{
        name: "test",
        location: "pkg/house",
        main: "ershoufang_list",
      }]
    });
    require(['test'], function() {
      console.log('pkg ershoufang_list callback')
    });
```

createConfIndex对packages的处理
```js
packagesIndex = [];
    each(requireConf.packages, function(packageConf) {
        var pkg = packageConf;
        if (typeof packageConf === 'string') {
          pkg = {
            name: packageConf.split('/')[0],
            location: packageConf,
            main: 'main'
          };
        }

        pkg.location = pkg.location || pkg.name;
        pkg.main = (pkg.main || 'main').replace(/\.js$/i, '');
        pkg.reg = createPrefixRegexp(pkg.name); //前缀匹配name
        packagesIndex.push(pkg);
      }
    );
    packagesIndex.sort(descSorterByKOrName);
```
packagesIndex一般会有name、location、main和reg这四个属性

在normalize中
```js
//packagesIndex中某项的name属性与moduleId相同，就将main属性拼接在moduleId之后
    each(packagesIndex, function(packageConf) {
        var name = packageConf.name;
        if (name === moduleId) {
          moduleId = name + '/' + packageConf.main;
          return false;
        }
      }
    );
```
packagesIndex的name属性与moduleId匹配时，把main属性拼接上

toUrl的处理
```js
    // paths处理和匹配
    var isPathMap;
    // 将url中的key用pathsIndex中value替换
    indexRetrieve(id, pathsIndex, function(value, key) {
      url = url.replace(key, value);
      isPathMap = 1;
    });

    // 如果pathsIndex没有匹配，就用packagesIndex处理url
    if (!isPathMap) {
      indexRetrieve(id, packagesIndex, function(value, key, item) {
        url = url.replace(item.name, item.location);
      });
    }
```
如果匹配，就用packagesIndex的location属性替换url中的name部分

## map

配置
```js
require.config({
      baseUrl: "http://j2.58cdn.com.cn/m58/njs/",
      map: {
        "pkg/house/ershoufang_list": {
          "test": "mod/common"
        }
      }
    });
    require(['pkg/house/ershoufang_list'], function() {
      console.log('pkg ershoufang_list callback')
    });
```

createConfIndex中的处理
```js
mappingIdIndex = createKVSortedIndex(requireConf.map, 1);
    each(mappingIdIndex, function(item) {
        item.v = createKVSortedIndex(item.v);
      }
    );
```

normalize中的处理
```js
// mappingIdIndex=[{reg1:xx,k1:xx,v1:{reg2:xx,k2:xx,v2:xx}}]，reg1匹配baseId，reg2匹配moduleId,将moduleId中的k2用v2替换
    indexRetrieve(baseId, mappingIdIndex, function(value) {
        indexRetrieve(moduleId, value, function(mdValue, mdKey) {
            moduleId = moduleId.replace(mdKey, mdValue);
          }
        );
     });
```
map就是paths多一层，先匹配父模块的id，再匹配子模块的id，最后替换的是子模块的id

## urlArgs

配置
```js
require.config({
    // ...

    urlArgs: 'v=2.0.0' // 指定所有模块的路径后参数
});
require.config({
    // ...

    urlArgs: {
        common: '1.2.0' // 为common和common下的子模块指定路径后参数
    }
});
```

createConfIndex中的处理
```js
urlArgsIndex = createKVSortedIndex(requireConf.urlArgs);
```

require.config中的处理
```js
if (newValue) {
        if (key === 'urlArgs' && typeof newValue === 'string') {
          defaultUrlArgs = newValue;
        } else {
```
在urlArgs设置为字符串时，会赋值给defaultUrlArgs

toUrl中的处理
```js
// 拼接查询字段，如果urlArgsIndex拼接了，defaultUrlArgs就不用拼接
    var isUrlArgsAppended;
    indexRetrieve(id, urlArgsIndex, function(value) {
      appendUrlArgs(value);
    });
    defaultUrlArgs && appendUrlArgs(defaultUrlArgs);
```
urlArgs主要作用是在模块的url上拼接一个查询字符串

## waitSeconds

配置
```js
require.config({
    // ...
    waitSeconds: 5
});
```
指定等待的秒数。超过等待时间后，如果有模块未成功加载或初始化，将抛出 MODULE_TIMEOUT 异常错误信息。

## noRequests

配置
```js
require.config({
      baseUrl: "http://j2.58cdn.com.cn/m58/njs/",
      noRequests: {
        "pkg/house/ershoufang_list": ["pkg/house/ershoufang_list"]
      }
    });
    require(['pkg/house/ershoufang_list'], function() {
      console.log('pkg ershoufang_list callback')
    });
```

createConfIndex中的处理
```js
noRequestsIndex = createKVSortedIndex(requireConf.noRequests);
    each(noRequestsIndex, function(item) {//[{reg:xx,k:xx,v:[xx,xx]}]
      var value = item.v;
      var mapIndex = {};
      item.v = mapIndex;
      if (!(value instanceof Array)) {
        value = [value];
      }
      each(value, function(meetId) { //[{reg:xx,k:xx,v:{xx:1,xx:1}}]
        mapIndex[meetId] = 1;
      });
    });
```

actualGlobalRequire函数中的处理
```js
each(pureModules, function(id) {
            var meet;
            // noRequestsIndex = //[{reg:xx,k:xx,v:{xx:1,xx:1}}]
            // reg要匹配id，v值的属性还要存在一个pureModules
            indexRetrieve(id, noRequestsIndex, function(value) {
                meet = value;
            });
            if (meet) {
              if (meet['*']) {
                noRequestModules[id] = 1;
              } else {
                each(pureModules, function(meetId) {
                  if (meet[meetId]) {
                    noRequestModules[id] = 1;
                    return false;
                  }
                });
              }
            }
          });

        nativeRequire(
          pureModules,
          function() { //require回调函数所在
            each(normalizedIds, function(id, i) {
              if (id == null) {
                normalizedIds[i] = normalize(requireId[i], baseId); // 对有感叹号的id做处理
              }
            });
            nativeRequire(normalizedIds, callback, baseId);
          },
          baseId,
          noRequestModules
        );
```

native中的处理
```js
each(ids, function(id) {
  if (!(BUILDIN_MODULE[id] || modIs(id, MODULE_DEFINED))) {
    modAddDefinedListener(id, tryFinishRequire); //为没有达到MODULE_DEFINED的模块增加监听函数
    if (!noRequests[id]) { //若存在就不加载
      (id.indexOf('!') > 0 ? loadResource : loadModule)(id, baseId);
    }
  }
});
```
noRequests的作用就是拒绝加载require中引入的模块
