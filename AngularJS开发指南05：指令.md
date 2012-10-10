Directives are a way to teach HTML new tricks. **指令**使我们用来扩展浏览器能力的技术之一。During DOM compilation directives are matched against the HTML and executed. This allows directives to register behavior, or transform the DOM.在DOM编译期间，和HTML关联着的指令会被检测到，并且被执行。这使得指令可以为DOM指定行为，或者改变它。

Angular comes with a built in set of directives which are useful for building web applications but can be extended such that HTML can be turned into a declarative domain specific language (DSL).AngularJS有一套完整的、可扩展的、用来帮助web应用开发的指令集，它使得HTML可以转变成“**特定领域语言(DSL)**”。

## Invoking directives from HTML 从HTML中调用指令
Directives have camel cased names such as ngBind.指令遵循驼峰式命名，如`ngBind`。 The directive can be invoked by translating the camel case name into snake case with these special characters :, -, or _.指令可以通过使用指定符号转化成链式风格的的名称来调用，特定符号包括 `:` ,`-`,`_`。 Optionally the directive can be prefixed with x-, or data- to make it HTML validator compliant. 你好可以选择给指令加上前缀，比如“x-”，“data-”来让它符合html的验证规则。，Here is a list of some of the possible directive names: ng:bind, ng-bind, ng_bind, x-ng-bind and data-ng-bind.这里有以下可以用的指令名称例子：ng:bind, ng-bind, ng_bind, x-ng-bind , data-ng-bind。

The directives can be placed in element names, attributes, class names, as well as comments. Here are some equivalent examples of invoking myDir. (However, most directives are restricted to attribute only.)指令可以做为元素名，属性名，类名，或者注释。下面是一些等效调用myDir指令的例子：

	<span my-dir="exp"></span>
	<span class="my-dir: exp;"></span>
	<my-dir></my-dir>
	<!-- directive: my-dir exp -->

Directives can be invoked in many different ways, but are equivalent in the end result as shown in the following example.指令可以通过很多不同的方式调用，但最后结果都是等效的。下例中对此做了演示。

### Source 
index.html

	<!doctype html>
	<html ng-app>
	  <head>
	    <script src="http://code.angularjs.org/angular-1.1.0.min.js"></script>
	    <script src="script.js"></script>
	  </head>
	  <body>
	    <div ng-controller="Ctrl1">
	      Hello <input ng-model='name'> <hr/>
	      &ltspan ng:bind="name"&gt <span ng:bind="name"></span> <br/>
	      &ltspan ng_bind="name"&gt <span ng_bind="name"></span> <br/>
	      &ltspan ng-bind="name"&gt <span ng-bind="name"></span> <br/>
	      &ltspan data-ng-bind="name"&gt <span data-ng-bind="name"></span> <br/>
	      &ltspan x-ng-bind="name"&gt <span x-ng-bind="name"></span> <br/>
	    </div>
	  </body>
	</html>

script.js:

	function Ctrl1($scope) {
	  $scope.name = 'angular';
	}

End to end test:


	it('should show off bindings', function() {
	  expect(element('div[ng-controller="Ctrl1"] span[ng-bind]').text()).toBe('angular');
	});


## 字符串替换式
During the compilation process the compiler matches text and attributes using the $interpolate service to see if they contain embedded expressions. 在编译期间，编译器会用$interpolate服务去检查文本中是否嵌入了表达式。These expressions are registered as watches and will update as part of normal digest cycle.这个表达式会被当成一个监视器一样注册，并且在 作为$digest循环中的一部分，它会自动更新。 An example of interpolation is shown here:下面是一个替换式的例子：

	<img src="img/{{username}}.jpg">Hello {{username}}!</img>

## Compilation process, and directive matching 编译过程和指令匹配
Compilation of HTML happens in three phases:HTML的编译分为三个阶段：

1.  First the HTML is parsed into DOM using the standard browser API.首先浏览器会用它的标准API将HTML解析成DOM。 This is important to realize because the templates must be parsable HTML. 你需要认清这一点，因为我们的模板必须是可被解析的HTML。This is in contrast to most templating systems that operate on strings, rather than on DOM elements.这是AngularJS和那些“以字符串为基础而非以DOM元素为基础的”模板系统的区别之处。

