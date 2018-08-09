# Promise

promise 是异步编程的一种解决方案，比传统的解决方案---回调函数和事件---更合理且更强大。

从语法上讲，Promise是一个对象，从它可以获取异步操作的消息。Promise提供统一的API，各种异步操作都可以用同样的方法进行处理。

**promise对象有以下两个特点**。

1. 对象的状态不受外界影响。 Promise对象代表一个异步操作，有 3 种状态: Pending (进 行中)、 Fulfilled (己成功)和 Rejected (己失败)。只有异步操作的结果可以决定当前是哪一种 状态，任何其他操作都无法改变这个状态。这也是“ Promise”这个名字的由来，它在英语中意 思就是“承诺”，表示其他手段无法改变。
2. -旦状态改变就不会再变，任何时候都可以得到这个结果。 Promise 对象的状态改变只 有两种可能:从 Pending 变为 Fulfilled 和从 Pending 变为 Rejected。只要这两种情况发生，状态 就凝固了，不会再变，而是一直保持这个结果，这时就称为 Resolved (己定型)。 就算改变己经 发生，再对 Promise 对象添加回调函数，也会立即得到这个结果。这与事件 CEvent)完全不同。 事件 的特点 是，如果错过了它，再去监 昕是得不到结果的 

有了 Promise 对象，就可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回
调函数 。 此外， Promise 对象提供统 一 的接口，使得控制异步操作更加容易 。

**Promise也有一些缺点**

首先，无法取消promise，一旦新建它就会立即执行。无法中途取消。其次，如果不设置回调函数，promise内部抛出的错误不会反映到外部，再者，当处于Pending状态时，无法得知目前进展到哪一个阶段。

## 基本用法

ES6规定，promise对象是一个构造函数，用来生成promise实例。

下面代码创造了一个promise实例

```js
var promise = new Promise(function(reject,resolve){
	//... some code
	if(/*异步操作成功*/){
		resolve(value)
	}else{
		reject(error)
	}
	
})
```

resolve 函数的作用是，将 Promise 对象的状态从“未完成”变为“成功”(即从 Pending 变为 Resolved)，在异步操作成功时调用，并将异步操作的结果作为参数传递出去: reject 函 数的作用是，将 Promise 对象的状态从“未完成”变为“失败”(即从 Pending 变为 Rejected), 在异步操作失败时调用，并将异步操作报出的错误作为参数传递出去 。

promise实例生成以后，可以用then方法分别指定resolved状态和rejected状态的回调函数。

```js
promise.then(function(value){
	//success
},function(){
	//failure
})
```

下面是一个promise对象的简单举例子

```js
funciton timeout(ms){
	return new Promise((resolve,reject) => {
		setTimeput(resolve,ms,'done')
	})
}

timeout(100).then((value) => {
	console.log(value)
})
```

上面的代码中， timeout方法返回一个Promise实例，表示一段时间以后才会发生的结果。 过了指定的时间 Cms 参数)以后， Promise 实例的状态变为 Resolved， 就会触发 then 方法绑 定的回调函数 。

用promise实现一个异步加载图片

```js
function loadImage(imgUrl){
	return new Promise((resolve,reject) => {
		var image = new Image();
		
		image.onload = function(){
			resolve(image)
		}
		
		image.onerror = function(){
			reject(new Error('Could not load image at' + imgUrl))
		}
		
		image.src = imgUrl;
	})
}
```

下面是一个用promise对象实现的Ajax操作的例子

```js
var getJson = function(url){
	var promise = new Promise(function(resolve,reject){
		var client = new XMLHttpRequest();
		client.open("GET",url);
		client.onreadystatechange = handler;
		client.responseType = "json";
		client.setRequestHeader("Accept","application/json");
		client.send();
		
		funciton handler(){
			if(this.readyState !== 4){
				return;
			}
			
			if(this.status === 200){
				resolve(this.response);
			}else{
				reject(new Error(this.statusText));
			}
		}
	})
	return promise;
}

getJson("/posts.json").then(function(json){
	console.log('Contents: ' + json)
},function(error){
	console.error("出错了",error)
})
```

上面的代码中， getJSON 是对 XMLHttpRequest 对象的封装，用于发出一个针对 JSON 数据的 HTTP 请求，并返回 一个 Promise 对象。需要注意的是，在 getJSON 内部， resolve 函数和 reject 函数调用时都带有参数。

