---
title: pythonic技巧
tags: [python,language]
categories: language
---
pythonic 技巧
========================

1. 赋值可以采用元组赋值方法: `a,b = 1, 2` , 函数的返回值也可是多值返回, `a, b = test(c)`这一点与golang很像, 支持多个, 对于数组赋值也可以通过序列解包来进行赋值`a,b = [1,2]`

2. 对于方法处理,可以进行多次嵌套, 如 `a.split('\n').strip()`

3. if...else....可以进行三目运算,类似c语言, `a = b if a > 10 else c`
<!--more-->
4. 多值判断用 `if 70 < a < 90 : ` 不用 `if a > 70 and a < 90` 或者使用`if a in (1,2,3):`判断多个条件, 判断空值或None 用`if a`而不是 `if len(a)>0 or a != None`,判断多个and 或 or条件 可以使用 `if all(a>60,b>70):`

5. for 循环 尽量使用推导式 `a= [x for x in b_list]`,同时list的下标和数值用 `for k,v in enumerate(a)`,推导式比较省内存,因为其默认就是使用的生成器的方式进行.

6. print 可以通过打印'\r'来不进行回车,从而实现刷新当前显示数据

7. lambda指定匿名函数,`filter(lambda x: x+=1, list_a)` 将list_a依次通过lambda函数处理 

8. yield 生成器迭代数据 

	```python
		def fibs(n):
			a,b,i =1,1,1
			while i<n:
				i=i+1
				yield a
				a,b=b,a+b
		lres = ist(fibs(10))
	```

9. 装饰器定义切面函数,适用于日志,审计等场景

	```
		def wraped_func(f):
			def wrap(*args,**kwargs):
				do_something
				return f(*args,**kwargs)
			return wrap
		
		@wraped_func
		def func():
			pass
	```

10. 调试python代码可以通过导入断点 `breakpoint()` (3.7+) `import pdb; pdb.set_trace()`(3.6-)

11. 格式化字符串, 使用f,
	
	```
		n ,x = 1,31  
		a = f"xxx {n} {x/10:.5f} ddd" 
		输出:a = "xxx 1 3.10000 ddd"
	```

12. 排序list使用sorted(), 默认升序,指定reverse=True参数降序

13. 使用set进行数据去重, 可以直接使用 set.add() 方法加入数据到set中,这样产生的数据没有重复, 这样比判断是否再list中再添加的方法时间消耗更低

14. 使用get获取dict中的值并指定默认值,使用setdefault()设置字典项的默认值.