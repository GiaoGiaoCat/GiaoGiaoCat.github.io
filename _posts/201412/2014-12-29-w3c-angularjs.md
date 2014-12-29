---
layout: post
title:  "W3CSchool AngularJS 教程阅读笔记"
date:   2014-12-27 16:30:00
categories: development
tags: learningnote javascript
author: "Victor"
---

### AngularJS 指令

* ``ng-app`` 指令初始化一个 AngularJS 应用程序。
* ``ng-init`` 指令初始化应用程序数据。一般不使用这个方法。
* ``ng-model`` 指令把应用程序数据绑定到 HTML 元素。
* ``ng-repeat`` 指令对于集合中（数组中）的每个项会 克隆一次 HTML 元素。
* ``ng-bind`` 指令把数据绑定到 HTML。使用 表达式 的方法。

#### 一些知识点

* ``ng-bind`` 和 ``{{ }}`` 表达式 的作用是一样的。
* ``ng-model`` 可以理解为双向数据绑定，或者说把变量绑定到 HTML 元素上。


### AngularJS 控制器

#### 基本介绍

* ``ng-controller`` 指令定义了应用程序控制器。
* 控制器是 JavaScript 对象，由标准的 JavaScript **对象的构造函数** 创建。
* 控制器的 ``$scope`` 是控制器所指向的应用程序 HTML 元素。

#### 特点

* 控制器可以把函数作为对象属性。
* 控制器可以带有方法。
* 在大型的应用程序中，通常是把控制器存储在外部文件中。

```html
<!DOCTYPE html>
<html>
<body>
  <div ng-app="" ng-controller="namesController">
    <ul>
      <li ng-repeat="x in names">
        {\{ x.name + ', ' + x.country }}
      </li>
    </ul>
    First Name：<input type="text" ng-model="person.firstName"><br>
    Last Name：<input type="text" ng-model="person.lastName"><br>
    Full Name：{\{person.firstName + " " + person.lastName}}<br>
    Full Name：{\{person.fullName}}<br>
    User Name：{\{userName()}}
  </div>
  <script src="namesController.js"></script>
  <script src="//www.w3cschool.cc/try/angularjs/1.2.5/angular.min.js"></script>
</body>
</html>
```

```javascript
//namesController.js
function namesController($scope) {
    $scope.names = [
        {name:'Jani',country:'Norway'},
        {name:'Hege',country:'Sweden'},
        {name:'Kai',country:'Denmark'}
    ];
    $scope.person = {
      firstName: "John",
      lastName: "Doe",
      fullName: function() {
            var x;
            x = $scope.person;
            return x.firstName + " " + x.lastName;
      }
    };
    $scope.userName = function() {
      var x;
      x = $scope.person;
      return x.firstName + " " + x.lastName;
    };
}
```

### AngularJS 过滤器

* 可用于转换和格式化数据。
* 可以通过一个管道字符（|）和一个过滤器添加到表达式中。
* 向指令添加过滤器。
* 过滤输入。

```html
<div ng-app="" ng-controller="costController">
数量：<input type="number" ng-model="quantity">
价格：<input type="number" ng-model="price">
<p>总价 = {\{ (quantity * price) | currency }}</p>
</div>
```

```html
<div ng-app="" ng-controller="namesController">
<p>向指令循环对象添加过滤器：</p>
<ul>
  <li ng-repeat="x in names | orderBy:'country'">
    {\{ x.name + ', ' + x.country }}
  </li>
</ul>
<div>
```

```html
<div ng-app="" ng-controller="namesController">
<p>输入过滤：</p>
<p><input type="text" ng-model="name"></p>
<ul>
  <li ng-repeat="x in names | filter:name | orderBy:'country'">
    {\{ (x.name | uppercase) + ', ' + x.country }}
  </li>
</ul>
</div>
```

### AngularJS XMLHttpRequest

* ``$http`` 是 AngularJS 中的一个核心服务，用于读取远程服务器的数据。
* ``$http.get(url)`` 是用于读取服务器数据的函数。

```json
[
  {
    "Name" : "Alfreds Futterkiste",
    "City" : "Berlin",
    "Country" : "Germany"
  },
  {
    "Name" : "Berglunds snabbköp",
    "City" : "Luleå",
    "Country" : "Sweden"
  }
]
```

```html
<div ng-app="" ng-controller="customersController">
<ul>
  <li ng-repeat="x in names">
    {\{ x.Name + ', ' + x.Country }}
  </li>
</ul>
</div>
<script>
function customersController($scope,$http) {
    $http.get("http://www.w3cschool.cc/try/angularjs/data/Customers_JSON.php")
    .success(function(response) {$scope.names = response;});
}
</script>
```

### AngularJS HTML DOM

