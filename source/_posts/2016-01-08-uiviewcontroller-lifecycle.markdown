---
layout: post
title: "IOS UIViewController 的生命周期"
date: 2016-01-08 11:28:14 +0800
comments: true
categories: 
---
UIViewController可能是IOS SDK里面用到最多的Object了,所以很重要的是去理解生命周期事件和怎样去使用它们

** 1.生命周期事件顺序 **      
1. {% highlight objc %}-(void)loadView{% endhighlight %}   
2. {% highlight objc %}-(void)viewDidLoad{% endhighlight %}    
3. {% highlight objc %}-(void)viewWillAppear{% endhighlight %}     
4. {% highlight objc %}-(void)viewWillLayoutSubviews{% endhighlight %}       
5. {% highlight objc %}-(void)viewDidLayoutSubviews{% endhighlight %}       
6. {% highlight objc %}-(void)viewDidAppear{% endhighlight %}   

** 2.我们在这些事件里面应该干什么呢? **     
* {% highlight objc %}-(void)loadView{% endhighlight %}-----创建controller管理的views  
      这个函数只有在view controller创建程序完成时才会调用,你可以覆盖该方法去手动创建你自己的子view       
* {% highlight objc %}-(void)viewDidLoad{% endhighlight %}-----当view被load进内存的时候调用  
      当viewController的view创建时调用,记住这个时候view的bounds还不是最终的值，这里是做初始化init和启动一些object的好地方  
* {% highlight objc %}-(void)viewWillAppear{% endhighlight %}----通知view controller 	它的view被添加到view显示树上   
      当view被显示到屏幕上的时候被调用,这个时候view的bounds已经被确定了,但是旋转方向还没有确定,这个函数每次被显示的时候都会被调用，这里不要放那些只需要执行一次的代码     
* {% highlight objc %}-(void)viewWillLayoutSubviews{% endhighlight %}---- 通知view controller布局它的subviews      
      这个方法在每次frame变化时或者方向旋转时都会被标记为需要layout,这是**bounds被确定的的第一步**如果你没有使用自动布局或者你可以在这里改变你subviews的布局         
* {% highlight objc %}-(void)viewDidLayoutSubviews{% endhighlight %}----通知view controller它已经布局了它的subviews     
      你可以在这里添加一些改变布局的代码    
* {% highlight objc %}-(void)viewDidAppear{% endhighlight %} ----通知view controller 被添加到显示树
      这里是一个执行跟显示view相关额外任务的好地方,例如动画,这个方法会在显示动画完成调用,所以到了这一步view已经被用户看到了,在某些情况下,这里还是load data和send request的好地方