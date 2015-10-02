# AngularJS in Patterns

<!--toc-->

## Table of Contents

* [翻译](#翻译)
* [摘要](#摘要)
* [引言](#引言)
* [AngularJS 概览](#AngularJS-概览)
  * [局部模板（Partials）](#局部模板（Partials）)
  * [控制器](#控制器)
  * [Scope](#scope)
  * [指令](#指令)
  * [Filters](#filters)
  * [Services](#services)
* [AngularJS Patterns](#angularjs-patterns)
  * [Services](#services-1)
    * [Singleton](#singleton)
    * [Factory Method](#factory-method)
    * [Decorator](#decorator)
    * [Facade](#facade)
    * [Proxy](#proxy)
    * [Active Record](#active-record)
    * [Intercepting Filters](#intercepting-filters)
  * [Directives](#directives-1)
    * [Composite](#composite)
  * [Interpreter](#interpreter)
    * [Template View](#template-view)
  * [Scope](#scope-1)
    * [Observer](#observer)
    * [Chain of Responsibilities](#chain-of-responsibilities)
    * [Command](#command)
  * [Controller](#controller-1)
    * [Page Controller](#page-controller)
  * [Others](#others)
    * [Module Pattern](#module-pattern)
    * [Data Mapper](#data-mapper)
    * [Observer Pattern as an External Service](#observer-pattern-as-an-external-service)
* [References](#references)

<!--endtoc-->

## 翻译

- [日本语](https://github.com/mgechev/angularjs-in-patterns/blob/master/i18n/README-ja-jp.md) by [morizotter](https://twitter.com/morizotter)
- [俄语](http://habrahabr.ru/post/250149/)
- [法语](https://github.com/mgechev/angularjs-in-patterns/blob/master/i18n/README-fr-fr.md) by [manekinekko](https://github.com/manekinekko)
- 原版[英语](https://mgechev.github.io/angularjs-in-patterns/) by mgechev

## 摘要

学习一样新事物的最好方式之一就是看看它在用哪些你已经知道的东西。
本文不是想让读者熟悉设计模式或架构模式；它给出了对面向对象，设计模式和架构模式的基本理解。
本文的目的是描述几种不同的软件设计和架构模式如何应用于 AngularJS 或 AngularJS 单页面应用中。

## 引言

本文以对 AngularJS 的简要概览开始。概览这一章节解释了 AngularJS 的主要组成部分————指令（drectives），过滤器（filters），控制器（controllers），服务（services），作用域（scope）。第二章节按照上述几个组成部分分组，列出并描述了不同设计和架构模式在框架中各部分的应用。如果有部分模式在不同的部分被用到，将会有明显提及。

最后一个章节包含了一些经常在 AngularJS 单页面应用中用到的架构模式。

## AngularJS 概览

AngularJS 是一个Google开发的 JavaScript 框架。它的产生是为了给开发 CRUD 类单页面应用（SPA）一个坚实的基础。
SPA是一个web应用，一旦加载完成，用户发起操作时，就不需要重新加载整个页面。这意味着所有应用需要的资源都（数据，模板，脚本，样式）应该在初次请求时加载，或者以更好的方式加载————信息和资源都只在需要时加载。
既然大部分 CRUD 类应用都有相同的特点和要求，AngularJS 想要做的事便是提供一套理想化且易用的特性。AngularJS 中结果重要的特性如下：

- 双向数据绑定
- 依赖注入
- 可测试性
- 抽象化

将 AngularJS 应用分割成相互分离的部分来达到关注点分离的目的，比如：

- 局部模板（Partials）
- 控制器
- 指令
- 服务
- 过滤器

这些组成部分可以被归类到不同的模块，这有助于我们做出更高层的抽象，处理更复杂的情况。每一个部分都封装了应用逻辑的特定部分。

### 局部模板（Partials）

局部模板就是一些HTML字符串。它里面可能有带 AngularJS 表达式的元素或属性。AngularJS 与其它框架的区别就是 AngularJS 的模板不是一种需要转换成HTML的中间格式（例如mustache.js）。

一般来说，每个SPA会在初始阶段载入 `index.html` 文件。就 AngularJS 来说，这个文件会包括一系列标准的和自定义的HTML属性，标签和注释。它们可以配置并引导应用启动。所有后续的用户操作都只需要载入局部模板或改变应用的状态，例如通过框架提供的数据绑定。

**示例局部模板**

```HTML
<html ng-app>
 <!-- 用 ngController 指令增强 body 标签 -->
 <body ng-controller="MyController">
   <input ng-model="foo" value="bar">
   <!-- 带有 ng-click 指令的 button 标签, 
         和用 "{{ }}" 标记包围的字符串表达式'buttonText' -->
   <button ng-click="changeFoo()">{{buttonText}}</button>
   <script src="angular.js"></script>
 </body>
</html>
```

用 AngularJS 表达式局部地定义不同的交互引发将会调用什么方法处理。在上面的实例中，属性 `ng-click` 的值表示当前*作用域*的方法 `changeFoo` 将被调用。

### 控制器

AngularJS 控制器是 JavaScript 函数，它通过在“scope”上附加处理方法的方式，来处理用户与 web 应用间的交互（比如鼠标事件，键盘事件等等）。
所有控制器，组件通过 AngularJS 的依赖注入机制提供。控制器的另一个职责，是通过在“scope”上附加数据的方式，提供 *model* 给局部模板。
我们可以把这些数据看作 *view model*。

```JavaScript
function MyController($scope) {
  $scope.buttonText = 'Click me to change foo!';
  $scope.foo = 42;

  $scope.changeFoo = function () {
    $scope.foo += 1;
    alert('Foo changed');
  };
}
```

举个例子，如果我们把上面的示例控制器和先前给出的局部模板放到一块，用户可能通过以下几种方式交互：

1. 通过在输入框内输入字符，改变 `foo` 的值。由于双向数据绑定，将会立即显示 `foo` 的新值。
2. 通过点击 `Click me to change foo!` 按钮改变 `foo` 的值。

所有自定义元素、属性、注释或 class 都会被当做 AngularJS *指令*（directives），前提是指令已经定义。

### Scope

在 AngularJS 中，scope 是一个暴露给局部模板的 JavaScript 对象。Scope 可以包含不同的属性——原始类型，对象或方法。所有附加在 scope 上的方法可以在局部模板中的 AngularJS 表达式被解析的时候被调用，也可以直接在任何组件中调用它，只要它与这个 scope 相关联。
通过合理使用*指令*，附加在 scope 上的数据会被绑定到视图，就是说，每个局部视图上的改变会影响到 scope 的属性，每次 scope 的属性改变，局部视图上相应的地方也会跟着改变。

Scope 另一个重要的特性是它们都连接到了一个原型链（除了哪些被声明为 *isolated*（独立作用域）的 scope）。这样，任何一个子 scope 都可以调用它的父级 scope 上的方法，因为这些方法都是子级 scope 原型上的属性（直接或间接的）。

Scope 继承可以用下面的例子描述：

```HTML
<div ng-controller="BaseCtrl">
  <div id="child" ng-controller="ChildCtrl">
    <button id="parent-method" ng-click="foo()">Parent method</button>
    <button ng-click="bar()">Child method</button>
  </div>
</div>
```

```JavaScript
function BaseCtrl($scope) {
  $scope.foo = function () {
    alert('Base foo');
  };
}

function ChildCtrl($scope) {
  $scope.bar = function () {
    alert('Child bar');
  };
}
```

`div#child` 与 `ChildCtrl` 相关联，但因为注入到 `ChildCtrl` 内部的 scope 原型式地从它的父级 scope 继承了 `foo` 方法（即注入到 `BaseCtrl` 里），`foo` 方法可以在 `div#child` 内部的 `button#parent-method` 里使用。

### 指令

在 AngularJS 中，所有的 DOM 操作用过放置在指令（directive）中。根据一般经验，当你的控制器中有 DOM 操作时，你应该创建一个新的指令或者考虑重构一个已有的，可处理所需 DOM 操作的指令。
每个指令都有一个名字和与它相关的逻辑。在最简单的情况下，一个指令只包含名字和 *postLink* 方法的定义，这个方法封装了这个指令的所有逻辑。在比较复杂的情况下，一个指令还包含其它很多属性，比如：

- template（模板）
- compile function（编译方法）
- link function（链接方法）
- ...

通过引用指令名，可以在局部模板中使用指令。

示例:

```JavaScript
myModule.directive('alertButton', function () {
  return {
    template: '<button ng-transclude></button>',
    scope: {
      content: '@'
    },
    replace: true,
    restrict: 'E',
    transclude: true,
    link: function (scope, el) {
      el.click(function () {
        alert(scope.content);
      });
    }
  };
});
```

```HTML
<alert-button content="42">Click me</alert-button>
```

上面的例子中，`<alert-button></alert-button>` 标签将会被替换成 button 元素。当用户点击这个 button 时，字符串 `42` 将会被 alert 出来。

因为本文并不是要完整叙述 AngularJS 的 API，我们对指令的叙述到此为止。

### Filters

The filters in AngularJS are responsible for encapsulating logic required for formatting data. Usually filters are used inside the partials but they are also accessible in the controllers, directives, *services* and other filters through Dependency Injection.

Here is a definition of a sample filter, which changes the given string to uppercase:

```JavaScript
myModule.filter('uppercase', function () {
  return function (str) {
    return (str || '').toUpperCase();
  };
});
```

Inside a partial this filter could be used using the Unix's piping syntax:

```HTML
<div>{{ name | uppercase }}</div>
```

Inside a controller the filter could be used as follows:

```JavaScript
function MyCtrl(uppercaseFilter) {
  $scope.name = uppercaseFilter('foo'); //FOO
}
```

### Services

Every piece of logic, which doesn't belong to the components described above should be placed inside a service. Usually services encapsulate the domain specific logic, persistence logic, XHR, WebSockets, etc. When the controllers in the application became too "fat" the repetitive code should be placed inside a service.

```JavaScript
myModule.service('Developer', function () {
  this.name = 'Foo';
  this.motherLanguage = 'JavaScript';
  this.live = function () {
    while (true) {
      this.code();
    }
  };
});
```

The service could be injected inside any component, which supports dependency injection (controllers, other services, filters, directives).

```JavaScript
function MyCtrl(Developer) {
  var developer = new Developer();
  developer.live();
}
```

## AngularJS Patterns

In the next a couple of sections, we are going to take a look how the traditional design and architectural patterns are composed in the AngularJS components.

In the last chapter we are going to take a look at some architectural patterns, which are frequently used in the development of Single-Page Applications with (but not limited to) AngularJS.

### Services

#### Singleton

> 单例模式是一个用来限制类到对象的初始化过程的设计模式。当你只需要一个唯一的对象来协调系统操作时，这个模式非常有用。它的核心理念是，当系统只有一个或者限定数目的对象实例存在时，系统会运行得更高效。

下面的 UML 图描述了单例模式。

![Singleton](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/singleton.svg "Fig. 1")

When given dependency is required by any component, AngularJS resolves it using the following algorithm:

- Takes its name and makes a lookup at a hash map, which is defined into a lexical closure (so it has a private visibility).
- If the dependency exists AngularJS pass it as parameter to the component, which requires it.
- If the dependency does not exists:
  - AngularJS instantiate it by calling the factory method of its provider (i.e. `$get`). Note that instantiating the dependency may require recursive call to the same algorithm, for resolving all the dependencies required by the given dependency. This process may lead to circular dependency.
  - AngularJS caches it inside the hash map mentioned above.
  - AngularJS passes it as parameter to the component, which requires it.

We can take better look at the AngularJS' source code, which implements the method `getService`:

```JavaScript
function getService(serviceName) {
  if (cache.hasOwnProperty(serviceName)) {
    if (cache[serviceName] === INSTANTIATING) {
      throw $injectorMinErr('cdep', 'Circular dependency found: {0}', path.join(' <- '));
    }
    return cache[serviceName];
  } else {
    try {
      path.unshift(serviceName);
      cache[serviceName] = INSTANTIATING;
      return cache[serviceName] = factory(serviceName);
    } catch (err) {
      if (cache[serviceName] === INSTANTIATING) {
        delete cache[serviceName];
      }
      throw err;
    } finally {
      path.shift();
    }
  }
}
```

We can think of each service as a singleton, because each service is instantiated no more than a single time. We can consider the cache as a singleton manager. There is a slight variation from the UML diagram illustrated above because instead of keeping static, private reference to the singleton inside its constructor function, we keep the reference inside the singleton manager (stated in the snippet above as `cache`).

This way the services are actually singletons but not implemented through the Singleton pattern, which provides a few advantages over the standard implementation:

- It improves the testability of your source code
- You can control the creation of singleton objects (in our case the IoC container controls it for us, by instantiating the singletons lazy)

For further discussion on this topic Misko Hevery's [article](http://googletesting.blogspot.com/2008/05/tott-using-dependancy-injection-to.html) in the Google Testing blog could be considered.

#### Factory Method

>The factory method pattern is a creational pattern, which uses factory methods to deal with the problem of creating objects without specifying the exact class of object that will be created. This is done by creating objects via a factory method, which is either specified in an interface (abstract class) and implemented in implementing classes (concrete classes); or implemented in a base class, which can be overridden when inherited in derived classes; rather than by a constructor.

![Factory Method](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/factory-method.svg "Fig. 2")

Lets consider the following snippet:

```JavaScript
myModule.config(function ($provide) {
  $provide.provider('foo', function () {
    var baz = 42;
    return {
      //Factory method
      $get: function (bar) {
        var baz = bar.baz();
        return {
          baz: baz
        };
      }
    };
  });
});

```

In the code above we use the `config` callback in order to define new "provider". Provider is an object, which has a method called `$get`. Since in JavaScript we don't have interfaces and the language is duck-typed there is a convention to name the factory method of the providers that way.

Each service, filter, directive and controller has a provider (i.e. object which factory method, called `$get`), which is responsible for creating the component's instance.

We can dig a little bit deeper in AngularJS' implementation:

```JavaScript
//...

createInternalInjector(instanceCache, function(servicename) {
  var provider = providerInjector.get(servicename + providerSuffix);
  return instanceInjector.invoke(provider.$get, provider, undefined, servicename);
}, strictDi));

//...

function invoke(fn, self, locals, serviceName){
  if (typeof locals === 'string') {
    serviceName = locals;
    locals = null;
  }

  var args = [],
      $inject = annotate(fn, strictDi, serviceName),
      length, i,
      key;

  for(i = 0, length = $inject.length; i < length; i++) {
    key = $inject[i];
    if (typeof key !== 'string') {
      throw $injectorMinErr('itkn',
              'Incorrect injection token! Expected service name as string, got {0}', key);
    }
    args.push(
      locals && locals.hasOwnProperty(key)
      ? locals[key]
      : getService(key)
    );
  }
  if (!fn.$inject) {
    // this means that we must be an array.
    fn = fn[length];
  }

  return fn.apply(self, args);
}
```

From the example above we can notice how the `$get` method is actually used:

```JavaScript
instanceInjector.invoke(provider.$get, provider, undefined, servicename)
```

The snippet above calls the `invoke` method of `instanceInjector` with the factory method (i.e. `$get`) of given service, as first argument. Inside `invoke`'s body `annotate` is called with first argument the factory method. Annotate resolves all dependencies through the dependency injection mechanism of AngularJS, which was considered above. When all dependencies are resolved the factory method is being called: `fn.apply(self, args)`.

If we think in terms of the UML diagram above we can call the provider a "ConcreteCreator" and the actual component, which is being created a "Product".

There are a few benefits of using the factory method pattern in this case, because of the indirection it creates. This way the framework can take care of the boilerplates during the instantiation of new components like:

- The most appropriate moment, when the component needs to be instantiated
- Resolving all the dependencies required by the component
- The number of instances the given component is allowed to have (for services and filters only a single one but multiple for the controllers)

#### Decorator

>The decorator pattern (also known as Wrapper, an alternative naming shared with the Adapter pattern) is a design pattern that allows behavior to be added to an individual object, either statically or dynamically, without affecting the behavior of other objects from the same class.

![Decorator](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/decorator.svg "Fig. 4")

AngularJS provides out-of-the-box way for extending and/or enhancing the functionality of already existing services. Using the method `decorator` of `$provide` you can create "wrapper" of any service you have previously defined or used by a third-party:

```JavaScript
myModule.controller('MainCtrl', function (foo) {
  foo.bar();
});

myModule.factory('foo', function () {
  return {
    bar: function () {
      console.log('I\'m bar');
    },
    baz: function () {
      console.log('I\'m baz');
    }
  };
});

myModule.config(function ($provide) {
  $provide.decorator('foo', function ($delegate) {
    var barBackup = $delegate.bar;
    $delegate.bar = function () {
      console.log('Decorated');
      barBackup.apply($delegate, arguments);
    };
    return $delegate;
  });
});
```
The example above defines new service called `foo`. In the `config` callback is called the method `$provide.decorator` with first argument `"foo"`, which is the name of the service, we want to decorate and second argument factory function, which implements the actual decoration. `$delegate` keeps reference to the original service `foo`. Using the dependency injection mechanism of AngularJS, reference to this local dependency is passed as first argument of the constructor function.
We decorate the service by overriding its method `bar`. The actual decoration is simply extending `bar` by invoking one more `console.log statement` - `console.log('Decorated');` and after that call the original `bar` method with the appropriate context.

Using this pattern is especially useful when we need to modify the functionality of third party services. In cases when multiple similar decorations are required (like performance measurement of multiple methods, authorization, logging, etc.), we may have a lot of duplications and violate the DRY principle. In such cases it is useful to use [aspect-oriented programming](http://en.wikipedia.org/wiki/Aspect-oriented_programming). The only AOP framework for AngularJS I'm aware of could be found at [github.com/mgechev/angular-aop](https://github.com/mgechev/angular-aop).

#### Facade

>A facade is an object that provides a simplified interface to a larger body of code, such as a class library. A facade can:

>1. make a software library easier to use, understand and test, since the facade has convenient methods for common tasks;

>2. make the library more readable, for the same reason;

>3. reduce dependencies of outside code on the inner workings of a library, since most code uses the facade, thus allowing more flexibility in developing the system;

>4. wrap a poorly designed collection of APIs with a single well-designed API (as per task needs).

![Facade](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/facade.svg "Fig. 11")

There are a few facades in AngularJS. Each time you want to provide higher level API to given functionality you practically create a facade.

For example, lets take a look how we can create an `XMLHttpRequest` POST request:

```JavaScript
var http = new XMLHttpRequest(),
    url = '/example/new',
    params = encodeURIComponent(data);
http.open("POST", url, true);

http.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
http.setRequestHeader("Content-length", params.length);
http.setRequestHeader("Connection", "close");

http.onreadystatechange = function () {
  if(http.readyState == 4 && http.status == 200) {
    alert(http.responseText);
  }
}
http.send(params);
```
But if we want to post this data using the AngularJS' `$http` service we can:

```JavaScript
$http({
  method: 'POST',
  url: '/example/new',
  data: data
})
.then(function (response) {
  alert(response);
});
```
or we can even:

```JavaScript
$http.post('/someUrl', data)
.then(function (response) {
  alert(response);
});
```
The second option provides pre-configured version, which creates a HTTP POST request to the given URL.

Even higher level of abstraction is being created by `$resource`, which is build over the `$http` service. We will take a further look at this service in [Active Record](#active-record) and [Proxy](#proxy) sections.

#### Proxy

>A proxy, in its most general form, is a class functioning as an interface to something else. The proxy could interface to anything: a network connection, a large object in memory, a file, or some other resource that is expensive or impossible to duplicate.

![Proxy](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/proxy.svg "Fig. 9")

We can distinguish three different types of proxy:

- Virtual Proxy
- Remote Proxy
- Protection Proxy

In this sub-chapter we are going to take a look at AngularJS' implementation of Virtual Proxy.

In the snippet bellow, there is a call to the `get` method of `$resource` instance, called `User`:

```JavaScript
var User = $resource('/users/:id'),
    user = User.get({ id: 42 });
console.log(user); //{}
```

`console.log` would outputs an empty object. Since the AJAX request, which happens behind the scene, when `User.get` is invoked, is asynchronous, we don't have the actual user when `console.log` is called. Just after `User.get` makes the GET request it returns an empty object and keeps reference to it. We can think of this object as virtual proxy (a simple placeholder), which would be populated with the actual data once the client receives response by the server.

How does this works with AngularJS? Well, lets consider the following snippet:

```JavaScript
function MainCtrl($scope, $resource) {
  var User = $resource('/users/:id'),
  $scope.user = User.get({ id: 42 });
}
```

```html
<span ng-bind="user.name"></span>
```
Initially when the snippet above executes, the property `user` of the `$scope` object will be with value an empty object (`{}`), which means that `user.name` will be undefined and nothing will be rendered. Internally AngularJS will keep reference to this empty object. Once the server returns response for the get request, AngularJS will populate the object with the data, received from the server. During the next `$digest` loop AngularJS will detect change in `$scope.user`, which will lead to update of the view.

#### Active Record

>The Active Record object is an object, which carries both data and behavior. Usually most of the data in these objects is persistent, responsibility of the Active Record object is to take care of the communication with the database in order to create, update, retrieve or delete the data. It may delegate this responsibility to lower level objects but calls to instance or static methods of the active record object cause the database communication.

![Active Record](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/active-record.svg "Fig. 7")

AngularJS defines a service called `$resource`. In the current version of AngularJS (1.2+) it is being distributed in module outside of the AngularJS' core.

According to the AngularJS' documentation `$resource` is:

>A factory which creates a resource object that lets you interact with RESTful server-side data sources.
>The returned resource object has action methods which provide high-level behaviors without the need to interact with the low level $http service.

Here is how `$resource` could be used:

```JavaScript
var User = $resource('/users/:id'),
    user = new User({
      name: 'foo',
      age : 42
    });

user.$save();
```

The call of `$resource` will create a constructor function for our model instances. Each of the model instances will have methods, which could be used for the different CRUD operations.

This way we can use the constructor function and its static methods by:

```JavaScript
User.get({ userid: userid });
```

The code above will immediately return an empty object and keep reference to it. Once the response have been successfully returned and parsed, AngularJS will populate this object with the received data (see [proxy](#proxy)).

You can find more details for `$resource` [The magic of $resource](http://blog.mgechev.com/2014/02/05/angularjs-resource-active-record-http/) and [AngularJS' documentation](https://docs.angularjs.org/api/ngResource/service/$resource).

Since Martin Fowler states that

> responsibility of the Active Record object is to take care of the communication with the databse in order to create...

`$resource` does not implements exactly the Active Record pattern, since it communicates with RESTful service instead of the database. Anyway, we can consider it as "Active Record like RESTful communication".

#### Intercepting Filters

>Create a chain of composable filters to implement common pre-processing and post-processing tasks during a Web page request.

![Composite](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/intercepting-filters.svg "Fig. 3")

In some cases you need to do some kind of pre and/or post processing of HTTP requests. In the case of the Intercepting Filters you pre/post process given HTTP request/response in order to include logging, security or any other concern, which is influenced by the request body or headers. Basically the Intercepting Filters pattern include a chain of filters, each of which process data in given order. The output of each filter is input of the next one.

In AngularJS we have the idea of the Intercepting Filters in `$httpProvider`. `$httpProvider` has an array property called `interceptors`, which contains a list of objects. Each object may have properties called: `request`, `response`, `requestError`, `responseError`.

`requestError` is an interceptor, which gets called when a previous interceptor threw an error or resolved with a rejection, respectively `responseError` is being called when the previous `response` interceptor has thrown an error.

Here is a basic example how you can add interceptors using object literal:

```JavaScript
$httpProvider.interceptors.push(function($q, dependency1, dependency2) {
  return {
   'request': function(config) {
       // same as above
    },
    'response': function(response) {
       // same as above
    }
  };
});
```

### Directives

#### Composite

>The composite pattern is a partitioning design pattern. The composite pattern describes that a group of objects are to be treated in the same way as a single instance of an object. The intent of a composite is to "compose" objects into tree structures to represent part-whole hierarchies.

![Composite](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/composite.svg "Fig. 3")

According to the Gang of Four, MVC is nothing more than combination of:

- Strategy
- Composite
- Observer

They state that the view is composition of components. In AngularJS the situation is similar. Our views are formed by a composition of directives and DOM elements, on which these directives could be applied.

Lets look at the following example:

```HTML
<!doctype html>
<html>
  <head>
  </head>
  <body>
    <zippy title="Zippy">
      Zippy!
    </zippy>
  </body>
</html>
```

```JavaScript
myModule.directive('zippy', function () {
  return {
    restrict: 'E',
    template: '<div><div class="header"></div><div class="content" ng-transclude></div></div>',
    link: function (scope, el) {
      el.find('.header').click(function () {
        el.find('.content').toggle();
      });
    }
  }
});
```

This example defines a simple directive, which is a UI component. The defined component (called "zippy") has header and content. Click on its header toggles the visibility of the content.

From the first example we can note that the whole DOM tree is a composition of elements. The root component is the `html` element, directly followed by the nested elements `head` and `body` and so on...

In the second, JavaScript, example we see that the `template` property of the directive, contains markup with `ng-transclude` directive inside it. So this means that inside the directive `zippy` we have another directive called `ng-transclude`, i.e. composition of directives. Theoretically we can nest the components infinitely until we reach a leaf node.

### Interpreter

>In computer programming, the interpreter pattern is a design pattern that specifies how to evaluate sentences in a language. The basic idea is to have a class for each symbol (terminal or nonterminal) in a specialized computer language. The syntax tree of a sentence in the language is an instance of the composite pattern and is used to evaluate (interpret) the sentence.

![Interpreter](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/interpreter.svg "Fig. 6")

Behind its `$parse` service, AngularJS provides its own implementation of interpreter of a DSL (Domain Specific Language). The used DSL is simplified and modified version of JavaScript.
The main differences between the JavaScript expressions and AngularJS expressions that AngularJS expressions:

- may contain filters with UNIX like pipe syntax
- don't throw any errors
- don't have any control flow statements (exceptions, loops, if statements although you can use the ternary operator)
- are evaluated in given context (the context of the current `$scope`)

Inside the `$parse` service are defined two main components:

```JavaScript
//Responsible for converting given string into tokens
var Lexer;
//Responsible for parsing the tokens and evaluating the expression
var Parser;
```

Once given expression have been tokenized it is cached internally, because of performance concerns.

The terminal expressions in the AngularJS DSL are defined as follows:

```JavaScript
var OPERATORS = {
  /* jshint bitwise : false */
  'null':function(){return null;},
  'true':function(){return true;},
  'false':function(){return false;},
  undefined:noop,
  '+':function(self, locals, a,b){
        //...
      },
  '*':function(self, locals, a,b){return a(self, locals)*b(self, locals);},
  '/':function(self, locals, a,b){return a(self, locals)/b(self, locals);},
  '%':function(self, locals, a,b){return a(self, locals)%b(self, locals);},
  '^':function(self, locals, a,b){return a(self, locals)^b(self, locals);},
  '=':noop,
  '===':function(self, locals, a, b){return a(self, locals)===b(self, locals);},
  '!==':function(self, locals, a, b){return a(self, locals)!==b(self, locals);},
  '==':function(self, locals, a,b){return a(self, locals)==b(self, locals);},
  '!=':function(self, locals, a,b){return a(self, locals)!=b(self, locals);},
  '<':function(self, locals, a,b){return a(self, locals)<b(self, locals);},
  '>':function(self, locals, a,b){return a(self, locals)>b(self, locals);},
  '<=':function(self, locals, a,b){return a(self, locals)<=b(self, locals);},
  '>=':function(self, locals, a,b){return a(self, locals)>=b(self, locals);},
  '&&':function(self, locals, a,b){return a(self, locals)&&b(self, locals);},
  '||':function(self, locals, a,b){return a(self, locals)||b(self, locals);},
  '&':function(self, locals, a,b){return a(self, locals)&b(self, locals);},
  '|':function(self, locals, a,b){return b(self, locals)(self, locals, a(self, locals));},
  '!':function(self, locals, a){return !a(self, locals);}
};
```

We can think of the function associated with each terminal as implementation of the `AbstractExpression`'s interface.

Each `Client` interprets given AngularJS expression in a specific context - specific scope.

Few sample AngularJS expressions are:

```JavaScript
// toUpperCase filter is applied to the result of the expression
// (foo) ? bar : baz
(foo) ? bar : baz | toUpperCase
```

#### Template View

> Renders information into HTML by embedding markers in an HTML page.

![Template View](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/template-view.svg "Fig. 8")

The dynamic page rendering is not that trivial thing. It is connected with a lot of string concatenations, manipulations and frustration. Far easier way to build your dynamic page is to write your markup and embed little expressions inside it, which are lately evaluated in given context and so the whole template is being compiled to its end format. In our case this format is going to be HTML (or even DOM). This is exactly what the template engines do - they take given DSL, evaluate it in the appropriate context and then turn it into its end format.

Templates are very commonly used especially in the back-end.
For example, you can embed PHP code inside HTML and create a dynamic page, you can use Smarty or you can use eRuby with Ruby in order to embed Ruby code inside your static pages.

For JavaScript there are plenty of template engines, such as mustache.js, handlebars, etc. Most of these engines manipulate the template as a string. The template could be located in different places - as static file, which is fetched with AJAX, as `script` embedded inside your view or even inlined into your JavaScript.

For example:

```html
<script type="template/mustache">
  <h2>Names</h2>
  {{#names}}
    <strong>{{name}}</strong>
  {{/names}}
</script>
```

The template engine turns this string into DOM elements by compiling it within a given context. This way all the expressions embedded in the markup are evaluated and replaced by their value.

For example if we evaluate the template above in the context of the following object: `{ names: ['foo', 'bar', 'baz'] }`, so we will get:

```html
<h2>Names</h2>
  <strong>foo</strong>
  <strong>bar</strong>
  <strong>baz</strong>
```

AngularJS templates are actually HTML, they are not in an intermediate format like the traditional templates are.
What AngularJS compiler does is to traverse the DOM tree and look for already known directives (elements, attributes, classes or even comments). When AngularJS finds any of these directives it invokes the logic associated with them, which may involve evaluation of different expressions in the context of the current scope.

For example:

```html
<ul ng-repeat="name in names">
  <li>{{name}}</li>
</ul>
```

in the context of the scope:

```javascript
$scope.names = ['foo', 'bar', 'baz'];
```

will produce the same result as the one above. The main difference here is that the template is not wrapped inside a `script` tag but is HTML instead.


### Scope

#### Observer

>The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods. It is mainly used to implement distributed event handling systems.

![Observer](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/observer.svg "Fig. 7")

There are two basic ways of communication between the scopes in an AngularJS application. The first one is calling methods of parent scope by a child scope. This is possible since the child scope inherits prototypically by its parent, as mentioned above (see [Scope](#scope)). This allows communication in a single direction - child to parent. Some times it is necessary to call method of given child scope or notify it about a triggered event in the context of the parent scope. AngularJS provides built-in observer pattern, which allows this. Another possible use case, of the observer pattern, is when multiple scopes are interested in given event but the scope, in which context the event is triggered, is not aware of them. This allows decoupling between the different scopes, non of the scopes should be aware of the rest of the scopes.

Each AngularJS scope has public methods called `$on`, `$emit` and `$broadcast`. The method `$on` accepts topic as first argument and callback as second. We can think of the callback as an observer - an object, which implements the `Observer` interface (in JavaScript the functions are first-class, so we can provide only implementation of the `notify` method):

```JavaScript
function ExampleCtrl($scope) {
  $scope.$on('event-name', function handler() {
    //body
  });
}
```

In this way the current scope "subscribes" to events of type `event-name`. When `event-name` is triggered in any parent or child scope of the given one, `handler` would be called.

The methods `$emit` and `$broadcast` are used for triggering events respectively upwards and downwards through the scope chain.
For example:

```JavaScript
function ExampleCtrl($scope) {
  $scope.$emit('event-name', { foo: 'bar' });
}
```

The scope in the example above, triggers the event `event-name` to all scopes upwards. This means that each of the parent scopes of the given one, which are subscribed to the event `event-name`, would be notified and their handler callback will be invoked.

Analogical is the case when the method `$broadcast` is called. The only difference is that the event would be transmitted downwards - to all children scopes.
Each scope can subscribe to any event with multiple callbacks (i.e. it can associate multiple observers to given event).

In the JavaScript community this pattern is better known as publish/subscribe.

For a best practice example see [Observer Pattern as an External Service](#observer-pattern-as-an-external-service)

#### Chain of Responsibilities

>The chain-of-responsibility pattern is a design pattern consisting of a source of command objects and a series of processing objects. Each processing object contains logic that defines the types of command objects that it can handle; the rest are passed to the next processing object in the chain. A mechanism also exists for adding new processing objects to the end of this chain.

![Chain of Responsibilities](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/chain-of-responsibilities.svg "Fig. 5")

As stated above the scopes in an AngularJS application form a hierarchy known as the scope chain. Some of the scopes are "isolated", which means that they don't inherit prototypically by their parent scope, but are connected to it via their `$parent` property.

When `$emit` or `$broadcast` are called we can think of the scope chain as event bus, or even more accurately chain of responsibilities. Once the event is triggered it is emitted downwards or upwards (depending on the method, which was called). Each subsequent scope may:

- Handle the event and pass it to the next scope in the chain
- Handle the event and stop its propagation
- Pass the event to the next scope in the chain without handling it
- Stop the event propagation without handling it

In the example bellow you can see an example in which `ChildCtrl` triggers an event, which is propagated upwards through the scope chain. In the case above each of the parent scopes (the one used in `ParentCtrl` and the one used in `MainCtrl`) are going to handle the event by logging into the console: `"foo received"`. If any of the scopes should be considered as final destination it can call the method `stopPropagation` of the event object, passed to the callback.

```JavaScript
myModule.controller('MainCtrl', function ($scope) {
  $scope.$on('foo', function () {
    console.log('foo received');
  });
});

myModule.controller('ParentCtrl', function ($scope) {
  $scope.$on('foo', function (e) {
    console.log('foo received');
  });
});

myModule.controller('ChildCtrl', function ($scope) {
  $scope.$emit('foo');
});
```

The different handlers from the UML diagram above are the different scopes, injected to the controllers.

#### Command

>In object-oriented programming, the command pattern is a behavioral design pattern in which an object is used to represent and encapsulate all the information needed to call a method at a later time. This information includes the method name, the object that owns the method and values for the method parameters.

![Command](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/command.svg "Fig. 11")

Before continuing with the application of the command pattern lets describe how AngularJS implements data binding.

When we want to bind our model to the view we use the directives `ng-bind` (for single-way data binding) and `ng-model` (for two-way data binding). For example, if we want each change in the model `foo` to reflect the view we can:

```html
<span ng-bind="foo"></span>
```

Now each time we change the value of `foo` the inner text of the span will be changed. We can achieve the same effect with more complex AngularJS expressions, like:

```html
<span ng-bind="foo + ' ' + bar | uppercase"></span>
```

In the example above the value of the span will be the concatenated uppercased value of `foo` and `bar`. What happens behind the scene?

Each `$scope` has method called `$watch`. When the AngularJS compiler find the directive `ng-bind` it creates a new watcher of the expression `foo + ' ' + bar | uppercase`, i.e. `$scope.$watch("foo + ' ' + bar | uppercase", function () { /* body */ });`. The callback will be triggered each time the value of the expression change. In the current case the callback will update the value of the span.

Here are the first a couple of lines of the implementation of `$watch`:

```javascript
$watch: function(watchExp, listener, objectEquality) {
  var scope = this,
      get = compileToFn(watchExp, 'watch'),
      array = scope.$$watchers,
      watcher = {
        fn: listener,
        last: initWatchVal,
        get: get,
        exp: watchExp,
        eq: !!objectEquality
      };
//...
```

We can think of the `watcher` object as a command. The expression of the command is being evaluated on each [`"$digest"`](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$digest) loop. Once AngularJS detects change in the expression, it invokes the `listener` function. The `watcher` command encapsulates the whole information required for watching given expression and delegates the execution of the command to the `listener` (the actual receiver). We can think of the `$scope` as the command's `Client` and the `$digest` loop as the command's `Invoker`.

### Controllers

#### Page Controller

>An object that handles a request for a specific page or action on a Web site. Martin Fowler

![Page Controller](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/page-controller.svg "Fig. 8")

According to [4](#references) the page controller:

>Page Controller pattern accept input from the page request, invoke the requested actions on the model, and determine the correct view to use for the resulting page. Separate the dispatching logic from any view-related code

Since there is a lot of duplicate behavior between the different pages (like rendering footers, headers, taking care of the user's session, etc.) page controllers can form a hierarchy. In AngularJS we have controllers, which are with more limited scope of responsibilities. They don't accept user requests, since this is responsibility of the `$route` or `$state` services and the page rendering is responsibility of the directives `ng-view`/`ui-view`.

Similarly to the page controllers, AngularJS controllers handle user interactions, provide and update the models. The model is exposed to the view when it is being attached to the scope, all methods invoked by the view, in result of user actions, are ones, which are already attached to the scope. Another similarity between the page controllers and the AngularJS controllers is the hierarchy, which they form. It corresponds to the scope hierarchy. That way common actions can be isolated to the base controllers.

The controllers in AngularJS are quite similar to the code-behind in ASP.NET WebForms, since their responsibilities almost overlap.
Here is an example hierarchy between few controllers:

```HTML
<!doctype html>
<html>
  <head>
  </head>
  <body ng-controller="MainCtrl">
    <div ng-controller="ChildCtrl">
      <span>{{user.name}}</span>
      <button ng-click="click()">Click</button>
    </div>
  </body>
</html>
```

```JavaScript
function MainCtrl($scope, $location, User) {
  if (!User.isAuthenticated()) {
    $location.path('/unauthenticated');
  }
}

function ChildCtrl($scope, User) {
  $scope.click = function () {
    alert('You clicked me!');
  };
  $scope.user = User.get(0);
}
```

This example aims to illustrates the most trivial way to reuse logic by using a base controller, anyway in production applications I don't recommend you to put your authorization logic in the controllers. The access to the different routes could be determined on a higher level of abstraction.

The `ChildCtrl` is responsible for handling actions such as clicking the button with label `"Click"` and exposing the model to the view, by attaching it to the scope.

### Others

#### Module Pattern

This is actually not a design pattern from Gang of Four, neither one from P of EAA. This is a traditional JavaScript pattern, which main goal is to provide encapsulation and privacy.

Using the module pattern you can achieve privacy based on the JavaScript's functional lexical scope. Each module may have zero or more private members, which are hidden in the local scope of a function. This function returns an object, which exports the public API of the given module:

```javascript
var Page = (function () {

  var title;

  function setTitle(t) {
    document.title = t;
    title = t;
  }

  function getTitle() {
    return title;
  }

  return {
    setTitle: setTitle,
    getTitle: getTitle
  };
}());
```

In the example above we have IIFE (Immediately-Invoked Function Expression), which after being called returns an object, with two methods (`setTitle` and `getTitle`). The returned object is being assigned to the `Page` variable.

In this case the user of the `Page` object doesn't has direct access to the `title` variable, which is defined inside the local scope of the IIFE.

The module pattern is very useful when defining services in AngularJS. Using this pattern we can simulate (and actually achieve) privacy:

```javascript
app.factory('foo', function () {

  function privateMember() {
    //body...
  }

  function publicMember() {
    //body...
    privateMember();
    //body
  }

  return {
    publicMember: publicMember
  };
});
```

Once we want to inject `foo` inside any other component we won't be able to use the private methods, but only the public ones. This solution is extremely powerful especially when one is building a reusable library.

### Data Mapper

>A Data Mapper is a Data Access Layer that performs bidirectional transfer of data between a persistent data store (often a relational database) and an in memory data representation (the domain layer). The goal of the pattern is to keep the in memory representation and the persistent data store independent of each other and the data mapper itself.

![Data Mapper](https://rawgit.com/mgechev/angularjs-in-patterns/master/images/data-mapper.svg "Fig. 10")

As the description above states, the data mapper is used for bidirectional transfer of data between a persistent data store and an in memory data representation. Usually our AngularJS application communicates with API server, which is written in any server-side language (Ruby, PHP, Java, JavaScript, etc.).

Usually, if we have RESTful API `$resource` will help us communicate with the server in Active Record like fashion. Although, in some applications the data entities returned by the server are not in the most appropriate format, which we want to use in the front-end.

For instance, lets assume we have application in which each user has:

- name
- address
- list of friends

And our API has the methods:

- `GET /user/:id` - returns the user's name and the address of given user
- `GET /friends/:id` - returns the list of friends of given user

Possible solution is to have two different services, one for the first method and one for the second one. Probably more useful solution would be if we have a single service called `User`, which loads the user's friends when we request the user:

```javascript
app.factory('User', function ($q) {

  function User(name, address, friends) {
    this.name = name;
    this.address = address;
    this.friends = friends;
  }

  User.get = function (params) {
    var user = $http.get('/user/' + params.id),
        friends = $http.get('/friends/' + params.id);
    $q.all([user, friends])
    .then(function (user, friends) {
      return new User(user.name, user.address, friends);
    });
  };
  return User;
});
```

This way we create pseudo-data mapper, which adapts our API according to the SPA requirements.

We can use the `User` service by:

```javascript
function MainCtrl($scope, User) {
  User.get({ id: 1 })
  .then(function (data) {
    $scope.user = data;
  });
}
```

And the following partial:

```html
<div>
  <div>
    Name: {{user.name}}
  </div>
  <div>
    Address: {{user.address}}
  </div>
  <div>
    Friends with ids:
    <ul>
      <li ng-repeat="friend in user.friends">{{friend}}</li>
    </ul>
  </div>
</div>
```

### Observer Pattern as an External Service

##### About

Below is an example taken from [here](https://github.com/greglbd/angular-observer-pattern). This is an angular factory which creates a service implementing the Observer Pattern.  It works well with the ControllerAs method of working as it can be much more efficient that `$scope.$watch` and more specific to a unique scope or object than $emit and $broadcast when used correctly.

**Use Case:** You would use this pattern to communicate between 2 controllers that use the same model but are not connected in anyway.

##### Controller Example

Below example shows how to attach, notify and detach an event.

```javascript
angular.module('app.controllers')
  .controller('ObserverExample', ObserverExample);
ObserverExample.$inject= ['ObserverService', '$timeout'];

function ObserverExample(ObserverService, $timeout) {
  var vm = this;
  var id = 'vm1';

  ObserverService.attach(callbackFunction, 'let_me_know', id)

  function callbackFunction(params){
    console.log('now i know');
    ObserverService.detachByEvent('let_me_know')
  }

  $timeout(function(){
    ObserverService.notify('let_me_know');
  }, 5000);
}
```
Alternative way to remove event

```javascript
angular.module('app.controllers')
  .controller('ObserverExample', ObserverExample);
ObserverExample.$inject= ['ObserverService', '$timeout', '$scope'];

function ObserverExample(ObserverService, $timeout, $scope) {
  var vm = this;
  var id = 'vm1';
  ObserverService.attach(callbackFunction, 'let_me_know', id)

  function callbackFunction(params){
    console.log('now i know');
  }

  $timeout(function(){
    ObserverService.notify('let_me_know');
  }, 5000);

  // Cleanup listeners when this controller is destroyed
  $scope.$on('$destroy', function handler() {
    ObserverService.detachByEvent('let_me_know')
  });
}
```

## References

1. [Wikipedia](https://en.wikipedia.org/wiki). The source of all brief descriptions of the design patterns is wikipedia.
2. [AngularJS' documentation](https://docs.angularjs.org)
3. [AngularJS' git repository](https://github.com/angular/angular.js)
4. [Page Controller](http://msdn.microsoft.com/en-us/library/ff649595.aspx)
5. [Patterns of Enterprise Application Architecture (P of EAA)](http://martinfowler.com/books/eaa.html)
6. [Using Dependancy Injection to Avoid Singletons](http://googletesting.blogspot.com/2008/05/tott-using-dependancy-injection-to.html)
7. [Why would one use the Publish/Subscribe pattern (in JS/jQuery)?](https://stackoverflow.com/questions/13512949/why-would-one-use-the-publish-subscribe-pattern-in-js-jquery)