2.  The compilation of the DOM is performed by the call to the $compile() method.DOM的编译是有$compile方法来执行的。 The method traverses the DOM and matches the directives. 这个方法会遍历DOM并找到匹配的指令。If a match is found it is added to the list of directives associated with the given DOM element.一旦找到一个，它就会被加入一个指令列表中，这个列表是用来记录所有和当前DOM相关的指令的。 Once all directives for a given DOM element have been identified they are sorted by priority and their compile() functions are executed.一旦所有的指令都被确定了，会按照优先级被排序，并且他们的 compile方法会被调用。 The directive compile function has a chance to modify the DOM structure and is responsible for producing a link() function explained next. 指令的compile函数能修改DOM结构，并且要负责生成一个link()函数（后面会提到）。The $compile() method returns a combined linking function, $compile方法最后返回一个合并起来的链接函数，which is a collection of all of the linking functions returned from the individual directive compile functions.这是链接函数是每一个指令的compile函数返回的链接函数的集合。

3.  Link the template with scope by calling the linking function returned from the previous step. 通过调用一步所说的链接函数来将模板与作用域链接起来。This in turn will call the linking function of the individual directives allowing them to register any listeners on the elements and set up any watches with the scope.这会轮流调用每一个指令的链接函数，让每一个指令都能对DOM注册监听事件，和建立对作用域的的监听。 The result of this is a live binding between the scope and the DOM.这样最后就形成了作用域的DOM的动态绑定。 A change in the scope is reflected in the DOM.任何一个作用域的改变都会在DOM上体现出来。

	var $compile = ...; // injected into your code
	var scope = ...;
	 
	var html = '<div ng-bind='exp'></div>';
	 
	// Step 1: parse HTML into DOM element
	var template = angular.element(html);
	 
	// Step 2: compile the template
	var linkFn = $compile(template);
	 
	// Step 3: link the compiled template with the scope.
	linkFn(scope);

###Reasons behind the compile/link separation 编译和链接分离的合理性分析
At this point you may wonder why the compile process is broken down to a compile and link phase. 你可能会疑惑为什么编译过程和链接过程要分离。To understand this, let's look at a real world example with repeater:要明白其中的原因，你可以先看下面这个带有“重复指令”的例子：

	Hello {{user}}, you have these actions:
	<ul>
	  <li ng-repeat="action in user.actions">
	    {{action.description}}
	  </li>
	</ul>

The short answer is that compile and link separation is needed any time a change in model causes a change in DOM structure such as in repeaters.简短的说，编译和链接的分离是模型和DOM结构能够动态关联的一种需要。

When the above example is compiled, the compiler visits every node and looks for directives. 当上面的例子被编译后，编译器会遍历所有节点来寻找指令。The {{user}} is an example of an interpolation directive.例如{{user}}是一个替换式指令。 ngRepeat is another directive.`ngRepeat`是另一个指令。 But ngRepeat has a dilemma.但是`ngRepeat`有一个难题。 It needs to be able to quickly stamp out new lis for every action in user.actions.他需要为`user.actions`中的每一个`action` 构造一个li。This means that it needs to save a clean copy of the li element for cloning purposes and as new actions are inserted, the template li element needs to be cloned and inserted into ul. 这以为着它先要保存一个“干净”的li元素来用作克隆，然后等新的action插入进来时，克隆一个li并插入到ul中。But cloning the li element is not enough.但是仅仅克隆li的话工作还没完。 It also needs to compile the li so that its directives such as {{action.descriptions}} evaluate against the right scope.他还需要编译这个li才能把其中的像是{{action.descriptions}}的替换式替换成相应作用域下的值。 A naive method would be to simply insert a copy of the li element and then compile it. 我们可以用一个简单地方法来克隆和插入li元素然后编译它。But compiling on every li element clone would be slow,但是要编译每一个li的话，使用克隆会速度很慢， since the compilation requires that we traverse the DOM tree and look for directives and execute them.因为编译的工程需要我们遍历DOM树，并找到对应的指令并执行它们。 If we put the compilation inside a repeater which needs to unroll 100 items we would quickly run into performance problems.如果我们在一个需要循环100次循环体内执行编译的话，性能问题就会马上凸现出来。

