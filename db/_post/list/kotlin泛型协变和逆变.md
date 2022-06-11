```
Java 提供了 <? extends T> 和 <? super T> 使泛型拥有协变和逆变的特性
<? extends T> 称为上界通配符，<? super T> 称为下界通配符。使用上界通配符可以使泛型协变，而使用下界通配符可以使泛型逆变。
```

```
List<String> aList...
List<? extends Object> covariantList = aList;
List<? super String> contravariantList = aList;
```

kotlin协变和逆变

out 相当于extends

in 相当于super

```
open class Person
class Worker : Person() 
 

fun main() {
    val personArrayList: MutableList<Person> = ArrayList() 
    personArrayList.add(Person())
    personArrayList.add(Worker()) //Worker是Person的子类

    val workerArrayList: MutableList<Worker> = ArrayList()
    workerArrayList.add(Worker())

    setPerson(personArrayList) // ok
    setPerson(workerArrayList) //会报错，明显类型A是类型B的子类型，但是Generic<A>和Generic<B>没有关系
    setPersonOut(workerArrayList) //就是要这么传参，就要加协变关键字
  
    setWork(workerArrayList) // ok
    setWork(personArrayList) //会报错，明显类型A是类型B的子类型，但是Generic<A>和Generic<B>没有关系
    setWorkIn(personArrayList) //就是要这么传参，就要加逆变关键字
}

fun setPerson(pList: MutableList<Person>) {

}

//协变 类型A是类型B的子类型，那么支持Generic<A>赋值给Generic<B> 
//编译器只知道容器中的类型是B或者它的子类，但是具体什么类型却不知道。保证协变下的安全性，不能add添加，get读取获取到B(最顶级类)
fun setPersonOut(pList: MutableList<out Person>) {
//    pList.add(Person()) 会报错，例如在MutableList<Worker>传入一个Person是不行的。
}

fun setWork(wList:MutableList<out Worker>){

}

//逆变  类型A是类型B的子类型，那么支持Generic<B>赋值给Generic<A> 
//编译器只知道容器中的类型是A或者它的父类，但是具体什么类型却不知道。读取get只能拿到any(最顶级父类)，可以add添加A或它的子类(继承关系)
fun setWorkIn(wList:MutableList<in Worker>){
//    wList.get(0)
//    wList.add(Worker())  wList容器中的类型最小是A了，可以add添加A或它的子类
}
```