一般来说，调用 resolve 或 reject 以后， Promise 的使命就完成了，后继操作应该放到 then 方法里面，而不应该直接写在 resolve 或 reject 的后面。所以，最好在它 们前面加上 return i吾句，这样就不会产生意外 。

```js
new Promise (function(resolve,reject){
	return resolve(1)
	
	// 后面的语句不会执行
	console.log(2);
})

```

## Promise.prototype.then()

Promise 实例具有 then 方法，即 then 方法是定义在原型对象 Promise .prototype 上 的。它的作用是为 Promise实例添加状态改变时的回调函数。 前面说过， then方法的第一个参数是 Resolved 状态的回调函数，第二个参数(可选)是 Rejected 状态的回调函数。

then 方法返回的是一个新的 Promise 实例(注意，不是原来那个 Promise 实例)。因此可 以采用链式写法，即 then 方法后面再调用另一个 then 方法。

采用链式的then可以指定一组按照次序调用的回调函数。这时，前一个回调函数有可能返回的还是一个promise对象，而后一个回调函数就会等待改promise对象的状态发生变化，再被调用。

```js
getJson("/post/1.json").then(function(post){
	return getJSon(post.commentURL);
}).then(function funcA(comments){
	console.log("Resolve: ", comments);
},function funcB(err){
	console.log("Rejected: ", err);
})

```
上面的代码中，第一个 then 方法指定的回调函数返回的是另一个 Promise 对象。这时， 第二个 then 方法指定的回调函数就会等待这个新的 Promise 对象状态发生变化 。 如果变为 Resolved， 就调用 funcA:如果状态变为 Rejected，就调用 funcB.

```js
getJSON(”/post/l.json”) .then ( post=> getJSON(post . comrnentURL)
) .then (
comments => console .log (”Resolved:”, comments) , err => console.log (”Rejected: ”, err)
)
```

## Promise.prototype.catch()

promise.protoytype.catch方法是.then(null,rejectionn)的别名，用于指定发错误时的回调函数。

```js
getJson('/post.json').then(funciton(post){
//...
}).catch(function(error){
	console.log("发生错误！",error);
})
```

上面的代码中， getJSON 方法返回一个 Promise对象，如果该对象状态变为 Resolved，则 会调用 then 方法指定的回调函数 : 如果异步操作抛出错误，状态就会变为 R句巳cted，然后调 用 catch 方法指定的回调函数处理这个错误。另外， then 方法指定的回调函数如果在运行中 抛出错误，也会被 catch 方法捕获。

如果 Promise状态己经变成 Resolved，再抛出错误是无效的。因为promise的状态一旦改变，就会永久保持该状态，不会再改变了。

promise对象的错误具有“冒泡”性质，会一直向后传递，直到被捕获为止。也就是说，错误总会被下一个catch捕获。

```js
getJSON (’ lpostll . json ’) .then(function(post) {
	return getJSON(post.commentURL) ;
}) . then (function(comments) { 
	// some code
}) .catch (function(error) { 
	//处理前面 3 个 Promise 产生 的错误
});
```

因此，**建议使用 catch 方法，而不使 用 then 方法的第二个参数**。
跟传统的 try/catch 代码块不同的是，如果没有使用 catch 方法指定错误处理的回调函
数， Promise 对象抛出的错误不会传递到外层代码 ，即不会有任何反应 。

需要注意的是，catch返回的是一个promise对象，因此后面还可以接着调用then方法。

## Promist.all()

promise.all方法用于将多个promise实例包装成一个新的promise实例

```js
var p = Promist.all([p1,p2,p3]);
```

上面的代码中 ， Promise . all 方法接受 一个数组作为参数 ， pL p2 , p3 都是 promise对象的实例:如果不是 ， 就会先调用下面讲到的 Promise . resolve 方法 ， 将参数转为 Promise 实例 ，再进一步处理( Promise . all 方法的参数不一定是数组 ， 但是必须具有 Iterator接口 ， 且返回的每个成员都是 Promise 实例)。

p的状态由pLp2、p3决定， 分成两种情况。
1. 只有 pl、 p2、 p3抖的状态都变成 Fulfilled, p 的状态才会变成 Fulfilled，此时 pl、 p2、
p3 的返回值组成一个数组，传递给 p 的回调函数。
2. 只要 pl、p2、p3 中有一个被 Rejected, p 的状态就变成 Rejected， 此时第一个被 R句ected
的实例的返回值会传递给 p 的回调函数。

```js
var promist = [1,2,3,5,7].map(function(id){
	return getJson('/post/' + id + ".json");
})

promise.all(promise).then(function(posts){
	//...
}).catch(function(reson){
	//...
})
```

