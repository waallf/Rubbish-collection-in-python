# Rubbish-collection-in-python
python中的垃圾回收机制  
[参考](https://www.cnblogs.com/Xjng/p/5128269.html)  
python 中主要以计数为主，分代收集为辅  
## python中如果有一个对象的引用数为0，python中的虚拟机就会回收这个对象的内存  
### 导致引用计数+1的情况  
1. 对象被创建  a=23  
2. 对象没引用  b=a  
3. 对象被作为参数传到函数中  
4. 对象被作为一个元素，存储在容器中  
###导致引用计数-1的情况   
1. 对象的别名被销毁，例如 del a  
2. 对象的别名被赋予新的对象 a=24  
3. 一个函数离开他的作用域  
4. 对象所在容器被销毁，或从容器中删除对象  

## 循环引用导致内存泄漏  
```
def f2():
  while True():
    c1=classA()
    c2=classA()
    c1.t=c2
    c2.t=c1
    del c1 
    del c2
```  
创建c1,c2后，c1记为1，c2记为1，互相引用后，计数都变为2，del c1,c2后，计数变为1，仍然不会被释放。  
由于循环引用，导致垃圾回收器不会回收他们，就会导致内存泄漏。  
```
deff3():
    # print gc.collect()
    c1=ClassA()
    c2=ClassA()
    c1.t=c2
    c2.t=c1
    del c1
    del c2
    print gc.garbage
    print gc.collect() #显式执行垃圾回收
    print gc.garbage
    time.sleep(10)
if __name__ == '__main__':
    gc.set_debug(gc.DEBUG_LEAK) #设置gc模块的日志
    f3()
```  
垃圾回收的对象会放在gc.garbage列表里面  
gc.collect() 会返回不可达的对象数目  

有三种情况会触发垃圾回收机制：  
1. 调用gc.collect()  
2. 当gc模块的计数器达到阈值的时候  
3. 当程序退出的时候  
### gc中的垃圾回收机制  
这个机制主要作用是发现并处理不可达的垃圾对象  
分代收集：把对象分为三代，对象在创建的时候，放在第一代中，如果在一次一代的垃圾检查中，对象存活下来，会被放到二代中。同理，在一次二代垃圾检查中，该对象存活下来，就被放到三代中。  
可以通过gc.get_count()获取一个长度为3的列表计数器，例如（441，3，0），其中441是距离上一次检测，Python分配内存的数目减去释放内存的数目。内存是分配的不是引用计数增加的。3 是距离上一次二代检查，一代检查的次数，同理，0是指上一次三代检测，二代检测的次数。  
通过gc.get_threshold 获取gc自动回收垃圾的阈值，例如(700,10,10)。 每次计数器的增加，gc模块就会检查增加后的金丝狐是否达到阈值，如果是，那么就会执行对应代数的垃圾检测，重置计数器。  
例如：   
(699,3,0)增加到（700,3,0），gc 模块就会执行gc.collect(0),检查一代对象的垃圾，并重置计数器为（0,4,0）  
(699,9,0),增加到(700,9,0) gc会执行gc.collect(1)，检测一，二代对象，并重置计数器  
（699,9,9）增加到(700,9,9)，gc执行gc.collect(2)， 检测一二三代垃圾，并重置计数器(0,0,0)  

如果循环引用中的两个对象都定义了__del__方法，gc模块不会销毁这些不可达对象，因为gc模块不知道应该先调用那个对象的__del__方法。所以为了安全起见，会把gc模块放到gc.garbage中，但是不会销毁对象。
