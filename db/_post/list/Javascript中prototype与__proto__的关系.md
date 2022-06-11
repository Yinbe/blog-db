```
function Person(){
}
//自定义原型对象的属性和方法
Person.prototype.name="Tom";
Person.prototype.sayName=function(){
 alert(this.name);
};
//原型对象中的所有属性和方法 都会自动被所有实例所共享
var person1=new Person();
var person2=new Person();
person1.sayName();//Tom
person2.sayName();//Tom
alert(person1.__proto__===Person.prototype);//true

```

只要创建了一个新函数，每个函数在创建之后都会获得一个prototype的属性，这个属性指向函数的原型对象（原型对象在定义函数时同时被创建），此原型对象又有一个名为“constructor”的属性，反过来指向函数本身，达到一种循环指向。

Person.prototype

```javascript
{name: "Tom", sayName: ƒ, constructor: ƒ}
name: "Tom"
sayName: ƒ ()
constructor: ƒ Person()
__proto__: Object
```

每个new出来的对象的__proto__属性都指向其构造函数的原型对象

var person=new Person();

```
Person {}
__proto__:
name: "Tom"
sayName: ƒ ()
constructor: ƒ Person()
__proto__: Object
```
