---
layout: post
date: 2017-08-31 11:00
title: The Relationship of Java Access Identifiers with Spring DI 
description: Java访问修饰符与Spring 依赖注入之间的关系及注意的问题。
categories: [Spring,Java]
tags: [Spring,Java]
---

# Java访问修饰符与Spring 依赖注入之间的关系及注意的问题
------
> 本文的思考来自昨晚公司项目编码测试中的一个Exception，小小低级错误的排查工作却轮番上阵了好几个人，甚至项目组老大，问题实在不足以引起足够的重视，当时一度认为是项目中其他Jar包（Shiro）引起的该Bean的装配失败，使用该Bean中方法调用引起NullPointerException。

## 引起异常的原代码如下：
```Java
@RestController
@RequestMapping(value = "/api/data_operation")
public class DatabaseOperationController {
    @Autowired
	private DatabaseService databaseService;
	
	/**
	 * 查询线上数据库选项
	 * 
	 * @return
	 */
	@GetMapping("/table_list")
	private ResponseModel getTableList() {
	    List<Database> databaseList = databaseService.getAll();
		......
		return new ResponseModel.Builder().msg("查询成功").result(resultArray).build();
    	}
}
```
**调用该方法后抛出NullPointerException异常，找出问题所在了吗？仔细找找！**

## 异常的总结思考：
> * Java修饰符中有4中访问修饰符:private、package(默认)、protected和public，访问权限逐渐递增。
> * Spring DI/IOC 一般采用private修饰。如：private DatabaseService databaseService;
> * 调用依赖注入的Class的方法的java访问修饰符权限应该在private之上，即采用package(默认)、protected和public中的一种。
> * 采用了private修饰的方法，在其中调用private修饰的依赖注入的类，则该类对象必定为null.因此，该方法调用的依赖注入的类的java访问修饰符一定要高于其类的访问修饰符。

## [Java语言中有4种访问修饰符][1]

1.package是默认的保护模式，又加做包访问，没有任何修饰符时就采用这种保护模式。包访问允许域和方法被同一个包内任何类的任何方法访问.（包内访问）。
 
2.private标识得访问模式，表示私有的域和方法只能被同一个类中的其他方法访问，实现了数据隐藏；必要时，可以通过方法访问私有变量.（类内访问）。
 
3.public修饰符用于暴露域和方法，以便在类定义的包外部能访问它们。对包和类中必要的接口元素，也需要使用这个级别；main()方法必须是public的，toString()方法也必须是public的。一般不会用public暴露一个域，除非这个域已经被声明为final。（跨包访问）。
 
4.protected修饰符提供一个从包外部访问包(有限制)的方法。在域和方法前增加protected修饰符不会影响同一个包内其他类的方法对它们的访问。要从包外部访问包(其中含有protected成员的类)，必须保证被访问的类是带有protected成员类的子类。也就是说，希望包中的一个类被包之外的类继承重用时，就可以使用这个级别。一般应该慎用。（包中类被包外类继承慎用）。

[1]: http://wuhaidong.iteye.com/blog/851754