The solution is to break the compilation process into two phases;而我们的解决方案就是将编译工程分为两个阶段。 the compile phase where all of the directives are identified and sorted by priority, 编译阶段将指令识别出来并按优先级排序。and a linking phase where any work which links a specific instance of the scope and the specific instance of an li is performed.编译阶段将作用域中的实例和li进行链接。

ngRepeat works by preventing the compilation process form descending into the li element. `ngRepeat` 会阻止li子元素的编译。Instead the ngRepeat directive compiles li separately.取而代之的是 `ngRepeat`指令会单独对li进行编译。 The result of of the li element compilation is a linking function which contains all of the directives contained in the li element, ready to be attached to a specific clone of the li element.这个编译结束后会生成一个链接函数，这个函数包含了准备li元素上的所有指令，并等待被绑定到相应克隆出来的li元素上。 At runtime the ngRepeat watches the expression and as items are added to the array it clones the li element, creates a new scope for the cloned li element and calls the link function on the cloned li.在执行期，`ngRepeat`之指令会监视表达式，当有新的元素增加到对应的数组之后，它就会新克隆一个li元素，为它创建一个新作用域，并使用链接函数把它和对应作用域链接上。

Summary:总结：

compile function编译函数 - The compile function is relatively rare in directives,编译函数在指令中是很少的， since most directives are concerned with working with a specific DOM element instance rather than transforming the template DOM element.因为大部分指令都只是为了处理相应的DOM元素实例，而不是改变模板DOM元素。 Any operation which can be shared among the instance of directives should be moved to the compile function for performance reasons.考虑到性能问题，任何指令的实例见能被共享的操作都应该移到编译函数中。

link function链接函数 - It is rare for the directive not to have a link function. 指令很少不带有链接函数，A link function allows the directive to register listeners to the specific cloned DOM element instance as well as to copy content into the DOM from the scope.链接函数可以让指令对相应克隆元素注册事件，还可以将作用域中的内容复制到DOM中。


## Writing directives如何写指令(短版) (short version)
In this example we will build a directive that displays the current time.
下面这个例子演示一个获取当前时间的指令。

## Source

index.html:

	<!doctype html>
	<html ng-app="time">
	  <head>
	    <script src="http://code.angularjs.org/angular-1.1.0.min.js"></script>
	    <script src="script.js"></script>
	  </head>
	  <body>
	    <div ng-controller="Ctrl2">
	      Date format: <input ng-model="format"> <hr/>
	      Current time is: <span my-current-time="format"></span>
	    </div>
	  </body>
	</html>

script.js:

	function Ctrl2($scope) {
	  $scope.format = 'M/d/yy h:mm:ss a';
	}
	 
	angular.module('time', [])
	  // Register the 'myCurrentTime' directive factory method.
	  // We inject $timeout and dateFilter service since the factory method is DI.
	  .directive('myCurrentTime', function($timeout, dateFilter) {
	    // return the directive link function. (compile function not needed)
	    return function(scope, element, attrs) {
	      var format,  // date format
	          timeoutId; // timeoutId, so that we can cancel the time updates
	 
	      // used to update the UI
	      function updateTime() {
	        element.text(dateFilter(new Date(), format));
	      }
	 
	      // watch the expression, and update the UI on change.
	      scope.$watch(attrs.myCurrentTime, function(value) {
	        format = value;
	        updateTime();
	      });
	 
	      // schedule update in one second
	      function updateLater() {
	        // save the timeoutId for canceling
	        timeoutId = $timeout(function() {
	          updateTime(); // update DOM
	          updateLater(); // schedule another update
	        }, 1000);
	      }
	 
	      // listen on DOM destroy (removal) event, and cancel the next UI update
	      // to prevent updating time ofter the DOM element was removed.
	      element.bind('$destroy', function() {
	        $timeout.cancel(timeoutId);
	      });
	 
	      updateLater(); // kick off the UI update process.
	    }
	  });


### Demo

