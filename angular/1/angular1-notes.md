# Angular 1 Notes

See https://angularjs.org/ for details about AngularJS 1.

See https://docs.angularjs.org/guide for the developer's guide.

See https://docs.angularjs.org/api for the API reference.

## From Lynda.com's Learning AngularJS 1

See https://www.lynda.com/AngularJS-tutorials/Up-Running-AngularJS/154414-2.html
for details.

**What is AngularJS?**

* JavaScript framework
* For creating Single Page Applications (SPAs)

**Why Use AngularJS?**

* Advantages
  - Excellent templating engine
  - Handles DOM "masterfully"
  - Easy data manipulation
* MVC Architecture
  - Models
  - Views
  - Controllers
* Core Features
  - Directives
    + Basically, commands that start with "ng-"
  - Data binding
  - Dependency Injection
  - Filters
  - Modules
  - Routes
  - Services
  - Controllers
  
Angular uses an MV* (MV star) or an MVW (MV whatever) architecture.


### Controllers

**js/controllers.js**

```js
var myApp = angular.module('myApp', []);

myApp.controller('MyController', ['$scope', ...], function MyController($scope, ...) {
  // ...
});
```

**index.html**

```html
<!DOCTYPE html>
<html ng-app="myApp">
  <head>...</head>
  <body>
    <div ng-controller="MyController">
    </div>
  </body>
</html>
```

### ngRepeat Directive

```html
  <ul>
    <li ng-repeat="item in list">
      {{ item.name }}
    </li>
  </ul>
```

### ngSrc Directive

```html
  <img ng-src="images/{{ item.url_thumbnail }}" .../>
```