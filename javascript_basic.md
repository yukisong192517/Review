# javascript基础知识

## 类型
* 原始类型： `boolean,null,undefined,number,string,boolean`
* 原始类型都是值类型，没有函数可以调用
* 值类型的判断： typeof ,返回的是类型的字符串值。但注意typeof null 的值为 'object'.因此null的判断需要符合类型的判断
`（!a && typeof a === 'object'）`。
* 为什么null返回的是object呢?js最初的版本使用的是32位系统，低位存储的是变量类型信息。000是对象，但是null是全0.因此被判断为object
### 对象类型

原始类型中存储的是值，对象类型存储的是指针。基本类型的变量都存在栈中，这些类型在内存中分别占有固定大小的空间，通过按值来访问。
引用类型在内存空间中保存在堆内存中，因为这种值的大小不固定，因此不能把它们保存到栈内存中，但内存地址大小的固定的，因此保存在堆内存中，在栈内存中存放的只是该对象的访问地址。当查询引用类型的变量时， 先从栈中读取内存地址， 然后再通过地址找到堆中的值。对于这种，我们把它叫做按引用访问。
因此当变量赋值给另一个变量的时候，赋值的是地址，因此当变量修改的时候，修改的是堆中的值，因此两个值都会发生变化。

### typeof 和 instanceof

* 对于原始类型，除去null，都可以正确判断，对于对象，函数的typeof是function。
* 对于对象的判断，instanceof可以正确的进行判断。因为instanceof是基于原型链进行判断的。但是instance是不能判断原始类型的。
```
class primitiveString{
 static[Symbol.hasInstance](x){ // 自定义instance
 return typeof x === 'string'
 }
}
```
### 类型转换

* 显示类型转换
    见下面相互转换表格
* 隐式类型转换
    四则运算符：
    * 如果某个操作数是字符串或者能够转换成为字符串的+进行拼接操作。如果其中一个操作是数组对象，先toPrimitive，再调用defaultValue。
* 字符串，数字，布尔值之间的相互转换

|   | toString  | toNumner  |toBoolean   |
|---|---|---|---|
| String  |  /  | '1' => 1, 'a' => NaN | ''=> false,其余为true  |
| Numner  | 5=>'5','1.07e21'  | /  | 0,-0,NaN 为false,其余为true  |
| Boolean  | 'true','false' | true => 1, false => 0  | /  |
| undefined,null  | 'null','undefined' | 0 | false |
| object  | `[object Object]`，如果自行定义toString则调用 | valueOf,toString若均不返回数字类型，NaN  | true |
| array  | `[1,2,3]`=> '1,2,3'| `Number([1,2]) => NaN`, `Number([1]) => 1`,`Number(['a']) => NaN`,`Number(['']) => 0`| true |

###  `==` 和`===`

`==`如果双方类型不一样就会进行类型转换，

1. 如果双方类型相同，比较大小
2. 不同，先判断是否在比较undefined和null 是的话返回true
3. 如果两者类型是在比较string和number，则会把string转换为number进行数值的比较
4. 如果其中一方为boolean，则会把Boolean转为number再进行判断。
5. 其中一方是否是object，另一方是string，number或者symbol。如果是的话，把object转换为原始类型再进行判断。

## this

* 什么是this

  当一个函数被调用时，会创建一个活动记录(有时候也称为执行上下文)。这个记录会包 含函数在哪里被调用(调用栈)、函数的调用方法、传入的参数等信息。this 就是记录的 其中一个属性，会在函数执行的过程中用到。

* 调用栈和调用位置