## Writing directives 如何写指令(长版) (long version)
An example skeleton of the directive is shown here, for the complete list see below.
下面是写一个完整的指令的例子。

	var myModule = angular.module(...);
	 
	myModule.directive('directiveName', function factory(injectables) {
	  var directiveDefinitionObject = {
	    priority: 0,
	    template: '<div></div>',
	    templateUrl: 'directive.html',
	    replace: false,
	    transclude: false,
	    restrict: 'A',
	    scope: false,
	    compile: function compile(tElement, tAttrs, transclude) {
	      return {
	        pre: function preLink(scope, iElement, iAttrs, controller) { ... },
	        post: function postLink(scope, iElement, iAttrs, controller) { ... }
	      }
	    },
	    link: function postLink(scope, iElement, iAttrs) { ... }
	  };
	  return directiveDefinitionObject;
	});


In most cases you will not need such fine control and so the above can be simplified.大部分情况下你不需要控制这么多细节。 All of the different parts of this skeleton are explained in following sections.这其中的具体内容下面会有完整的解释。 In this section we are interested only isomers of this skeleton.这一节里我们只关注大部分指令相同的地方。

The first step in simplyfing the code is to rely on the default values. Therefore the above can be simplified as:
要简化上面的代码，我们首先要依赖一下基本选项的默认值。使用默认值的话，上面的代码可以简化成：

	var myModule = angular.module(...);
	 
	myModule.directive('directiveName', function factory(injectables) {
	  var directiveDefinitionObject = {
	    compile: function compile(tElement, tAttrs) {
	      return function postLink(scope, iElement, iAttrs) { ... }
	    }
	  };
	  return directiveDefinitionObject;
	});

Most directives concern themselves only with instances, not with template transformations, allowing further simplification:大部分指令只关心实例，并不需要将模板进行变形，所以我们还可以简化：

	var myModule = angular.module(...);
	 
	myModule.directive('directiveName', function factory(injectables) {
	  return function postLink(scope, iElement, iAttrs) { ... }
	});

### Factory method工厂函数
The factory method is responsible for creating the directive.工厂函数是用来创建指令的。 It is invoked only once,它只会被调用一次。 when the compiler matches the directive for the first time.就是当编译器第一次匹配到相应指令的时候。 You can perform any initialization work here.你可以在其中进行任何初始化的工作。 The method is invoked using the $injector.invoke which makes it injectable following all of the rules of injection annotation.调用它时使用的是 `$injector.invoke` ， 所以它遵循所有注入器的规则。

###Directive Definition Object 指令定义对象

The directive definition object provides instructions to the compiler. The attributes are:
指令定义对象给编译器提供了生成指令需要的细节。这个对象的属性有：
*  name名称 - Name of the current scope.当前作用域的名称。 Optional defaults to the name at registration.在注册是可选的。

*  priority优先级 - When there are multiple directives defined on a single DOM element, sometimes it is necessary to specify the order in which the directives are applied.当一个DOM上有多个指令时，有会需要指定指令执行的顺序。 The priority is used to sort the directives before their compile functions get called. 这个**优先级**就是用来在执行指令的compile函数前先排序的。Higher priority goes first.高优先级的先执行。 The order of directives within the same priority is undefined.相同优先级的指令顺序没有被指定谁先执行。

*  terminal终点 - If set to true then the current priority will be the last set of directives which will execute (any directives at the current priority will still execute as the order of execution on same priority is undefined).如果被设置为true，那么该指令就会在同一个DOM的指令集和中最后被执行。任何其他“terminal”的指令也仍然会执行，因为同级的指令顺序是没有被定义的。

