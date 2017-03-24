# PHP中的spl_autoload、__autoload和spl_autoload_registe的异同？

原文来自SO的问答[someone explain spl_autoload, __autoload and spl_autoload_register?](http://stackoverflow.com/a/7987085/1969039)

`spl_autoload_register()`允许同时注册多个函数(或者类的静态方法)，PHP会将加入到autoload的函数或静态方法加入栈或队列中，
当我们使用"new Class"的时候PHP会一次引入对应文件达到自动加载的目的。

举个例子:
 
```php
    spl_autoload_register('myAutoloader');

    function myAutoloader($className)
    {
        $path = '/path/to/class';

        include $path.$className.'.php';
    }

    //-------------------------------------

    $myClass = new MyClass();
```

在上面的示例中，"MyClass"为你需要实例化的类名，PHP会将"MyClass"作为字符串传入`spl_autoload_register()`
函数中，它将以变量的形式传入myAutoloader这样便可以"include"这个类或文件了。

这样做的好处是我们不要通过include或require语法显示的引入这个类，是不是很方便？


如上示例所示，我们仅需要简单的去实例化一个类就可以了。当我们通过`spl_autoload_register`注册一个函数时，我们所要做的就是指出我们的类所在的文件目录，剩下的交给PHP就行了，
它会自行调用刚刚注册的函数来引入对应的文件

相比于`__autoload`函数使用`spl_autoload_register`功能将更加简洁，它表现在：

1 我们不要要在每个文件中实现autoload函数

2 `spl_autoload_register`能够注册多个autoload方法提升自动加载的速度

如:

```php
    spl_autoload_register('MyAutoloader::ClassLoader');
    spl_autoload_register('MyAutoloader::LibraryLoader');
    spl_autoload_register('MyAutoloader::HelperLoader');
    spl_autoload_register('MyAutoloader::DatabaseLoader');

    class MyAutoloader
    {
        public static function ClassLoader($className)
        {
             //your loading logic here
        }

	
        public static function LibraryLoader($className)
        {
             //your loading logic here
        }
```
---------------------
关于[spl_autoload][1]，文档中给出的定义是：

> 本函数提供了__autoload()的一个默认实现。如果不使用任何参数调用 spl_autoload_register() 函数，则以后在进行 __autoload() 调用时会自动使用此函数。


通常来说我们的项目文件会集中在目录下；并且应用程序不仅使用`.php`文件，同时也会使用`.inc`扩展名的配置文件，

这个时候我们可以通过PHP的`set_include_path()`函数将包含所有文件的文件目录加入的PHP的引入目录中(include path)

然后将需要引入的所有扩展文件列表注册到`spl_autoload_extensions()`函数中，PHP就会自动查找对于那个扩展名的文件了。

再来个例子:

    set_include_path(get_include_path().PATH_SEPARATOR.'path/to/my/directory/');
    spl_autoload_extensions('.php, .inc');
    spl_autoload_register();

Since spl_autoload is the default implementation of the `__autoload()` magic method, PHP will call spl_autoload when you try and instantiate a new class. 

而apl_autoload其实就对`__autoload`魔法函数的一种默认实现，当我们试图实例化一个对象时，便会调用spl_autoload函数了

  [1]: http://php.net/manual/zh/function.spl-autoload.php
