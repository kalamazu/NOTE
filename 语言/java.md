#反射
[反射](https://www.cnblogs.com/chanshuyi/p/head_first_of_reflection.html)
**反射就是在运行时才知道要操作的类是什么，并且可以在运行时获取类的完整构造，并调用对应的方法**

反射获取对象的步骤：
- 获取类的Class对象实例
- 根据class对象实例获取Constructor对象
- 使用Constructor对象的newInstance方法获取反射类对象

如果要调用某个方法，需要经过下面步骤：
- 获取方法的Method对象
- 利用invoke方法调用方法
