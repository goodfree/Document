### 原型模式（Prototype Pattern）    

用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。  

原型模式可以解决构建复杂对象的资源消耗问题，能够在某些场景下提高创建对象的效率，同时也可以保护原型文件，也就是保护性拷贝，  
某个对象对外可能是只读的，为了防止外部对这个只读对象进行修改。  

◆ 优点：    
原型模式是在内存中二进制流的拷贝，比new性能要好，特别是在循环体内创建大量对象时。

◆ 缺点：  
因为是在内存中拷贝，所以构造函数式不会执行的。所以实际应用时一定要注意这个问题。

◆ 浅拷贝  

  