如果**作为参数的 Promise 实例自身定义了 catch 方法，那么它被 rejected 时并不会触
发 Promise . all ( )的 catch 方法**。

## Promise.race()

promise.race()方法同样是将多个Promise实例包装成一个新的Promise实例

```js
var p = Promise.race([p1,p2,p3]);
```

Promise.race 方法的参数与 Promise.all 方法一样，如果不是 Promise 实例，就会先 调用下面讲到的 Promise.resolve 方法，将参数转为 Promise 实例 ，再进一步处理。

```js
const p = Promise.race([
	fetch('/resource-that-may-take-a-while'),
	new Promise(function(resolve,reject){
		setTimeout(() => reject(new Error('request timeout')),5000)
	})
]);

p.then(response => console.log(resoponse));
p.catch(error => console.log(error));
```

上面的代码中，如果 5 秒之内 fetch 方法无法返回结果，变量 p 的状态就会变为 Rejected, 从而触发 catch 方法指定的回调函数。

## Promise.resolve

有时需要将现有对象转为Promise对象，Promise方法就起到这个作用。

* 参数是一个 Promise 实例
如果参数是 Promise 实例 ，那么 Promise .resolve 将不做任何修改，原封不动地返回这 个实例 。

* 参数是一个 thenable对象
thenable 对象指的是具有 then 方法的对象，比如下面这个对象。

```js
let thenable = {
then : function(resolve , reject) {
resolve (42);
```
Promise.resolve 方法会将这个对象转为 Promise 对象，然后立即执行 thenable 对象 的 then 方法。

* 参数不是具有 then 方法的对象或根本不是对象 

如果参数是一个原始值，或者是一个不具有 then 方法的对象，那么 Promise . resolve
方法返回一个新的 Promise对象，状态为 Resolved。

* 不带有任何参数

Promise.resolve 方法允许在调用时不带有参数 ， 而直接返回 一个 Resolved 状态的
Promise 对象 。

## Promise.reject

Promise.reject(reason )方法也会返回 一 个新的 Promise 实例，状态为 Rejected。

```js
var p = Promise.reject ( ’ 出错了 ’ );
//等 同于
var p =new Promise((resolve, reject) => reject( ’出错了’))

p.then(null, function (s) { console .log(s)
});
// 出错了
```

上面的代码中， Promise.reject 方法的参数是一个 thenable 对象，执行以后，后面
catch 方法的参数不是 reject 抛出的“出错了”这个字符串，而是 thenable 对象。

## 两个有用的附加方法

ES6 的 Promise API 提供的方法不是很 多 ，可以自己部署 一些有用 的方法 。下面部署两个不 在 ES6 中但很有用的方法。

### done()

无论 Promise 对象的回调链以 then 方法还是 catch 方法结尾，只要最后一个方法抛出错
误， 都有可能无法捕捉到(因为 Promise 内部的错误不会冒泡到全局) 。为此，我们可以提供一 个 done 方法，它总是处于回调链的尾端，保证抛出任何可能出现的错误。

```js
asyncFunc()
.then(f1)
.catch(r1)
.then(f2)
.done();
```

它的实现代码相当简单。

```js
promise.prototype.done = function(onFulfilled,onRejected){
	this.then(onFulfilled,onRejected){
		.catch(function(reason){
			setTimeout(() => {throw reason},0)
		})
	}
}

```

由上可见， done方法可以像then方法那样使用，提供Fulfilled和R句ected状态的回调函数，也可以不提供任何参数。但不管怎样， done 方法都会捕捉到任何可能出现的错误，并向全局抛出。


### finally()

finally()方法用于指定不管promise对象最后状态如何都会执行的操作，它与done方法的最大区别在于，他接受一个普通的回调函数作为参数，该函数不管怎么样都必须执行。

下面是一个例子，服务器使用Promise处理请求，然后使用Promise关掉服务器。

```js
server.listen(0).then(function(){
	// run test
})
.finally(server.stop);
```

它的实现也很简单。

```js
Promise.prototype.finally = function(callback) {
	let p = this.constructor;
	return this.then(
		value => P.resolve(callback().then(() => value));
		reson => P.resolve(callback().then(() => { throw reason}));
	)
}
```

上面的代码中 ， 不管前面的 Promise 是 fulfilled 还是 rejected，都会执行回调函数 callback 。


## 应用

### 加载图片

我们可以将图片的加载写成一个Promise，一旦加载完成，Promise的状态发生变化。

