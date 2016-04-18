# 遇见C++ Lambda



C++中，一个lambda表达式表示一个可调用的代码单元。我们可以将其理解为一个未命名的内联函数。它与普通函数不同的是，lambda必须使用尾置返回来指定返回类型。
例如调用<algorithm>中的std::sort，ISO C++ 98 的写法是要先写一个compare函数：
```
boolcompare(int&a,int&b)
{
returna>b;//降序排序
}
```

然后，再这样调用：
```
sort(a,a+n,compare);
```
然而，用ISO C++ 11 标准新增的Lambda表达式，可以这么写：
```
sort(a,a+n,[](inta,intb){returna>b;});//降序排序
```
这样一来，代码明显简洁多了。


遇见C++ Lambda

Written by Allen Lee

 

If you die when there's no one watching, and your ratings drop and you're forgotten.

– Marilyn Manson, Lamb Of God

 

##生成随机数字  
