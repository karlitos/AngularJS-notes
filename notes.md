# Notes

##  Lectures

* [What’s wrong with Angular 1](https://medium.com/este-js-framework/whats-wrong-with-angular-js-97b0a787f903#.aoh5gvju3)
* [Why Does There Have to be Something Wrong with AngularJS?](https://johnpapa.net/why-does-there-have-to-be-something-wrong-with-angularjs/)
* [Why you should not use AngularJS](https://medium.com/@mnemon1ck/why-you-should-not-use-angularjs-1df5ddf6fc99#.qeaqka2qm)

## Q/A

* Round-Trip Web-Application
  * __A:__ Round-trip apps are apps that change pages on every get/post/put request. The round trip is the path the application takes from the browser, to the server, to the browser to display every change in state to the user.
* where to put the `<script>` tags - head/body ?
* where/how to link local resources/remote (CDN) resources ?
* JavaScript closures
* what is the __perent scope_ of an __isolated scope__ defined in a directive

## AngularJS

### Scopes

* Collection of variables and functions defined in specific context (accessible in one moment / place)
* There is always one root-scope
* scope inheritance - scope B can inherit from scope A and thus all the variables and functions from A are accessible in B
* variables and functions of the parent scope can be overridden in the child scope
* you can still access parents variables and functions of the pare scope, but only indirectly __ HOW ?? __
* it is possible to define _isolated scopes_ - mostly applied in _directive definitions_ where the accessibility of variables and functions can be precisely defined
* allows creating enclosed, reusable components, prevents collisions/overriding of variables

### Inversion control - Dependency injection
* defining the dependencies with parameters, the resolution and initialization happens to the runtime and is done by the framework/environment
* the opposite of the _dependency injection_ is _hard-coding_ of the dependencies in the code

### Building blocks

#### Modules

* most coarse-grained Angular structure
* encapsulates a collection of common, related components
* container for:
  * Controller
  * Services
  * Directives
  * Filter
* aggregation meaningful done by module-responsibility/assignment _e.g. user-management_

#### Controller
* the business-logic of the _scopes_ for a certain view
* controller should be kept _simple_, complex logic should be encapsulated in _Services_
* controller should not _manipulate DOM_, this should be done in _directives_

#### Models
* plain JavaScript data types
* 2-way data binding work only, when models are defined in _scopes_

`````` javascript
var app = angular.module('myApp');

app.controller('MyCtrl', function($scope) {
  $scope.user = {// simple JS Object
    name: 'John Doe',
    age: 27,
    email: 'john@doe.com'
  };

  $scope.phones = [ ...] // array
});
``````

#### Routes
* _deep linking_  use of a hyperlink that links to a specific, generally searchable or indexed, piece of web content on a website
* linking between the URL and related _controller/template_

`````` javascript
var app = angular.module('myApp');

app.config(function($routeProvider) {
  $routeProvider
    .when('/', {
      templateUrl: './templates/mainTemplate.html'
      controller: 'MainCtrl'
    })
    .when('/user', {
      templateUrl: './templates/userTemplate.html'
      controller: 'UserCtrl'
    })
    .otherwise({redirectTo: '/'
  });
});
``````

#### Templates/Views
* _templates_ contain _HTML_, _directives_ and _expressions_ which will be interpreted by the browser
* _views_ are instantiated _templates_

#### Expressions
* access to the data defined in a particular _scope_
* the _{{ }}_ thing
* if the result of the evaluation is _null_ or _undefined_ there is no output
* you can not use _loops_ or _control structures_ (if/else)
* you can use additional _filter_

#### Services
* thin controller, fat service
* each service will has at most one instance -> _singleton pattern_
* there are 5 ways to define a service
  * __Provider__ - general service definition API, has to contain a __$get()__ function which returns the instance of the service
  * __Factory__ - build on top of the __Provider__ without broad configuration possibilities
  * __constructor__ service(...) - allows to register an object of an already declared Class as a Service
  * __value as a service__ value(...) - allows to register a value as a Service, the value can be primitive data-type, Object or a function
  * __constant as a service__ constant(...) - allows to register a constant value as a Service, which value cannot be changed during the runtime

##### Provider configuration

``````javascript
angular.module('myApp').provider('logger', function(){
  // private
  var prefix = '';

  //  logger implementation
  var log = function(level, message){
    console.log(prefix + '[' + level + ']' + message);
  };
  // public configuration method
  this.setPrefix = function(newPrefix){
    prefix = newPrefix;
  }

  this.$get = function(){
    return {
      error: function(message){
        log('ERROR', message)
      },
      debug: function(message){
        log('DEBUG', message)
      }
    }
  };
});

...
//confoguration
angular.module('myApp').config(function(loggerProvider){
  loggerProvider.setPrefix('|=> ... ')
});

...
// dependency injection
angular.module('myApp').controller('myCtrl', function(logger){
  logger.debug('Some debugging message');
});
``````
* The _config()_ method will be called with the __specified service name__ (e.g. 'logger') together with the 'Provider' suffix => _loggerProvider_

#### Directives
* allow creating of new, re-usable components/HTML elements
* directives are created with the __directive('someName', function(){...})__ method with two parameters:
  * name of the directive
  * so called factory function
* naming convention:
  * declaration: _camelCase_ (opacityAndColorPicker)
  * in HTML: _snake-case_ (opacity-and-color-picker)
* the 'factory function' has to return either an function - _postLink -Function_ or an _directive-definition-object (DDO)_

##### postLink-Function
* it gets at least 3 parameters:
  * scope - reference to the scope valid for this directive
  * element - reference to the DOM element this directive affects
  * attrs - reference to the object allowing us to access the HTML-attributes of the DOM elements
* when returning only the _postLink -Function_ all directive properties will have the default values

``````javascript
var directiveDefinitionObject = {
   priority: 0,
   template: '<div></div>', // or // function(tElement, tAttrs) { ... },
   // or templateUrl: 'directive.html', // or // function(tElement, tAttrs) { ... },
   transclude: false,
   restrict: 'A',
   templateNamespace: 'html',
   scope: false,
   controller: function($scope, $element, $attrs, $transclude, otherInjectables) { ... },
   controllerAs: 'stringIdentifier',
   bindToController: false,
   require: 'siblingDirectiveName', // or // ['^parentDirectiveName', '?optionalDirectiveName', '?^optionalParent'],
   compile: function compile(tElement, tAttrs, transclude) {
     return {
       pre: function preLink(scope, iElement, iAttrs, controller) { ... },
       post: function postLink(scope, iElement, iAttrs, controller) { ... }
     }
     // or // return function postLink( ... ) { ... }
   },
   // or // link: {
   //  pre: function preLink(scope, iElement, iAttrs, controller) { ... },
   //  post: function postLink(scope, iElement, iAttrs, controller) { ... }
   // }
   // or // link: function postLink( ... ) { ... }
 };
``````
* this means that the directive could be applied as a property of a '<div>' element by default
> __Best Practice:__ It's recommended to use the "directive definition object" form.

##### directive-definition-object (DDO)
* an Object with attributes defining the directive
  * __restrict__ - defines the area of validity/applicability of the directive
    * E - Element name (default): ``<my-directive></my-directive>``
    * A - Attribute (__default__): ``<div my-directive="exp"></div>``
    * C - Class: ``<div class="my-directive: exp;"></div>``
    * M - Comment: ``<!-- directive: my-directive exp -->``
  * __template__ - for short 1-2 lines long template description
  * __templateUrl__ - linkage to more complex descriptions
  * the templates are HTML markup that may:
    * eplace the contents of the directive's element (default).
    * Wrap the contents of the directive's element (if transclude is true)
  * __scope__ - the scope of the directive, can be true, an object or a falsy value
    * __falsy__: No scope will be created, parent's scope will be used
    * __true__: A new child scope that prototypically inherits from its parent will be created
    * __{...}__ (an object hash): A new "isolate" scope is created for the directive's element.
    * Following explanation is borrowed from the [AngularJs wiki](https://github.com/angular/angular.js/wiki/Understanding-Scopes):

    ...directives often need access to a few parent scope properties. The object hash is used to set up two-way binding (using '=') or one-way binding (using '@') between the parent scope and the isolate scope. There is also '&' to bind to parent scope expressions. So, these all create local scope properties that are derived from the parent scope. Note that attributes are used to help set up the binding -- you can't just reference parent scope property names in the object hash, you have to use an attribute. E.g., this won't work if you want to bind to parent property `parentProp` in the isolated scope: `<div my-directive>` and `scope: { localProp: '@parentProp' }`. An attribute must be used to specify each parent property that the directive wants to bind to: `<div my-directive the-Parent-Prop=parentProp>` and `scope: { localProp: '@theParentProp' }`.

     ...use `attrs.$observe('attr_name', function(value) { ... })` in the linking function to get the interpolated value of isolate scope properties that use the '@' notation.  E.g., if we have this in the linking function -- `attrs.$observe('interpolated', function(value) { ... })` -- `value` would be set to 11.  (`scope.interpolatedProp` is undefined in the linking function.  In contrast, `scope.twowayBindingProp` is defined in the linking function, since it uses the '=' notation.)
     * passing parameters to the expressions from the _parent scope_ has to be done with so called __parameter object__. It is a map of local variable names and values into the expression wrapper fn. For example, if the expression is `increment(amount)` then we can specify the __amount__ value by calling the _localFn_ as `localFn({amount: 22})`.

  * __link__ - It is called for every __directive instance__! exactly once. This property is used only if the compile property is not defined. The link function is responsible for registering DOM listeners as well as updating the DOM. It is executed after the template has been cloned. It takes 3-4 parameters, depending on whether the __require__ property is set:
    * __scope__ - The scope to be used by the directive
    * __iElement__ - instance element - The element where the directive is to be used.
    * __iAttrs__ - instance attributes - List of attributes declared on this element shared between all directive linking functions.
    * __controller__ - the directive's required controller instance(s)
  * __compile__ - It is called for every __directive__ (not instance)! exactly once. The compile function deals with transforming the template DOM. It takes the following arguments:
    * __tElement__ - template element - The element where the directive has been declared.
    * __tAttrs__ - template attributes - List of attributes declared on this element shared between all directive compile functions.
  * __require__ - Require another directive and inject its controller as the fourth argument to the linking function.

#### Components
* Component helper introduced in _Angular 1.5.0_ allowing more easier way of writing self-contained reusable elements as when using directives
* Configuration overview:

|Name|Description|Default|
|----|-----------|-------|
|bindings|Define DOM attribute binding to component properties.|{ }|
|template|Template as string|‘ ‘ (Empty String)|
|templateUrl|Template via URL|	undefined|
|transclude|Access innerHTML|true|
|controller|Define a constructor function|function(){ }|
|controllerAs|An identifier name for a reference to the controller.|$ctrl|
* The __Component helper__ is a wrapper for the __directive__ function and adds some sugar on top
* Comparison
``````javascript
// AngularJS 1.4.x way to create a component
  angular.module('X')
      .directive('myComponent', function(){
        return {
          scope: {},
          template: '<div>{{$ctrl.data.name}}</div>',
          bindToController: {
            data: '='
          },
          controller: function(){},
          controllerAs: '$ctrl'
        }
      });      
// AngularJS 1.5.x way to create a component
  angular.module('X')
    .component('myComponent',{
      template:'<div>{{$ctrl.data.name}}</div>',
      bindings: {
        data: '='
      }
    });
``````


### Practical usage

#### ng-bind ng-bind-template ng-href
Instead of __expressions__ inside of DOM elements the __ng-bind__ directive in place of the element attribute can be used.

``````html
<h2>{{ book.title }}</h2>
<h3>{{ book.subtitle }}</h3>


<h2 ng-bind="book.title"></h2>
<h3 ng-bind="book.subtitle"></h3>
``````
This bring several improvements:
1. Prohibits showing expressions `{{ ... }}` on the page if they has not been evaluated yet (because of slow application/server)
2. Using expressions leads to creation of a new DOM elements every time they are evaluated. __ng-bind__ just updates the text/value of the DOM element, which leads to performance improvement and reduces the memory consumption.

__ng-bind-template__ allows combining of expressions and plaintext together in an single element. It allows also the usage of more expressions in one element.
``````html
<li ng-bind-template="Seiten: {{ book.numPages }}"></li>
<li ng-bind-template="Autor: {{ book.author }} ISBN: {{ book.isbn }}"></li>
``````
The __ng-href__ directive maks creation opf links more easy, by setting the hyperlink first then, when the expression its hold is evaluated.
``````html
<a ng-bind="book.publisher.name" ng-href="{{ book.publisher.url }}"  target="_blank"</a>
``````

### Unit tests - Jasmine
#### ngMock
* The ngMock module provides support to inject and mock Angular services into unit tests. In addition, ngMock also extends various core ng services such that they can be inspected and controlled in a synchronous manner within test code.
* The __module__ function registers a module configuration code. It collects the configuration information which will be used when the injector is created by inject.

``````javascript
// load the application module
    beforeEach(module('myModule'));
``````
* The __inject__ function wraps a function into an injectable function. The inject() creates new instance of $injector per test, which is then used for resolving references.
* To help with this, the injected parameters can, optionally, be enclosed with underscores. These are ignored by the injector when the reference name is resolved

``````javascript
// Defined out reference variable outside
var myService;

// Wrap the parameter in underscores
beforeEach( inject( function(_myService_){
  myService = _myService_;
}));

// Use myService in a series of tests.
it('makes use of myService', function() {
  myService.doStuff();
});
``````
## JS

###  Self-Invoking Functions
* A self-invoking expression is invoked (started) automatically, without being called.
* Function expressions will execute automatically if the expression is followed by ().
`````` javascript
(function () {
    var x = "Hello!!";      // I will invoke myself
})();
``````

### JavaScript Closures
* A closure is a function having access to the parent scope, even after the parent function has closed.

`````` javascript
var add = (function () {
    var counter = 0;
    return function () {return counter += 1;}
})();

add();
add();
add(); // the counter is now 3
``````
* _add_ is assigned the return value of a self-invoking function
* _add_ becomes a function =>  _return_ returns a function expression
*  it can access the counter in the parent scope.

### use strict
"use strict" defines that JavaScript code should be executed in _strict mode_. It is not a statement, but a literal expression, ignored by earlier versions of JavaScript.

The purpose of "use strict" is to indicate that the code should be executed in _strict mode_. With strict mode, you can not, for example, use undeclared variables.

#### Not Allowed in Strict Mode
* Using a variable or an object, without declaring it, is not allowed.
``````JavaScript
"use strict";
x = 3.14;                // This will cause an error (x is not defined)
x = {p1:10, p2:20};      // This will cause an error (x is not defined)
``````
* Deleting a variable (or object) or a function is not allowed.
``````JavaScript
"use strict";
var x = 3.14;
delete x;                // This will cause an error
function x(p1, p2) {};
delete x;                // This will cause an error
``````
* Duplicating a parameter name is not allowed.
``````JavaScript
"use strict";
function x(p1, p1) {};   // This will cause an error
``````
* Octal numeric literals and escape characters are not allowed.
``````JavaScript
"use strict";
var x = 010;             // This will cause an error
var x = \010;            // This will cause an error
``````
* Writing to a read-only property is not allowed.
* Writing to a get-only property is not allowed.
* Deleting an undeletable property is not allowed.
* The string "eval" cannot be used as a variable.
* The string "arguments" cannot be used as a variable.
* The with statement is not allowed.
* eval() is not allowed to create variables in the scope from which it was called
