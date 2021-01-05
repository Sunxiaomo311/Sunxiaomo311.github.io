---
title: 【Java】JDK动态代理与CGLib动态代理
comments: true
excerpt: JDK动态代理与CGLib动态代理
date: 2021-01-05 17:45:00
  - Java
categories:
  - 编程
  - 服务端
---
## 一、使用JDK动态代理

基于 jdk 的动态代理需要借助于 **jdk 中 reflect 包下(java.lang.reflect.Proxy)的 Proxy** 以及 **(java.lang.reflect.InvocationHandler) InvacationHandler** 去生成代理对象。

局限性是这种动态代理 **只适用于对有接口的类实现动态代理** 

原理其实是，**利用反射机制先生成要被代理对象的同级对象然后对这个同级对象进行增强** 

1、接口和实现类（代理对象）
```
public interface UserDao {
	public void add();
	public void update();
}
```
```
package cn.itcast.demo1;
 
public class UserDaoImp implements UserDao {
 
	@Override
	public void add() {
		System.out.println("this is add....");
 
	}
 
	@Override
	public void update() {
		System.out.println("this is update....");
	}
 
}
```
2、动态代理（重点）
```
package cn.itcast.demo1;
 
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
/**
 * 使用jdk动态代理，增强方法
 * 想使用jdk动态代理，代理对象类必须实现至少一个接口 
 * @author Mr.Tong
 *
 */
//继承InvocationHandler
public class JDKProxy implements InvocationHandler{
	private UserDao userDao;//想使用jdk动态代理，代理对象类必须实现至少一个接口
 
	public JDKProxy(UserDao userDao) {
		super();
		this.userDao = userDao;
	}
	public UserDao createProxy(){
		//利用接口获得一个增强后的代理对象
		UserDao proxy = (UserDao) Proxy.newProxyInstance(userDao.getClass().getClassLoader(), userDao.getClass().getInterfaces(), this);
		return proxy;
	}
	//调用对象的任何一个方法，都会执行该方法
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		 if("add".equals(method.getName())){
			 System.out.println("这里是add方法的增强部分");
			 Object result = method.invoke(userDao, args);
		 }
		 if("update".equals(method.getName())){
			 System.out.println("这里是update方法的增强部分");
			 Object result = method.invoke(userDao, args);
		 }
		return null;
	}
	
}
```
3、测试
```
package cn.itcast.demo1;
 
import org.junit.Test;
 
public class Test1 {
	@Test
	//不使用动态代理
	public void demo1(){
		 UserDao dao = new UserDaoImp();
		 dao.add();
		 dao.update();
	}
	@Test
	public void demo2(){
		UserDao userDao = new UserDaoImp();//实例化一个需要增强的对象
		UserDao daoProxy = new JDKProxy(userDao).createProxy();//jdk动态代理 返回一个增强之后的对象
		daoProxy.add();
	}
}
```

## 二、使用CGLib代理

JDK动态代理是基于接口的方式，换句话来说就是代理类和目标类都实现同一个接口，那么代理类和目标类的方法名就一样了；
CGLib动态代理是 **代理类去继承目标类，然后重写其中目标类的方法** ，这样也可以保证代理类拥有目标类的同名方法；

1、代理对象
```
package cn.itcast.demo2;
 
public class ProductDao {
	public void add(){
		System.out.println("this is add....");
	}
	public void update(){
		System.out.println("this is update....");
	}
}
```
2、生成代理（重点）
```

package cn.itcast.demo2;
 
import java.lang.reflect.Method;
 
import org.springframework.cglib.proxy.Callback;
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodProxy;
/**
 * CGLib生成代理的机制是继承
 * @author Mr.Tong
 *
 */
//继承cglib的MethodInterceptor
public class CGLibProxy implements org.springframework.cglib.proxy.MethodInterceptor{
	private ProductDao productDao;
 
	public CGLibProxy(ProductDao productDao) {
		super();
		this.productDao = productDao;
	}
	public ProductDao creatCGLibProxy(){
		//使用CGLib生成代理
		//1、创建核心类
		Enhancer enhancer = new Enhancer();
		//2、为它设置父类
		enhancer.setSuperclass(productDao.getClass());
		//3、设置回调
		enhancer.setCallback((Callback) this);
		return (ProductDao) enhancer.create();
	}
	@Override
	public Object intercept(Object proxy, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
		if ("add".equals(method.getName())) {
			System.out.println("CGLib代理对象add方法的增强部分");
			Object result = methodProxy.invokeSuper(proxy, args);
			return result;
		}
		return methodProxy.invokeSuper(proxy, args);
	}
}
```
3、测试
```
package cn.itcast.demo2;
 
import org.junit.Test;
 
public class Test2 {
	@Test
	public void demo1(){
		 ProductDao productDao = new ProductDao();
		ProductDao productDao2 = new CGLibProxy(productDao).creatCGLibProxy();
		productDao2.add();
	}
}
```


> 注意： Spring 的 AOP 的底层用到两种代理机制：
 > * JDK 的动态代理 :针对实现了接口的类产生代理.
 > * Cglib 的动态代理 :针对没有实现接口的类产生代理. 应用的是底层的字节码增强的技术 生成当前类的子类对象