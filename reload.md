# 重载输入输出运算符和友元函数
为什么operator<<运算符重载一定要为友元函数呢？

如果是重载双目操作符（即为类的成员函数），就只要设置一个参数作为右侧运算量，而左侧运算量就是对象本身。。。。。。

而 >>  或<< 左侧运算量是 cin或cout 而不是对象本身，所以不满足后面一点。。。。。。。。就只能申明为友元函数了。。。

如果一定要声明为成员函数，只能成为如下的形式：
```

ostream & operator<<(ostream &output)

{

　　return output;

}
```



所以在运用这个<<运算符时就变为这种形式了：data<<cout;

不合符人的习惯。
```
//输入输出运算符只能用友元函数重载
#include <iostream>
#include <assert.h>
#include <string.h>

using namespace std;

class Complex
{
    double re,im;
public:
    Complex(double r,double i):re(r),im(i)
    {

    }
    Complex()
    {
        re = 0;
        im = 0;
    }
    Complex operator!();
    Complex operator+(const Complex &obj);
    //重载输入输出运算符，只能用友元函数
    friend ostream &operator<<(ostream &os,const Complex &c);
    friend istream &operator>>(istream &is,Complex &c);
};

Complex Complex::operator +(const Complex &obj)
{
    Complex temp;
    temp.re = re + obj.re;
    temp.im = im + obj.im;
    return temp;
}

Complex Complex::operator !()
{
    Complex temp;
    temp.re = -re;
    temp.im = -im;
    return temp;
}

ostream & operator<<(ostream &os,const Complex &c)
{
    os << c.re;
    if(c.im > 0)
        os << "+" << c.im << "i" << endl;
    else
        os << c.im << "i" << endl;
    return os;
}

istream & operator>>(istream &is,Complex &c)
{
    is >> c.re >> c.im;
    return is;
}

int main(int argc,char *argv[])
{
    Complex obj1(1,2),obj2(3,4);
    Complex obj3 = obj1 + !obj2;
    cout << obj3;
    cin >> obj3;
    cout << obj3;
    return 0;
}
```