* ``ng-disabled`` 指令直接绑定应用程序数据到 HTML 的 disabled 属性。
* ``ng-show`` 指令隐藏或显示一个 HTML 元素。
* 可以使用一个评估为 true or false 的表达式（比如 ng-show="hour < 12"）来隐藏和显示 HTML 元素。

```html
<div ng-app="">
<p><button ng-disabled="mySwitch">点我！</button></p>
<p><input type="checkbox" ng-model="mySwitch">按钮</p>
<p ng-show="true">我是可见的。</p>
<p ng-show="false">我是不可见的。</p>
</div>
```

### AngularJS HTML 事件

* ``ng-click`` 指令定义了一个 AngularJS 单击事件。

```html
<div ng-app="" ng-controller="myController">
<button ng-click="count = count + 1">点我！</button>
<p>{\{ count }}</p>
</div>
```

```html
<div ng-app="" ng-controller="personController">
<button ng-click="toggle()">隐藏/显示</button>
<p ng-hide="myVar">姓名: {{person.firstName + " " + person.lastName}}</p>
</div>
<script>
function personController($scope) {
    $scope.person = {
        firstName: "John",
        lastName: "Doe"
    };
    $scope.myVar = false;
    $scope.toggle = function() {
        $scope.myVar = !$scope.myVar;
    };
}
</script>
```

### AngularJS 模块

* 模块定义了应用程序。
* 所有的控制器都应该属于一个模块。
* 模块保持全局命名空间中的整洁。
* 所有之前的例子都是不好的实践，它们使用了全局函数和全局变量。

#### AngularJS 应用程序文件

* 应用程序至少应该有一个模块文件，一个控制器文件。

```javascript
// myApp.js 模块文件
var app = angular.module("myApp", []);

// myCtrl.js 控制器文件
app.controller("myCtrl", function($scope) {
    $scope.firstName = "John";
    $scope.lastName = "Doe";
});
```

```html
<!DOCTYPE html>
<html>
<body>
  <div ng-app="myApp" ng-controller="myCtrl">
    {\{ firstName + " " + lastName }}
  </div>
  <script src="//www.w3cschool.cc/try/angularjs/1.2.5/angular.min.js"></script>
  <script src="myApp.js"></script>
  <script src="myCtrl.js"></script>
</body>
</html>
```

### AngularJS 输入验证

* AngularJS 表单和控件可以验证输入的数据。

```html
<!DOCTYPE html>
<html>

<body>
<h2>Validation Example</h2>

<form  ng-app=""  ng-controller="validateCtrl"
name="myForm" novalidate>
<p>Username:<br>
  <input type="text" name="user" ng-model="user" required>
  <span style="color:red" ng-show="myForm.user.$dirty && myForm.user.$invalid"><span ng-show="myForm.user.$error.required">Username is required.</span></span>
</p>

<p>Email:<br>
  <input type="email" name="email" ng-model="email" required>
  <span style="color:red" ng-show="myForm.email.$dirty && myForm.email.$invalid">
  <span ng-show="myForm.email.$error.required">Email is required.</span>
  <span ng-show="myForm.email.$error.email">Invalid email address.</span>
  </span>
</p>

<p>
  <input type="submit" ng-disabled="myForm.user.$dirty && myForm.user.$invalid || myForm.email.$dirty && myForm.email.$invalid">
</p>
</form>

<script src="//apps.bdimg.com/libs/angular.js/1.2.15/angular.min.js"></script>
<script>
function validateCtrl($scope) {
    $scope.user = 'John Doe';
    $scope.email = 'john.doe@gmail.com';
}
</script>

</body>
</html>
```

### AngularJS Include

* ``ng-include`` 指令来包含 HTML 内容。

```html
<body>
  <div class="container">
    <div ng-include="'myUsers_List.htm'"></div>
    <div ng-include="'myUsers_Form.htm'"></div>
  </div>
</body>
```

### AngularJS 应用程序

```html
<div ng-app="myTodoApp" ng-controller="myTodoCtrl">
  <h2>我的笔记</h2>
    <p><textarea ng-model="message" cols="40" rows="10"></textarea></p>
  <p>
    <button ng-click="save()">保存</button>
    <button ng-click="clear()">清除</button>
  </p>

  <p>剩下的字符数：<span ng-bind="left()"></span></p>
</div>
<script src= "http://apps.bdimg.com/libs/angular.js/1.2.15/angular.min.js"></script>
<script src="myTodoApp.js"></script>
<script src="myTodoCtrl.js"></script>
```

```javascript
// 应用程序文件 myTodoApp.js
var app = angular.module("myTodoApp", []);

// 控制器文件 myTodoCtrl.js
app.controller("myTodoCtrl", function($scope) {
    $scope.message = "";
    $scope.left  = function() {return 100 - $scope.message.length;};
    $scope.clear = function() {$scope.message="";};
    $scope.save  = function() {$scope.message="";};
});
```
