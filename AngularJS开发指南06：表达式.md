**表达式**是类似Javascript的代码片段，通常在绑定中用到，例如写在{{表达式}}。表达式是用 $parse 方法来处理的。

下面是一些合法的AngularJS表达式

1+2
3*10 | currency
user.name

###Angular 表达式 对比 JS 表达式
你可能会认为AngularJS视图中的表达式就是Javascript表达式，这种认识不完全对，因为AngularJS不会用Javascript的eval()去执行表达式。 不过除了一下几个需要区别的地方以外，你倒是可以把AngularJS表达式看成是Javascript表达式：

属性表达式：属性表达式是对应于当前的作用域的，不像Javascript对应的是全局window对象。

容忍未定义: 执行表达式时，Angular能够容忍undefined或者null，不像Javascript会抛出一个异常。

没有控制结构： 你不能在AngularJS表达式中使用“条件判断”，“循环”，“抛出异常”等控制结构。

过滤器(类似unix中的管道操作符)： 你可以通过过滤器链来传递表达式的结果。例如将日期对象转变成指定的阅读友好的格式。

如果你想要在表达式中使用标准的Javascript，那么你应该把它写成一个控制器的方法，然后在表达式中调用这个方法。如果你想在Javascript中执行AngularJS表达式，你可以使用$eval()方法。

###例子

###Source
index.html

	<!doctype html>
	<html ng-app>
	  <head>
	    <script src="http://code.angularjs.org/angular-1.0.2.min.js"></script>
	  </head>
	  <body>
	    1+2={{1+2}}
	  </body>
	</html>

End to end test

	it('should calculate expression in binding', function() {
	  expect(binding('1+2')).toEqual('3');
	});

###Demo



你可以用下面的例子试试别的表达式：
###Source

index.html:

	<!doctype html>
	<html ng-app>
	  <head>
	    <script src="http://code.angularjs.org/angular-1.0.2.min.js"></script>
	    <script src="script.js"></script>
	  </head>
	  <body>
	    <div ng-controller="Cntl2" class="expressions">
	      Expression:
	      <input type='text' ng-model="expr" size="80"/>
	      <button ng-click="addExp(expr)">Evaluate</button>
	      <ul>
	       <li ng-repeat="expr in exprs">
	         [ <a href="" ng-click="removeExp($index)">X</a> ]
	         <tt>{{expr}}</tt> => <span ng-bind="$parent.$eval(expr)"></span>
	        </li>
	      </ul>
	    </div>
	  </body>
	</html>

script.js:

	function Cntl2($scope) {
	  var exprs = $scope.exprs = [];
	  $scope.expr = '3*10|currency';
	  $scope.addExp = function(expr) {
	     exprs.push(expr);
	  };
	 
	  $scope.removeExp = function(index) {
	    exprs.splice(index, 1);
	  };
	}

End to end test :

	it('should allow user expression testing', function() {
	   element('.expressions :button').click();
	   var li = using('.expressions ul').repeater('li');
	   expect(li.count()).toBe(1);
	   expect(li.row(0)).toEqual(["3*10|currency", "$30.00"]);
	});

###Demo

###计算属性的值
属性的值计算是发生在作用域中的。 Javascript默认是以window为作用域的。AngularJS要使用window的话得用$window来指向全局window对象。 比如说，你使用window中定义的alert()方法，在angular表达式中必须写成$window.alert()才行。这是为了防止意外进入全局作用域（常见的bug来源）而设计的。

###Source
index.html:

	<!doctype html>
	<html ng-app>
	  <head>
	    <script src="http://code.angularjs.org/angular-1.0.2.min.js"></script>
	    <script src="script.js"></script>
	  </head>
	  <body>
	    <div class="example2" ng-controller="Cntl1">
	      Name: <input ng-model="name" type="text"/>
	      <button ng-click="greet()">Greet</button>
	    </div>
	  </body>
	</html>

script.js:

	function Cntl1($window, $scope){
	  $scope.name = 'World';
	 
	  $scope.greet = function() {
	    ($window.mockWindow || $window).alert('Hello ' + $scope.name);
	  }
	}


End to end test:

	it('should calculate expression in binding', function() {
	  var alertText;
	  this.addFutureAction('set mock', function($window, $document, done) {
	    $window.mockWindow = {
	      alert: function(text){ alertText = text; }
	    };
	    done();
	  });
	  element(':button:contains(Greet)').click();
	  expect(this.addFuture('alert text', function(done) {
	    done(null, alertText);
	  })).toBe('Hello World');
	});

###Demo


###容忍未定义

表达式在执行时是可以容忍`undifined`和`null`的。 在javascript，计算a.b.c会抛出一个异常，如果这不是一个对象的话。这对大部分的语言来说是有意义的，但是如果大多数时候表达式是用来作数绑定，像下面这种形式：

    {{a.b.c}}

那么表达式返回一个空值会比触发异常更有意义（因为通常我们是在等待服务器的响应，并且变量马上就会被定义和赋值）。如果表达式不能容忍未定义的值，那么我们绑定的代码就不得不写成：{{((a||{}).b||{}).c}}

类似的，执行未定义的函数 a.b.c()也会只是返回undefined，不会触发异常。

###没有流程控制结构
你不能在表达式中使用控制结构。这样设计得原因在于AngularJS的设计理念之一就是逻辑代码都应该在控制器里，而不是是同种。如果你需要使用条件、循环、或者处罚异常，你就应该写在控制器的方法里。

###过滤器
当将数据呈献给用户时，你很可能需要将数据转换为阅读友好的格式。比方说，你可能需要在显示之前将一个日期对象转换为用户本地的时间格式。你可以用链式的过滤器来传递表达式，像下面这样：

	name | uppercase

这个表达式会将`name`的值传递给uppercase这个过滤器。

链式过滤器使用的是下面这样的语法：

	value | filter1 | filter2

你也可以通过冒号来给过滤器传递参数，比如，将123显示成带有两位小数的形式：

	123 | number:2

###$符号

你可能会好奇，这个$的前缀有什么用?其实这只是一个angular专属，用来表示区别于其他函数的符号。如果angular不用$，那么执行a.length()会返回undeifined，因为a和angular都没有这样一个属性。

但是考虑到我们以后可能会增加一个length的方法（这会改变表达式的行为），同时你也想要自己增加一个length方法的话，那就会产生冲突。这种冲突可能出现都是因为，Angular的设计是在已有的对象上添加行为。使用$做前缀的话，就能使得开发者的代码和AngularJS和谐共处了。
