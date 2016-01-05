---
layout: post
title: "OC 运行时相关知识"
date: 2015-12-31 16:08:53 +0800
comments: true
categories: 
---
1.动态添加方法到某个类
1.1 首选创建一个IMP
例如:
{% highlight objc %}
UIColor * tttttttt(id self,SEL cmd,NSString *str){
	NSLog(@"%@",str);
	return [UIColor whiteColor];
}
{% endhighlight %}
1.2 首先获得Class实例
{% highlight objc %}
Class newClass = NSClassFromString:(@"NSColor");
  {% endhighlight %}
1.3 运行class_addMethod方法
{% highlight objc %}
class_addMethod(newClass,@selector(testMethod:),(IMP)ttt,"@@:");
id instance = [[newClass alloc]initWithDomain:@"some domain" code:0 userInfo:nil];
UIColor *color = [instance performSelector:@selector(testMetaClass:) withObject:@"1231111111111"];
    NSLog(@"%@",color);
 {% endhighlight %}
2.动态添加属性
2.1如果实现了NSKeyValueCoding 协议可以使用
{% highlight objc %}
[setValue:id obj forUndefinedKey:id key]和[valueForUndefinedKey:id key]
{% endhighlight %}
配合使用
2.2使用 objc_getAssociatedObject
运行时方法，通常添加一个category来添加
{% highlight objc %}
@implementation NSObject
(ExampleCategoryWithProperty) @dynamic objectTag; - (id)objectTag {
return objc_getAssociatedObject(self, ObjectTagKey); } -
(void)setObjectTag:(id)newObjectTag {
objc_setAssociatedObject(self, ObjectTagKey,
newObject,OBJC_ASSOCIATION_RETAIN_NONATOMIC); }
{% endhighlight %}
2.3
因为运行时不能添加实例变量,如果你有添加实例变量 可以这么做
{% highlight objc %}
#include  
#import  
@interface SomeClass : NSObject { 
 NSString *_privateName; 
}
@end 
@implementation SomeClass 
- (id)init { 
 self = [super init]; 
 if (self) _privateName = @"Steve";
  return self; 
} 
@end 
NSString *nameGetter(id self, SEL _cmd) { 
 Ivar ivar = class_getInstanceVariable([SomeClass class], "_privateName"); 
 return object_getIvar(self, ivar); 
} 
 void nameSetter(id self, SEL _cmd, NSString *newName) { 
 Ivar ivar = class_getInstanceVariable([SomeClass class], "_privateName"); 
 id oldName = object_getIvar(self, ivar); 
 if (oldName != newName) object_setIvar(self, ivar, [newName copy]);
 }
 int main(void) { 
 @autoreleasepool { 
 objc_property_attribute_t type = { "T", "@"NSString"" }; 
 objc_property_attribute_t ownership = { "C", "" }; // C = copy 
 objc_property_attribute_t backingivar = { "V", "_privateName" }; 
 objc_property_attribute_t attrs[] = { type, ownership, backingivar }; class_addProperty([SomeClass class], "name", attrs, 3); 
class_addMethod([SomeClass class], @selector(name), (IMP)nameGetter, "@@:"); class_addMethod([SomeClass class], @selector(setName:), (IMP)nameSetter, "v@:@"); 
id o = [SomeClass new]; 
NSLog(@"%@", [o name]); [o setName:@"Jobs"]; NSLog(@"%@", [o name]); 
 } 
}
{% endhighlight %}
3.交换方法
主要使用{% highlight objc %}class_getInstanceMethod(){% endhighlight %}获取Method实例,然后通过{% highlight objc %}class_addMethod和class_replaceMethod，method_exchangeImplementations(){% endhighlight %}方法来进行实现{% highlight objc %}
+(BOOL)swizzleMethod:(SEL)origSelector withMethod:(SEL)newSelector
{
Method origMethod = class_getInstanceMethod(self,origSelector);
Method newMethod = class_getInstanceMethod(self,newSelector);
if(origMethod && newMethod)
{
if(class_addMethod(self,origSelector,method_getImplementation(newMethod),method_getTypeEncoding(newMethod)))
{
class_replaceMethod(self,newSelector,method_getImplementation(origMethod),methodGetTypeEncoding(origMethod));
}else
{
method_exchangeImplementations(oriMethod,newMethod);
}
return YES ;
}
return NO ;
}{% endhighlight %}
然后实现该类的+load方法 再里面调用
{% highlight objc %}
+(load){
[self swizzleMethod:@selector(sendEvent:) withMethod:@selector(swizzledSendEvent)];
}
{% endhighlight %}

