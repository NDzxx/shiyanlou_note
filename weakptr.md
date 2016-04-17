# weak_ptr 的使用方法及意义

```
//weak_ptr的用处 
//创建时使用 shared_ptr
//使用是使用 weak_ptr 
//防止互相应用导致析构失败 
#include <cstdlib>
#include <vector>
#include <iostream>
using namespace std;
#include <boost/shared_ptr.hpp>
#include <boost/weak_ptr.hpp>
using namespace boost;
class B;
class A
{
  //shared_ptr<B> m_value;//析构函数将不能正常析构 
  weak_ptr<B> m_value;    //析构函数将能正常析构 
public:
  string str; 
  A(){str = "A";cout<<"A 构造"<<endl;}
  void SetValue(shared_ptr<B> p){m_value = p;}
  weak_ptr<B> GetData(){return m_value;}
  ~A(){cout<<"A 析构"<<endl;}
};
class B
{
  //shared_ptr<A> m_value;//析构函数将不能正常析构 
  weak_ptr<A> m_value;    //析构函数将能正常析构 
public:
  string str; 
  B(){str = "A";cout<<"B 构造"<<endl;}
  void SetValue(shared_ptr<A> p){m_value = p;}
  weak_ptr<A> GetData(){return m_value;}
  ~B(){cout<<"B 析构"<<endl;}
}; 
void Do()
{
    cout<<"-----------------------------"<<endl;
    shared_ptr<A> a(new A);
    shared_ptr<B> b(new B);
    a->SetValue(b);
    b->SetValue(a);
    weak_ptr<B> bb = a->GetData();
    if(bb.expired())
    {
      cout<<bb->str<<endl;
    }
    cout<<"-----------------------------"<<endl;
}
int main(int argc, char *argv[])
{
    Do();
    system("PAUSE");
    return EXIT_SUCCESS;
}
```
 

