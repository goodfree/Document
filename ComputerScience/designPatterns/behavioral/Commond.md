### 命令模式  
调用者执行命令，每一个具体命令 包含一个具体执行者，具体执行者真正执行命令；  
调用者不会直接接触执行者；  

● Command（抽象命令类）  
抽象命令类一般是一个抽象类或接口，在其中声明了用于执行请求的execute()等方法，通过这些方法可以调用请求接收者的相关操作。    

● ConcreteCommand（具体命令类）    
具体命令类是抽象命令类的子类，实现了在抽象命令类中声明的方法，它对应具体的接收者对象，将接收者对象的动作绑定其中。  
在实现execute()方法时，将调用接收者对象的相关操作(Action)。    

● Invoker（调用者）  
调用者即请求发送者，它通过命令对象来执行请求。一个调用者并不需要在设计时确定其接收者，因此它只与抽象命令类之间存在关联关系。  
在程序运行时可以将一个具体命令对象注入其中，再调用具体命令对象的execute()方法，从而实现间接调用请求接收者的相关操作。  

● Receiver（接收者）  
接收者执行与请求相关的操作，它具体实现对请求的业务处理。  

◆ 命令模式的优点：  
● 类间解耦  
Invoker（调用者）与Receiver（接收者）之间没有任何依赖关系，调用者实现功能时只需调用Command抽象类的execute方法就可以，不需要了解到底是哪个接收者执行。  

● 可扩展性  
Command的子类可以非常容易地扩展，而调用者Invoker和高层次的模块Client不产生严重的代码耦合。  

● 命令模式结合其他模式会更优秀  
命令模式可以结合责任链模式，实现命令族解析任务；结合模板方法模式，则可以减少Command子类的膨胀问题。  
缺点：Command的子类会产生膨胀的问题。  