*  scope 作用域- If set to:如果被定义成：

	*  true - then a new scope will be created for this directive.那么就会为当前指令创建一个新的作用域。 If multiple directives on the same element request new scope, only one new scope is created.如果有多个在同一个DOM上的指令要求创建新作用域，那么只有一个新的会被创建。 The new scope rule does not apply for the root of the template since the root of the template always gets a new scope.这一创建新作用域的规则不适用于模板的根节点，因为模板的根节点总是会得到一个新的作用域。

	*  {} (object hash)对象哈希 - then a new 'isolate' scope is created.那么一个新的“孤立的”作用域就会被创建。 The 'isolate' scope differs from normal scope in that it does not prototypically inherit from the parent scope.这个“孤立的”作用域区别于一般作用域的地方在于，它不会以原型继承的方式直接继承自父作用域。 This is useful when creating reusable components, 这对于创建可重用的组件是非常有用的，因为可重用的组件一般不应该读或写父作用域的数据。which should not accidentally read or modify data in the parent scope. 
	The 'isolate' scope takes an object hash which defines a set of local scope properties derived from the parent scope. 这个“孤立的”作用域使用一个对象哈希来表示，这个哈希定义了一系列本地作用域属性， 这些本地作用域属性是从父作用域中衍生出来的。These local properties are useful for aliasing values for templates. 这些属性主要用来分析模板的值。Locals definition is a hash of local scope property to its source:这个哈希的键值对是本地属性为键，它的来源为值。

		*  @ 或 @attr - bind a local scope property to the DOM attribute.将本地作用域成员成员和DOM属性绑定。 The result is always a string since DOM attributes are strings.绑定结果总是一个字符串，因为DOM的属性就是字符串。 If no attr name is specified then the local name and attribute name are same.如果DOM属性的名字没有被指定，那么就和本地属性名一样。 Given <widget my-attr="hello {{name}}"> and widget definition of scope: { localName:'@myAttr' }, then widget scope property localName will reflect the interpolated value of hello {{name}}.比如说<widget my-attr="hello {{name}}"> 和作用域对象: { localName:'@myAttr' }。 As the name attribute changes so will the localName property on the widget scope. 当`name`值改变的时候， 作用域中的LocalName也会改变。The name is read from the parent scope (not component scope).这个`name`是从父作用域中读来的（而不是组件作用域）。
		
		*  = 或 =expression(表达式) - set up bi-directional binding between a local scope property and the parent scope property.在本地作用域属性和父作用域属性间建立一个双向的绑定。 If no attr name is specified then the local name and attribute name are same.如果没有指定父作用域属性名称，那就和本地名称一样。 比如 <widget my-attr="parentModel"> 和作用域对象: { localModel:'=myAttr' }, then widget scope property localName will reflect the value of parentModel on the parent scope. Any changes to parentModel will be reflected in localModel and any changes in localModel will reflect in parentModel。本地属性`localModel`会反映父作用域中`parentModel`的值。localModel和parentModel的任一方改变都会影响对方。
		
		*  & 或 &attr - provides a way to execute an expression in the context of the parent scope.提供了一种能在父作用域下执行表达式的方法。 If no attr name is specified then the local name and attribute name are same.如果没有指定父作用域属性名称，那就和本地名称一样。 比如 <widget my-attr="count = count + value"> 和作用域对象：{ localFn:'increment()' }, then isolate scope property localFn will point to a function wrapper for the increment() expression. 本地作用域成员`localFn`会指向一个`increment`表达式的函数包装。Often it's desirable to pass data from the isolate scope via an expression and to the parent scope,通常你可以通过这个表达式从本地作用域给父作用域传值， this can be done by passing a map of local variable names and values into the expression wrapper fn. 操作方法是将本地变量名和值得对应关系传给这个表达式的包装函数。For example, if the expression is increment(amount) then we can specify the amount value by calling the localFn as localFn({amount: 22}).比如说，这个表达式是increment(amount)，那么你就可以用调用`localFn({amount:22})`的方式指定amount的值。

*  **controller** - Controller constructor function.控制器的构造对象。 The controller is instantiated before the pre-linking phase and it is shared with other directives if they request it by name (see require attribute).这个控制器函数是在预编译阶段被执行的，并且它是共享的，其他指令可以通过它的名字得到（参考依赖属性（require attribute））。 This allows the directives to communicate with each other and augment each other behavior.这就使得指令间可以互相交流来扩大自己的能力。 The controller is injectable with the following locals:会传递给这个函数的参数有：

	*  $scope - Current scope associated with the element当前元素关联的作用域。
	*  $element - Current element当前元素
	*  $attrs - Current attributes obeject for the element 当前元素的属性对象。
	*  $transclude - A transclude linking function pre-bound to the correct transclusion scope:function(cloneLinkingFn).

*  require - Require another controller be passed into current directive linking function.请求将另一个控制器作为参数传入到当前链接函数。 The require takes a name of the directive controller to pass in.这个请求需要传递被请求指令的控制器的名字。 If no such controller can be found an error is raised.如果没有找到，就会触发一个**错误**。 The name can be prefixed with:请求的名字可以加上下面两个前缀：
	
	*  ? - Don't raise an error. This makes the require dependency optional.不要触发错误，这只是一个可选的请求。
	*  ^ - Look for the controller on parent elements as well.没找到的话，在父元素的controller里面也查找有没有。