```js
const preloadImage = function(path){
	return new Promise(function(resolve,reject){
		var img = new Image();
		img.onload = resolve;
		img.onerror = reject;
		img.src = path;
	});
};
```

### Generator函数与Promise的结合

使用Generator函数管理流程，遇到异步操作通常返回一个Promise对象。

```js
function getFoo(){
	return new Promise((resolve,reject) => {
		resolve("foo");
	})
}

var g = function * (){
	try{
		var foo = yield getFoo();
		console.log(foo);
	}catch (e){
		console.log(e)
	}
}

function run(generator){
	var it = generator();
	function go(result){
		if(result.done) return result.value;
		return result.value.then(function(value){
			return go(it.next(value))
		},function(error){
			return go(it.throw(error))
 	}); 
 			
	}
	
   go(if.next());
}
run(g);
```


上面的Generator函数g中有一个异步操作getFoo,他返回的就是一个Promise对象，函数run用来处理这个Promise对象，并调用下一个next方法。

## Promise.try()

实际开发中经常遇到一种情况 : 不知道或者不想区分函数 f 是同步函数还是异步操作，但 是想用 Promise 来处理它 。 因为这样就可以不管 f 是否包含异步操作，都用 then 方法指定下 一步流程 ，用 catch 方法处理 f 抛出的错误 。一般的写法如下。

`Promise. resolve() . then (f)`
上面的写法有一个缺点 : 如果 f 是同步函数，那么它会在本轮事件循环的末尾执行。

```js
const f = () => console.log(’now’); Promise . resolve() .then(f);
console . log ( ’ next ’ );
//next
// now
```

上面的代码中，第二行是一个立即执行的匿名函数，会立即执行里面的async函数，因此如果f是同步的，就会得到同步的结果，如果f是异步的，就可以用then指定下一步，写法如下。

(async () => ff())().then(...)

需要注意的是，async() => f() 会吃掉 f()抛出的错误，所以，如果想捕获错误，需要使用Promise.catch方法

(async () => f())().then().catch()

第二种写法是使用 new Promise()

```js
const f = () => console.log('now');

(
	() => new Promise(
		resolve => resolve(f())
	)();
)

console.log('next');

// now
// next

```

鉴于这是 一个很常见的需求，所以目前有 一个提案组ithub.com/ljharb/proposal- promise-町〉 提供了 Promise.try方法替代上面的写法。

```js
const f = () => console.log('now');

Promise.try(f);
console.log('next');

// now
// next
```

由于 `Promise.try` 为所有操 作提供 了统一 的处理机制，所以如果想用 then 方法管理流 程， 最好都用 Promise.try包装一下。这样有许多好处，其中一点就是可以更好地管理异常。

上面的写法很笨拙，这时可以统 一 用 promise . catch ()捕获所有同步和异步的错误 。

```js
Promise.try(database.users . get({id : userid}))
.then (...)
.catch (...)
```
事实上， Promise.try 是模拟了 try 代码块，就像 promise.catch 模拟 catch 代码 块一样 。

## 模拟一个Promise


