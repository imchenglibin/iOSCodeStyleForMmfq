# 项目说明文档

##	1 项目整体结构

#### 结构示意图

	src
		commons
			components
			controllers
			utils
			view-models
			djexports.js(可选)
		modules
			module-a(拥有子模块)
				module-a-a
					components
					controllers
					view-models
					utils
				module-a-b
					...
				...
				index.js
				djexports.js(可选)
			module-b(不拥有自模块)
				components
				controllers
				view-models
				utils
				index.js
				djexports.js(可选)
#### 结构说明

	项目根目录下分为两块：commons和modules
	1 commons：
		这个目录主要用于存放公有的模块，该目录下分为四个子目录：components，controllers，utils，
		view-models分别用于存放公有的组件，控制器，工具类和view-model，其中比较常用的应该是components
		和utils目录，components用于存放一些基础的共有组建，utils用于存放公有的工具类，如http等
		
	2 modules：
		这个目录主要是用于存放项目的模块，项目整体采用 [MVVM 模式](http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html) 编码，
		模块的文件结构符合MVVM模式，拿商品流转补货模块来说，补货模块下面有四个子模块：智能补货，补货管理，补货审核，
		补货单管理，那么整体的结构应该为
		
		modules
			split
				split
					components(V)
					controllers(C)
					utils
					view-models(VM)
					index.js
				manager
					...
				approved
					...
				bill
					...
				djexports.js(可选)
		
		但是有些模块是不包含子模块，那么目录就可以直接为：
		
		modules
			module-name
				components
				controllers
				view-models
				utils
				index.js
				djexports.js(可选）
		
	到目前为止项目的目录结构说明基本已经结束了，但是还有两个东西index.js和djexports.js,
	index.js是项目打包的入口，djexports.js主要用于项目内跨模块调用，具体会在后面的ExportsPlugin
	里面讲解到

### 其他事项
    1 项目文件名统一小写，用‘-’作为分隔符（可以避免不同平台文件大小写敏感问题）
    2 index.js必须放在各个模块的根目录，djexports.js文件规定放在大模块的根目录下面，如上面的split
      模块
      
## 2项目编码规范说明

### 代码规范
    1 整体代码规范
      1 尽量使用const、let替代var
      2 字符串使用',编程构建字符串时，使用字符串模板而不是字符串连接
        ```javascript
            // bad
            function sayHi(name) {
              return 'How are you, ' + name + '?';
            }
            
            // bad
            function sayHi(name) {
              return ['How are you, ', name, '?'].join();
            }
            
            // good
            function sayHi(name) {
              return `How are you, ${name}?`;
            }
        ```
      3 使用字面量语法创建数组,如果你不知道数组的长度，使用 push
        ```javascript
            // bad
            const items = new Array();
            
            // good
            const items = [];
            
            const someStack = [];
    
            // bad
            someStack[someStack.length] = 'abracadabra';
            
            // good
            someStack.push('abracadabra');
        ```
      3 使用对象的多个属性时请建议使用对象的解构赋值
        ```javascript
            const props = {
                style: {
                    left: 30
                },
                className: 'class-name',
                children: []
            };
            
            const { children, className } = props;
        ```
      4 定义类时总是使用 class 关键字，避免直接修改 prototype，class 语法更简洁
        ```javascript
            // bad
            function Queue(contents = []) {
              this._queue = [...contents];
            }
            Queue.prototype.pop = function() {
              const value = this._queue[0];
              this._queue.splice(0, 1);
              return value;
            }
            
            // good
            class Queue {
              constructor(contents = []) {
                this._queue = [...contents];
              }
              pop() {
                const value = this._queue[0];
                this._queue.splice(0, 1);
                return value;
              }
            }
        ```
        定义类时，方法的顺序如下
        constructor
        public get/set 公用访问器，set只能传一个参数
        public methods 公用方法，公用相关命名使用小驼峰式写法(lowerCamelCase)
        private get/set 私有访问器，私有相关命名应加上下划线 _ 为前缀
        private methods 私有方法
        ```javascript
            // good
            class SomeClass {
              constructor() {
                // constructor
              }
            
              get aval() {
                // public getter
              }
            
              set aval(val) {
                // public setter
              }
            
              doSth() {
                // 公用方法
              }
            
              get _aval() {
                // private getter
              }
            
              set _aval() {
                // private setter
              }
            
              _doSth() {
                // 私有方法
              }
            }
        ```
        如果不是class类，不使用new
        ```javascript
            // not good
            function Foo() {
            
            }
            const foo = new Foo();
            
            // good
            class Foo {
            
            }
            const foo = new Foo();
        ```
      5 promise是一种异步处理模式, 发promise申明和调用分开，推荐异步方式使用Promise
        ```javascript
            // not good
            (new Promise((resolve, reject) => {}))
                .then((result) => {})
                .catch((error) => {});
                
            // good
            var promise = new Primise((resolve, reject) => {});
            promise.then((result) => {})
                .catch((error) => {})
        ```
      6 表示区块起首的大括号，不要另起一行
        ```javascript
            // bad
            function sayHi()
            {
                  
            }
              
            // good
            function sayHi() {
                  
            }
        ```
      7 不要省略句末的分号
        ```javascript
            x = y
        　　(function (){
        　　　　...
        　　})();
        　　
        　　// 等同于
        　　x = y(function (){...})(); //something wrong
        ```
      8 避免使用"相等"（==）运算符，尽量使用"严格相等"（===）运算符
        "相等"运算符会自动转换变量类型，造成很多意想不到的情况
      9 尽量不使用自增（++）和自减（--）运算符，用+=和-=代替
      10 总是使用大括号表示区块
        ```javascript
            // bad
            if (a) b();
              
            // good 
            if (a) {
                b();
            }
        ```
    
				
