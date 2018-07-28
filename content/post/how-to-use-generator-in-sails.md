+++
categories = ["Nodejs"]
date = "2017-03-13T22:50:49+08:00"
tags = ["Nodejs"]
title = "如何在Sails.js中使用generator"

+++

由于历史原因，公司的App项目用Sails.js框架来开发，那时候还没有ES6，当然是一层一层callback下去，后来引进了Async和Thenjs，代码才清晰了很多，但是遇到复杂的业务逻辑，写起来依旧痛苦。

在这个不写ES6都不好意思说写Nodejs的年代，天天写callback很不是滋味，所以必须找个机会在不换框架的情况也能使用yield来解决callback的问题。

首先想到的肯定是用co这个包，它有一个wrap的方法，可以把generator转成callback，试想一下，如果所有的controller的每个方法，都用co.wrap一下，岂不是达到了想要的效果？看了下sails的源码，貌似必须要修改它的源码才行。那么问题来了，怎么样才能在不修改它源码的情况也能达到同样的效果呢？

由于javascript默认对象传引用，所以是不是可以直接把加载到内存中的controller方法全部用co.wrap包一下呢？

在看了sails的很多源码，经过很多次的尝试后，事实证明这种猜想是可以的！

只需要在config/bootstrap.js文件加上下面的代码，就可以在controller中使用generator了。

```
var _ = require('lodash');
var coExpress = require('co-express');

sails.modules.loadControllers(function (err, modules) {
  if (err) {
    return callback(err);
  }
  sails.controllers = _.merge(sails.controllers, modules);

  // hacking every action of all controllers
  _.each(sails.controllers, function(controller, controllerId) {
    _.each(controller, function(action, actionId) {
      actionId = actionId.toLowerCase();
      console.log('hacking route:', controllerId, actionId);
      // co.wrap，generator => callback
      action = coExpress(action);
      sails.hooks.controllers.middleware[controllerId][actionId] = action;
    });
  });
  // reload routes
  sails.router.load(function () {
    // reload blueprints
    sails.hooks.blueprints.initialize(function () {
      sails.hooks.blueprints.extendControllerMiddleware();
      sails.hooks.blueprints.bindShadowRoutes();
      callback();
    });
  });
});
```

这里用了一个叫co-express的包，它的源码很简单，就是使用co.wrap包装了一下。重要的是，在修改了每个action之后，我们需要重新load一次routes和blueprints，这样才真正在内存中生效。