```js
/*
实现一个PromiseA+
*/

funciton Promise(executor){
	let self = this;
	self.status = "pending";// 存储promise状态 pending fulfilled rejected.
	self.value = undefined;// 存储成功后的值
	self.reason = undefined; // 记录失败的原因
	self.onFulfilledCallbacks = [];//  异步时候收集成功回调
	self.onRejectedCallbakcs = [];//  异步时候收集失败回调
	
	function resolve(value){
		if(self.status === "pending"){
			self.status === "fulfilled";;// resolve的时候改变promise的状态
			self.value = value;//修改成功的值
			 // 异步执行后 调用resolve 再把存储的then中的成功回调函数执行一遍
			self.onFulfilledCallback.forEach(element => { 
				element();
			})
		}
	}
	function reject(reason){
		if(self.status === "pending"){
			self.stutus = "rejected"; // reject的时候改变promise的状态
			self.reason = reason;//修改失败的原因
			// 异步执行后 调用reject 再把存储的then中的失败回调函数执行一遍
			self.onRejectedCallbacks.forEach(element => {
				element();
			})
		}
	}
	
	// 如果执行器中抛出异常 那么就把promise的状态用这个异常reject掉
	try{
		executor(resolve,reject);
	}catch(error){
		reject(error);
	}
}

Promise.prototype.then = function(onFulfilled,onRejected){
	let self = this;
	// 如果onFulfilled不是函数 那么就用默认的函数替代 以便达到值穿透
	onFulfilled = typeof onFulfilled === "function" ? onFulfilled: val => val;
	// 如果onrejected不是函数 那么就用默认的函数替代 以便达到值穿透
	onRejected = typeof onRejected === "function" ? onRejected : err => {throw err}
	
	let promise2 = new Promise((resolve,reject) => {
		if(self.status === "fulfilled"){
		// 加入setTimeout 模拟异步
       // 如果调用then的时候promise 的状态已经变成了fulfilled 那么就调用成功回调 并且传递参数为 成功的value
       
			setTimeout(() => {
			// 如果执行回调发生了异常 那么就用这个异常作为promise2的失败原因
				try{
					// x 是执行成功回调的结果
					let x = onFulfilled(self.value);
					// 调用resolvePromise函数 根据x的值 来决定promise2的状态
					resolvePromise(promise2,x,reslove,reject);
				}catch(error){
					console.log(error);
				}
			},0)
		}
		
		if(self.status === "rejected"){
		  // 加入setTimeout 模拟异步
         // 如果调用then的时候promise 的状态已经变成了rejected 那么就调用失败回调 并且传递参数为 失败的reason

			setTimeout(() => {
			// 如果执行回调发生了异常 那么就用这个异常作为promise2的失败原因
				try{
				 	// x 是执行失败回调的结果
					let x = onRejected(self.reason);
					 // 调用resolvePromise函数 根据x的值 来决定promise2的状态
					resolvePromise(promise2,x,reslove,reject);
				}catch(error){
					console.log(error);
				}
			},0)
		}
		
		if(self.status === "pending"){
			self.onFulfilledCallbacks.push(() => {
				setTimeout(() => {
				try{
					let x = onFulfilled(self.value);
					resolvePromise(promise2,x,reslove,reject);
				}catch(error){
						console.log(error);
					}
				},0)
			})
			
			self.onRejectedCallbacks.push(() => {
				setTimeout(() => {
				try{
					let x = onRejected(self.reason);
					resolvePromise(promise2,x,reslove,reject);
				}catch(error){
					console.log(error);
				}
				},0)
			})
		}
	})
	return promise2;
}

```

```js
function resolvePromise(prmise2,x,resolve,reject){
	 // 首先判断x和promise2是否是同一引用 如果是 那么就用一个类型错误作为Promise2的失败原因reject
	 if(promise2 === x) return reject(throw typeError('typeError:大佬，你循环引用了!'))
	 
	 let called;
	 
	 if(x !== null & (typeof x ==="object" || typeof x === "function")){
	 	// 如果x是一个对象或者函数 那么他就有可能是promise 需要注意 null typeof也是 object 所以需要排除掉
        //先获得x中的then 如果这一步发生异常了，那么就直接把异常原因reject掉
	     try{
	     	let then = x.then;
	     	if(typeof then === "function"){
	     	//如果then是个函数 那么就调用then 并且把成功回调和失败回调传进去，如果x是一个promise 并且最终状态时成功，那么就会执行成功的回调，如果失败就会执行失败的回调如果失败了，就把失败的原因reject出去，做为promise2的失败原因，如果成功了那么成功的value时y，这个y有可能仍然是promise，所以需要递归调用resolvePromise这个方法 直达返回值不是一个promise
	     	 
		    	then.call(x,y => {
		    		if (called) return;
		    		called = true;
		    		resolvePromise(promise2,x,reject,resolve);
		    	},error =>{
		    		if(called) return;
		    		called = true;
		    		reject(error);
		    	})

	     	}else{
	     		resolve(x);
	     	}
	     }catch(error){
	     		if(called) return;
	    		called = true;
	    		reject(error);
	     }
	 }else{
	 // 如果是一个普通值 那么就直接把x作为promise2的成功value resolve掉
	 	resolve(x);
	 }
}
```

## 实现一个promise的语法糖

```js
Promise.defer = Promise.deferred = function (){
    let dfd = {};
    dfd.promise = new Promise((resolve,reject)=>{
        dfd.resolve = resolve;
        dfd.reject = reject;
    })
    return dfd
}

// 使用语法糖
let newGetImgWidthHeight = function(imgUrl){
        let dfd = Promise.defer();
        let img = new Image();
        img.onload = function(){
            dfd.resolve(img.width+'-'+img.height)
        }
        img.onerror = function(e){
            dfd.reject(e)
        }
        img.url = imgUrl;
        return dfd.promise
    }
```