*  restrict - String of subset of EACM which restricts the directive to a specific directive declaration style. If omitted directives are allowed on attributes only.EACM中的任意一个之母。它是用来限制指令的声明格式的。如果没有这一项。那就只允许使用属性形式的指令。

	*  E - Element name: <my-directive></my-directive>
	*  A - Attribute: <div my-directive="exp"> </div>
	*  C - Class: <div class="my-directive: exp;"></div>
	*  M - Comment: <!-- directive: my-directive exp -->


*  template - replace the current element with the contents of the HTML.将当前的元素替换掉。 The replacement process migrates all of the attributes / classes from the old element to the new one.这个替换过程会自动将元素的属性和css类名添加到新元素上。更多细节请查考章节“创建widgets”。 See Creating Widgets section below for more information.

templateUrl - Same as template but the template is loaded from the specified URL.和template属性一样，只不过这里指示的是一个模板的URL。 Because the template loading is asynchronous the compilation/linking is suspended until the template is loaded.因为模板加载是异步的，所有编译和链接都会等到加载完成后再执行。

*  replace - if set to true then the template will replace the current element, rather than append the template to the element.如果被设置成true那么现在的元素会被模板替换，而不是被插入到元素中。

*  transclude - compile the content of the element and make it available to the directive.将元素编译好，使得指令可以开始使用它。 Typically used with ngTransclude.一般情况下需要和ngTransclude指令一起使用。 The advantage of transclusion is that the linking function receives a transclusion function which is pre-bound to the correct scope.使用嵌入的好处在于链接好书可以获取到预绑定在作用域上的函数。 In a typical setup the widget creates an isolate scope, but the transclusion is not a child, but a sibling of the isolate scope. 在一个典型的初始化过程中，widget会创建一个孤立的作用域，但是嵌入并不是其中一个子成员，而是这孤立作用域的兄弟成员。This makes it possible for the widget to have private state, and the transclusion to be bound to the parent (pre-isolate) scope.这使得widget可以有一个私有的状态，并且嵌入被绑定在父作用于上。

	*  true - transclude the content of the directive.嵌入指令的内容。
	*  'element' - transclude the whole element including any directives defined at lower priority.嵌入整个元素，包括优先级较低的指令。

*  compile: This is the compile function described in the section below.这就是后面将要讲到的编译函数。
*  link: 这就是后面将要讲到的链接函数。只有没有提供编译函数时才会用到这个值。

## Compile function编译函数
	
	function compile(tElement, tAttrs, transclude) { ... }

The compile function deals with transforming the template DOM.编译函数是用来处理需要修改模板DOM的情况的。 Since most directives do not do template transformation, it is not used often.因为大部分指令都不需要修改模板，所以这个函数也不常用。 Examples that require compile functions are directives that transform template DOM, such as ngRepeat, or load the contents asynchronously, such as ngView.需要用到的例子有`ngTrepeat`，这个是需要修改模板的，还有`ngView`这个是需要异步载入内容的。 The compile function takes the following arguments.编译函数接受一下参数。
	*  tElement - template element - The element where the directive has been declared.指令所在的元素。 It is safe to do template transformation on the element and child elements only.对这个元素及其子元素进行变形之类的操作是安全的。

	*  tAttrs - template attributes - Normalized list of attributes declared on this element shared between all directive compile functions.这个元素上所有指令声明的属性，这些属性都是在编译函数里共享的， See Attributes.参考章节“属性”。

	*  transclude - A transclude linking function: function(scope, cloneLinkingFn).一个嵌入的链接函数function(scope, cloneLinkingFn)。


NOTE: The template instance and the link instance may not be the same objects if the template has been cloned.注意：如果模板被克隆了，那么模版实例和链接实例可能不是同一个对象。 For this reason it is not safe in the compile function to do anything other than DOM transformation that applies to all DOM clones.所以在编译函数不要进行任何DOM变形之外的操作。 Specifically, DOM listener registration should be done in a linking function rather than in a compile function.更重要的，DOM监听事件的注册应该在链接函数中做，而不是编译函数中。

A compile function can have a return value which can be either a function or an object.编译函数可以返回一个对象或者函数。

