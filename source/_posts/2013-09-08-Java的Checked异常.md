---
title: Java的Checked异常
date: 2013-09-08
category: others
toc: true
tags:
- java
---
Java里的异常分为Checked和Unchecked两类，其中RuntimeException及其派生类被称为Unchecked异常，其余则是Checked异常。

> +- Throwable   // 可以被throw/catch
> |+- Error      // 通常表示错误，跟Unchecked异常差不多
> |+- Exception  // Checked异常
> ||+- RuntimeException // Unchecked异常

编译器会检查所有抛出的的Checked异常都有被显式处理（throws也算显式处理）。

示例
-----

对于Unchecked异常，使用上跟C#的异常没什么两样：

``` java
class UncheckedException extends RuntimeException{}

public void Run(){
	if( x <= 0){
		throw new UncheckedException();
	}
}
```

而对于Checked异常，如果像上面那样写的话，编译会失败，提示"Unhandled Exception: CheckedException"：

``` java
class CheckedException extends Exception{}

public void Run(){
	if( x <= 0){
		throw new CheckedException(); // Error
	}
}
```

要通过编译，可以加上try/catch处理掉这个异常：

``` java
public void Run(){
	try{
		if( x <= 0){
			throw new CheckedException();
		}
	catch(CheckedException e){
	}
}
```

也可以在方法后声明throws：

``` java
public void Run() throws CheckedException{
	if( x <= 0){
		throw new CheckedException();
	}
}
```

不过这样的话，调用Run()的方法也必须处理这个异常了：

``` java
public void Run2(){
	try{
		Run();
	catch(CheckedException e){
	}
}

public void Run3() throws CheckedException{
	Run();
}
```

throws是方法签名一部分吗?
-----

throws和返回类型都是方法的元数据，自然也是方法签名的一部分：

``` java
interface I {
	void f() throws Exception;
}
```
因为方法可以重载，所以我们可以写出同样的名字但参数不同的方法：
``` java
class A{
	void f(int x){}
	void f(String x){} // OK
}
```
返回类型不同则会编译错误，因为编译器将不知道要调用哪个:
``` java
class A{
	int f(){return 0;}
	String f(){return "";} // Error: f() is already defined in A
}
class B{
	int f(int x){return 0;}
	String f(String x){return "";} // OK
}
```
throws与返回类型类似：
``` java
class A{
	void f() throws Exception{}
	void f() throws RuntimeException{} // Error: f() is already defined in A
}

class C{
	void f(int x){}
	void f(String x) throws Exception{} // OK
}
```
可以这么认为：** 方法名和参数类型确定要调用哪个方法，返回类型跟throws对调用者产生约束。 **

Checked异常带来的问题
-----

假如原先有这么段代码：
``` java
public class NoPowerException extends Exception {}
public class Car{
	private int power = 0;

	public void Run() throws NoPowerException{
		if(power <= 0){
			throw new NoPowerException();
		}
	}
}
public class Main {
    public static void main(String[] args) {
	    Car car = new Car();
	    try {
		    car.Run();
	    } catch (NoPowerException e) {
	    }
    }
}
```
后来出现了一种能飞的车，但有重量限制，这时候纠结了……
``` java
public class TooHeavyException extends Exception{}
public class AirCar extends Car {
	private int weight = 0;
	@Override
	public void Run() throws NoPowerException {
		if(weight > 10){
			throw new TooHeavyException(); // Unhandled Exception：TooHeavyException 
		}
		super.Run();
	}
}
public class Main {
    public static void main(String[] args) {
	    Car car = new AirCar();
	    try {
		    car.Run();
	    } catch (NoPowerException e) {
	    }
    }
}
```
- 方法1: 将TooHeavyException改继承自RuntimeException
> 某一天调用者大呼坑爹：IDE只提示我要处理NoPowerException，这个TooHeavyException哪来的，是愚人节彩蛋吗？

- 方法2: Car.Run和AirCar.Run的方法后面都加上TooHeavyException的声明
> 因为子类而修改父类是大忌，Car本来对TooHeavyException一无所知，加这个声明很难说得通，如果Car这个类是第三方或者JDK里的，这方法就走不通了。

- 方法3: 能飞的还叫Car做啥？我们从零开始造AirCar吧~
> 代价好高……

- 方法4: 不叫Run，叫DoRun()吧~
> 假如Car里面有其他代码调用过Run()方法，那就哭了。这么改下去，代码更难读了……

- 方法n: 期待各位补充

小结
-----

方法的返回类型和throws都是对调用方的约束，并且这种约束还不能通过继承实现多态的，相比之下，Erlang/Javascript在这方面都没有限制，扩展起来倒是舒服很多。