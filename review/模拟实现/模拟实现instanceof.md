思想：构造函数的原型存在于对象的原型链上，则返回true，否则返回false。

```javascript
function instanceof(obj,constructor){
	let proto = obj.__proto;
	let {prototype} = constructor;
	while(proto!==null){
		if(proto===prototype){
			return true;
		}
		proto = prototype.__proto__;
	}
	return false;
}
```