*  returning a function返回函数 - is equivalent to registering the linking function via the link property of the config object when the compile function is empty.等效于在编译函数不存在时，使用配置对象的`link`属性注册的链接函数。

*  returning an object with function(s) registered via pre and post properties 返回一个通过`pre`或`post`属性注册了函数的对象- allows you to control when a linking function should be called during the linking phase.使你能更具体的链接函数的执行点。 See info about pre-linking and post-linking functions below。参考下面`pre-linking`和`post-liking`函数的解释。


## Linking function链接函数

	function link(scope, iElement, iAttrs, controller) { ... }

The link function is responsible for registering DOM listeners as well as updating the DOM. 链接函数负责注册DOM事件和更新DOM。It is executed after the template has been cloned.它是在模板被克隆之后执行的。 This is where most of the directive logic will be put.它也是大部分指令逻辑代码编写的地方。

scope - Scope - The scope to be used by the directive for registering watches.指令需要监听的作用域。

iElement - instance element - The element where the directive is to be used.指令所在的元素 It is safe to manipulate the children of the element only in postLink function since the children have already been linked.只有在`postLink`函数中对元素的子元素进行操作才是安全的，因为那时它们才已经全部连接好。

iAttrs - instance attributes实力属性 - Normalized list of attributes declared on this element shared between all directive linking functions. See Attributes.一个标准化的、所有声明在当前元素上的属性列表，这些属性在所有链接函数间是共享的。参考“属性”。

controller - a controller instance控制器实例 - A controller instance if at least one directive on the element defines a controller如果至少有一个指令定义了控制器，那么这个控制器就会被传递. The controller is shared among all the directives, which allows the directives to use the controllers as a communication channel.控制器也是指令间共享的，指令可以用它来相互通信。

Pre-linking function
Executed before the child elements are linked.在子元素被链接前执行。 Not safe to do DOM transformation since the compiler linking function will fail to locate the correct elements for linking.不能用来进行DOM的变形，以为可能导致链接函数找不到正确的元素来链接。

Post-linking function
Executed after the child elements are linked. Safe to do DOM transformation in here.所有元素都被链接后执行。可以操作DOM的变形。

## Attributes 属性
The Attributes object属性对象 - passed as a parameter in the link() or compile() functions - is a way of accessing:作为参数传递给链接函数和编译函数。这使得下列资源可以被使用。

*  normalized attribute names标准化的属性名: Since a directive such as 'ngBind' can be expressed in many ways such as 'ng:bind', or 'x-ng-bind', 因为指令的名称，如`ngBind`可以有很多种变形表示，如`ng:bind`，或者`x-ng-bind`，the attributes object allows for normalized accessed to the attributes.这个对象使得可以用标准的名称获取到相应的属性。

*  directive inter-communication指令间通信: All directives share the same instance of the attributes object which allows the directives to use the attributes object as inter directive communication.所有指令间共享同一个属性对象的实例，这使得指令可以通过这个属性对象通信。

*  supports interpolation支持替换式: Interpolation attributes are assigned to the attribute object allowing other directives to read the interpolated value.属性中若包含替换式，那么其他指令能够独到替换式的值。

*  observing interpolated attributes监视替换式属性: Use $observe  to observe the value changes of attributes that contain interpolation (e.g. src="{{bar}}").使用$observe 能监视使用了替换式的属性(比如 src="{{bar}}")。 Not only is this very efficient but it's also the only way to easily get the actual value because during the linking phase the interpolation hasn't been evaluated yet and so the value is at this time set to undefined.这是一种高效的，也是为唯一的方法来获取变量的值。因为在链接阶段替换式还没有被替换成值，所有变量此时是undefined。

	function linkingFn(scope, elm, attrs, ctrl) {
	  // get the attribute value
	  console.log(attrs.ngModel);
	 
	  // change the attribute
	  attrs.$set('ngModel', 'new value');
	 
	  // observe changes to interpolated attribute
	  attrs.$observe('ngModel', function(value) {
	    console.log('ngModel has changed value to ' + value);
	  });
	}
	
