# 下标运算符重载
 下标运算符重载    

我们常用下标运算符operator[]来访问数组中的某个元素.它是一个双目运算符,第一个运算符是数组名,第二个运算符是数组下标.在类对象中,我们可以重载下标运算符,用它来定义相应对象的下标运算.

注意,C++不允许把下标运算符函数作为外部函数来定义,它只能是非静态的成员函数.下标运算符定义的一般形式:
      T1 T::operator[](T2)；
其中,T是定义下标运算符的类,它不必是常量.T2表示下标,它可以是任意类型,如整形,字符型或某个类. T1是数组运算的结果.它也可以是任意类型,但为了能对数组赋值,一般将其声明为引用形式.在有了上面的定义之后,可以采用下面两种形式之任一来调用它:
     x[y]或 x.operator[](y)
x的类型为T,y的类型为T2.
    下面来看一个简单的例子：
```

#include <iostream.h>

 

class aInteger

{

public:

   aInteger(int size)

   {

       sz = size;

       a = new int[size];

   }

 

   int& operator [] (int i);

 

   ~aInteger()

   {

       delete []a;

   }

private:

   int* a;

   int sz;

};

 

int& aInteger::operator [](int i)

{

   if (i < 0 || i > sz)

   {

       cout << "error, leap the pale" << endl;

   }

 

   return a[i];

}

 

int main()

{

   aInteger arr(10);

   for (int i = 0; i < 10; i++)

   {

       arr[i] = i+1;

       cout << arr[i] << endl;

   }

 

   int n = arr.operator [] (2);

   cout << "n = " << n << endl;

 

   return 0;

}

 ```

在整形数组ainteger中定义了下标运算符，这种下标运算符能检查越界的错误。现在使用它：
  ainteger ai(10);
  ai[2]=3;
  int i=ai[2];
对于ai[2]=3,他调用ai.operator(2),返回对ai::a[2]的引用，接着再调用缺省的赋值运算符，把3的值赋给此引用，因而ai::a[2]的值为3。注意，假如返回值不采用引用形式，ai.operator(2)的返回值是一临时变量，不能作为左值，因而，上述赋值会出错。对于初始化i=ai[2],先调用ai.operator(2)取出ai::a[2]的值。然后再利用缺省的复制构造函数来初始化i.

实例：
```

//**********************************

//***    下标重载运算符       ***

//**********************************

 

#include <iostream.h>

 

class charArray

{

public:

   charArray(int len)

   {

       length = len;

       buffer = new char[length];

   }

 

   int getLength()

   {

       return length;

   }

 

   char& operator[](int i);

 

   ~charArray()

   {

       delete[]buffer;

   }

private:

   int length;

   char* buffer;

};

 

char& charArray::operator[] (int i)

{

   static char ch = 0;

   if (i >= 0 && i < length)

       return buffer[i];

   else

   {

       cout << "out of range!" << endl;

       return ch;

   }

}

 

int main()

{

   int i;

   charArray str1(7);

   char* str2 = "string";

   for (i = 0; i < 7; i++)

   {

       str1[i] = str2[i];

   }

 

   for (i = 0; i < 7; i++)

   {

       cout << str1[i];

   }

   cout << str1.getLength() << endl;

 

   return 0;

}  
```