```
function baz() {
// 当前调用栈是:baz
// 因此，当前调用位置是全局作用域
console.log( "baz" );
bar(); // <-- bar 的调用位置 
}
```
* 分析出调用栈和调用位置之后，就可以分析this的绑定了
    * 独立函数调用: 函数调用时应用了 this 的默认绑定，因此 this 指向全局对象。严格模式下this的绑定不受调用位置影响，为全局对象
       ```
       function foo() { console.log( this.a );}
       var a = 2; 
       foo(); // 2
       ```
    * 隐式绑定
    调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。对象属性中只有最后一层影响this的绑定。
    
    ```
    function foo() {
        console.log( this.a );
    }
    
    var obj = {
        a: 2,
        foo: foo
    };
    
    obj.foo(); // 2
    ```
    
    * 隐式绑定丢失
      
        ```
        function foo() { console.log( this.a );
        }
        var obj = { a: 2,
        foo: foo };
        var bar = obj.foo; // 函数别名!
        var a = "oops, global"; // a 是全局对象的属性
        bar(); // "oops, global"
        ```
        隐式绑定的丢失常常发生在函数赋值给别名，函数作为参数传入，或者回调函数丢失this。
        ```
        function foo() {
            console.log( this.a );
        }
        
        function doFoo(fn) {
            // fn其实引用的是foo
            
            fn(); // <-- 调用位置！
        }
        
        var obj = {
            a: 2,
            foo: foo
        };
        
        var a = "oops, global"; // a是全局对象的属性
        
        doFoo( obj.foo ); // "oops, global"
        
        // ----------------------------------------
        
        // JS环境中内置的setTimeout()函数实现和下面的伪代码类似：
        function setTimeout(fn, delay) {
            // 等待delay毫秒
            fn(); // <-- 调用位置！
        }
        ```
    * 显示绑定
    
        call & apply，它们的第一个参数是一个对象，它们会把这个对象绑定到 this，接着在调用函数时指定这个 this。因为你可以直接指定 this 的绑定对象，因此我 们称之为显式绑定。
        
        * 显式绑定无法解决绑定丢失的问题。
            * 硬绑定 与 es5的bind函数
            ```
            function foo(something) {
                console.log( this.a, something );
                return this.a + something;
            }
            
            // 简单的辅助绑定函数
            function bind(fn, obj) {
                return function() {
                    return fn.apply( obj, arguments );
                }
            }
            
            var obj = {
                a: 2
            };
            
            var bar = bind( foo, obj );
            var bar = foo.bind(obj); // es5的硬绑定。
            
            var b = bar( 3 ); // 2 3
            console.log( b ); // 5
            ```
            * api调用的上下文
            
            ```
            function foo(el) {
            	console.log( el, this.id );
            }
            
            var obj = {
                id: "awesome"
            }
            
            var myArray = [1, 2, 3]
            // 调用foo(..)时把this绑定到obj
            myArray.forEach( foo, obj );
            // 1 awesome 2 awesome 3 awesome
            ```
    * new 绑定
        * 调用new操作符的时候会执行下面的步骤
            1. 创建(或者说构造)一个全新的对象。
            2. 这个新对象会被执行`[[原型]]`连接。
            3. 这个新对象会绑定到函数调用的this。
            4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。
            
        ```
        function foo(a) {
            this.a = a;
        }
        
        var bar = new foo(2); // bar和foo(..)调用中的this进行绑定
        console.log( bar.a ); // 2
        ```
        * 手写一个new
        ```
        function create(){
           let obj = new Object();
            Con = [].shift.call(arguments);
            obj.__proto__ = Con.prototype;
            let ret = Con.apply(obj,arguments);
            return ret instanceof Object ? ret : obj;
        }
        var person = create(Person, ...)
        ```
    * 优先级
        1. 函数是否在new中调用(new绑定)?如果是的话this绑定的是新创建的对象。
             var bar = new foo()
        2. 函数是否通过call、apply(显式绑定)或者硬绑定调用?如果是的话，this绑定的是 指定的对象。
             var bar = foo.call(obj2)
        3. 函数是否在某个上下文对象中调用(隐式绑定)?如果是的话，this 绑定的是那个上 下文对象。
             var bar = obj1.foo()
        4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到 全局对象。
             var bar = foo()
    * bind 与 函数curry话
    ```
    function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
    }
    // 把数组“展开”成参数
    foo.apply( null, [2, 3] ); // a:2, b:3
    // 使用 bind(..) 进行柯里化
    var bar = foo.bind( null, 2 ); bar( 3 ); // a:2, b:3
    ```
    * 间接引用
      间接引用下，调用这个函数会应用默认绑定规则。间接引用最容易在赋值时发生。
      ```
      // p.foo = o.foo的返回值是目标函数的引用，所以调用位置是foo()而不是p.foo()或者o.foo()
      function foo() {
          console.log( this.a );
      }
      
      var a = 2;
      var o = { a: 3, foo: foo };
      var p = { a: 4};
      
      o.foo(); // 3
      (p.foo = o.foo)(); // 2
      ```
      
     * 软绑定：
     如果给默认绑定指定一个全局对象和undefined以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改this的能力。
    * this词法
    箭头函数无法使用上述四条规则，而是根据外层（函数或者全局）作用域（词法作用域）来决定this。箭头函数的this一旦确定无法更改。
    ```
    function foo() { setTimeout(() => {
    // 这里的 this 在此法上继承自 foo()
                 console.log( this.a );
             },100);
    }
    var obj = { a:2
         };
         foo.call( obj ); // 2
    ```