##Understanding Transclusion and Scopes 理解嵌入和作用域
It is often desirable to have reusable components. 开发者总是希望组件能尽量重用。Below is a pseudo code showing how a simplified dialog component may work 。下面是一个简单地对话框组件的伪代码。


	<button ng-click="show=true">show</button>
	<dialog title="Hello {{username}}."
		visible="show"
		on-cancel="show = false"
		on-ok="show = false; doSomething()">
		Body goes here: {{username}} is {{title}}.
	</dialog>

Clicking on the "show" button will open the dialog. 点击`show`按钮会打开对话框。The dialog will have a title,对话框有一个标题， which is data bound to username,标题绑定了一个`username`。 and it will also have a body which we would like to transclude into the dialog.还包含了一个我们希望嵌入对话框的`body`。

Here is an example of what the template definition for the dialog widget may look like.下面是一个模板对话框组件模板的例子。

	<div ng-show="show">
	  <h3>{{title}}</h3>
	  <div class="body" ng-transclude></div>
	  <div class="footer">
	    <button ng-click="onOk()">Save changes</button>
	    <button ng-click="onCancel()">Close</button>
	  </div>
	</div>

在我们正确设置好作用域前，它不会被正确渲染。
The first issue we have to solve is that the dialog box template expect title to be defined, but the place of instantiation would like to bind to username. 第一个要处理的问题就是，对话框模板希望标题是被定义好的，但是初始化的时候需要绑定成`username`。Furthermore the buttons expect onOk as well as onCancel functions to be present in the scope.然后，onOK和onCancel函数要在作用域里被定义好，才能让button按钮起作用。 This limits the usefulness of the widget.这限制了组件的重用。 To solve the mapping issue we use the locals to create local variables which the template expects as follows:要解决这个映射的问题，我们需要创建一个本地的作用域：

	scope: {
	  title: '=',             // set up title to accept data-binding
	  onOk: '&',              // create a delegate onOk function
	  onCancel: '&',          // create a delegate onCancel function
	  show: '='
	}
	
Creating local properties on widget scope creates two problems:在组件上创建本地作用域会造成两个问题：

*  isolation孤立 - if the user forgets to set title attribute of the dialog widget the dialog template will bind to parent scope property. 如果用户忘了给`title`属性赋值，那么模板就会绑定到父作用域的值上。This is unpredictable and undesirable.这是一个比较讨厌的问题。

*  嵌入transclusion - the transcluded DOM can see the widget locals,嵌入的DOM可以获取到组件的本地变量， which may overwrite the properties which the transclusion needs for data-binding.这可能是的嵌入需要的用来做数据绑定的属性被覆写。 In our example the title property of the widget clobbers the title property of the transclusion.在我们的例子里，组件的`title`属性覆写了嵌入的`title`属性。

To solve the issue of lack of isolation, 要解决作用域不独立的问题，the directive declares a new isolated scope. 指令需要声明一个新的孤立作用域。An isolated scope does not prototypically inherit from the child scope,它不从父作用域作原型继承， and therefore we don't have to worry about accidentally clobbering any properties.这样我们也不用担心会发生属性的覆盖。

However isolated scope creates a new problem:但是孤立作用域也会导致一个新问题： if a transcluded DOM is a child of the widget isolated scope then it will not be able to bind to anything. 如果嵌入的DOM是孤立作用域的一个子元素，那它就不会被绑定任何东西。For this reason the transcluded scope is a child of the original scope,为了解决这个问题，需要在组件创建本地作用域前，  让嵌入的作用域成为子作用域的一个属性， before the widget created an isolated scope for its local variables. This makes the transcluded and widget isolated scope siblings.这让组件的作用域和嵌入内容成为兄弟节点。


This may seem as unexpected complexity, but it gives the widget user and developer the least surprise.这可能感觉有点复杂，但是却能给widget的使用者和开发者惊喜。

Therefore the final directive definition looks something like this
所以，指令最后会定义成这个样子：

	transclude: true,
	scope: {
	  title: 'bind',          // set up title to accept data-binding
	  onOk: 'expression',     // create a delegate onOk function
	  onCancel: 'expression', // create a delegate onCancel function
	  show: 'accessor'        // create a getter/setter function for visibility.
	}


## Creating Components创建组件
It is often desirable to replace a single directive with a more complex DOM structure. This allows the directives to become a short hand for reusable components from which applications can be built.

Following is an example of building a reusable widget.

### Source












