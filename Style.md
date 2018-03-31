# Google的C++编码规范
### 1.头文件
##### 1.1.define保护
> 所有头文件都应该使用#define来防止头文件被多重包含，命名格式是：  
> ```
> <PROJECT>_<PATH>_<FILE>_H_
> ```
为保证唯一性，头文件的命名应该基于所在项目源代码树的全路径。例如，项目foo中的头文件foo/src/bar/baz.h 可按如下方式保护:
```
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
...
#endif // FOO_BAR_BAZ_H_
```
##### 1.2.include声明
> 尽量减少.h文件的#include的数量。
我们在代码里面用到类File则头文件只要include声明类File。

##### 1.3.函数内联
> 内联函数的好处是在编译的时候直接嵌套减少语句跳转带来的开销；
> 当函数小于10行的时候可将其定义为内联函数，内联函数不要包含控制语句。

##### 1.4.包含文件的名称和次序
> 建议次序：C库，C++库，其他库的.h，项目内的.h。
如：google-swesome-project/src/foo/internal/fooserver.cc的包含顺序：
```
#include "foo/public/fooserver.h"

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "base/basictypes.h"
#include "foo/public/bar.h"
```
### 2.作用域
##### 2.1.命名空间
> using namespace std尽量不要在.h头文件使用，在.cpp实现文件中使用  
> std是一个命名空间，里面包含很多的关键字，里面的一些关键字如max min之类很容易在自己编程> 的时候冲突，所以在比较大的项目里面要谨慎使用。  
> 优点：省事。
> 缺点：容易冲突，引入大量不需要的命名空间，编译之后占用大量内存。  
举例子：
```
int x;  
std::cin >> x ;  
std::cout << x << std::endl;  
```
```
using std::cin;  
using std::cout;  
using std::endl;  
int x;  
cin >> x;  
cout << x << endl;  
```
```
using namespace std;  
int x;  
cin >> x;  
cout << x << endl;  
```
##### 2.2.嵌套类
> 嵌套类符合局部使用规则，只是不能在其他头文件中前置声明，尽量不要用public；

##### 2.3.局部变量和全局变量
> 局部变量尽量在最小作用域内，并在变量声明时进行初始化；
> 局部变量是一个对象的时候不需要在最小作用域内，避免重复的调用构造函数和析构函数；
> 使用命名空间中的非成员函数或静态成员函数，尽量不要使用全局函数，也就是命名空间里尽量不要用public；
> 多线程中禁止使用class类型的全局变量，避免不明确行为导致的bug；

### 3.C++类
##### 3.1.构造函数
> 构造函数进行各种几乎没有实际意义初始化操作；
> 不要再构造函数中调用虚函数，也不要在无法报出错误十进行可能失败的初始化；
> 编译器提供的默认构造函数不会对变量进行初始化，如果定义了其他构造函数，那么编译器不再提供默认构造函数；

##### 3.2.隐式类型转换
> 在类的操作里，不要定义隐式类型转换。对于转换运算符和单参数构造函数，使用explicit关键字；
隐式类型转换：
```
class Test
{
  explicit Test(int a);
  ……
  
}
// 这个操作是合法的；
Test(10);

// 这个操作在没有关键字explicit的时候是可以运行的，因为编译器会将int自动类型转换为Test类型，但是加入关键字后就是错误的；
Test() my_test = 10;
```

##### 3.2.结构体和类
> 只有数据时使用struct，其他一概使用class。

##### 3.3.继承
> 所有继承必须是public的；
> 不要过度使用实现继承(is a)，即使代码量会小一点，组合(has a)常常更合适一些。

##### 3.4.接口
> 纯接口：
> 1）只有纯虚函数和静态函数（析构函数除外）；
> 2）没有非静态数据成员；
> 3）没有定义任何构造函数；
> 4）如果是子类，也只能继承满足上述条件并以Interface为后缀的类；

##### 3.5.存取控制
> 将所有的数据成员声明为private，除非是static const类型成员。并提供相关存取函数。

##### 3.6.声明顺序
> public在private之前，成员函数在数据成员前；
一般的次序为：
1）typedefs和enums；
2）常量；
3）构造函数；
4）析构函数；
5）成员函数，含静态成员函数；
6）数据成员，含静态数据成员；

### 4.函数
##### 4.1.函数的参数顺序
> 函数的参数顺序：输入参数在先，后跟输出参数。

##### 4.2.函数体
> 函数体尽量短小, 紧凑, 功能单一，尽量控制在10行左右，否则可读性很差。

##### 4.3.引用参数
> 所有按引用传递的参数必须加上const，避免参数被无意中修改。
```
void main()  
{  
  int a = 10;  
  int b = 20;  
  add(a, b);     
}  
 
int my_add(int &a, const int &b)  
{
  // a += 10;         // 合法的，形参a为引用，可以改变它的值，执行之后a为20，其他函数再调用a的时候也是20  
  // b += 10; 	      // 非法的，形参a为常引用，不能改变它的值  
  return (a + b);  
}  
```

##### 4.4.函数重载
> 若要使用函数重载，则必须能让读者一看调用点就胸有成竹，而不用花心思猜测调用的重载函数到底是哪一种。

##### 4.5.缺省参数
> 禁止使用缺省函数。
```
// 缺省函数的作用
```

##### 4.6.函数返回类型后置语法
> 只有在常规写法 (返回类型前置) 不便于书写或不便于阅读时使用返回类型后置语法。
```
// 函数返回类型后置的作用
```

##### 4.7.类型转换
> 使用static_cast<>()等C++的类型转换，不要使用int y = (int)x。

##### 4.8.NULL和0
> 整数用0，实数用0.0，指针用NULL，字符串用'\0'。

### 5.命名约定
##### 5.1 通用规则
> 函数名、变量名、文件名应具有描述性，不要过度缩写，出了num，info，pc之类大家都知道的名字。
##### 5.2.文件命名
> 文件名要全部小写，可以包含下划线_或短线-，按项目约定来，如：my_first_class.h。

##### 5.3.类型命名
> 类型命名每个单词以大写字母开头，不包含下划线，如：class TableInfo。
> 类型包括——类、结构体、typedef、枚举。

##### 5.4.变量命名
> 变量名一律小写，单词间以下划线相连，类的成员变量以下划线结尾，如：string table_name。
> 全局变量以g_为前缀，如：g_num_of_table；

##### 5.5.常量命名
> 命名时以k开头，如：kDayInAWeek。

##### 5.6.函数命名
> 普通函数大小写混合，存取函数要求与变量名匹配：MyExcitingFunction()、set_my_exciting_member_variable()；
> 两种格式分别是每个单词大写开头和单词全部小写加下划线；

##### 5.7.命名空间
> 命名空间的名词是全小写，其命名基于项目名词和目录结构，如：cin。

##### 5.8.枚举命名
> 枚举的命名和常量或宏一致，如：kEnumName或者ENUM_NAME。

##### 5.9.宏命名
> 类似枚举命名一样全部大写、使用下划线，如：MY_NAME。


### 6.代码注释
##### 6.1.注释风格
> 使用 // 或 /* */，但是 // 更常用，统一就行。

##### 6.2.类注释
> 每个类的定义都要附带一份注释，描述类的功能和用法，除非它的功能相当明显。

##### 6.3.函数注释
> 函数声明处的注释描述函数功能，定义处的注释描述函数实现。

##### 6.4.变量注释
> 通常变量名本身足以很好说明变量用途，某些情况下，也需要额外的注释说明。

### 7.格式
##### 7.1.行长度
> 每一行代码字符数不超过80。

##### 7.2.缩进
> 只使用空格，每次缩进 2 个空格。

##### 7.3.函数声明与定义
> 返回类型和函数名在同一行，参数也尽量放在同一行，如果放不下就对形参分行，分行方式与函数调用一致。

##### 7.4.函数调用
> 要么一行写完函数调用，要么在圆括号里对参数分行，要么参数另起一行且缩进四格。如果没有其它顾虑的话，尽可能精简行数，比如把多个参数适当地放在同一行里。
```
bool retval = DoSomething(argument1, argument2, argument3);  
  
bool retval = DoSomething(averyveryveryverylongargument1,  
                          argument2, argument3);  
```

##### 7.5.条件语句
> 倾向于不在圆括号内使用空格。关键字 if 和 else 另起一行。
```
if (condition) {  // 圆括号里没有空格.
  ...  // 2 空格缩进.
} else if (...) {  // else 与 if 的右括号同一行.
  ...
} else {
  ...
}
```
##### 7.6.循环和选择语句
> switch 语句可以使用大括号分段，以表明 cases 之间不是连在一起的；
> 在单语句循环里，括号可用可不用。空循环体应使用 {} 或 continue。
```
// 选择语句
switch (var) {
  case 0: {  // 2 空格缩进
    ...      // 4 空格缩进
    break;
  }
  case 1: {
    ...
    break;
  }
  default: {
    assert(false);
  }
}

// 循环语句
for (int i = 0; i < kSomeNumber; ++i)
  printf("I love you\n");

for (int i = 0; i < kSomeNumber; ++i) {
  printf("I take it back\n");
}

// 空循环体
while (condition) {
  // 反复循环直到条件失效.
}
for (int i = 0; i < kSomeNumber; ++i) {}  // 可 - 空循环体.
while (condition) continue;  // 可 - contunue 表明没有逻辑.
```

##### 7.8.指针和引用表达式
> 句点或箭头前后不要有空格；
> 指针/地址操作符 (*, &) 之后不能有空格。
```
x = *p;
p = &x;
x = r.y;
x = r->y;
```
##### 7.9.布尔表达式
> 果一个布尔表达式超过 标准行宽，断行方式要统一一下。
```
if (this_one_thing > this_other_thing &&
    a_third_thing == a_fourth_thing &&
    yet_another && last_one) {
  ...
}
```
##### 7.10.函数返回值
> 不要在 return 表达式里加上非必须的圆括号。
```
return result;                  // 返回值很简单, 没有圆括号.
// 可以用圆括号把复杂表达式圈起来, 改善可读性.
return (some_long_condition &&
        another_condition);
// 错误的两个例子
return (value);                // 毕竟您从来不会写 var = (value);
return(result);                // return 可不是函数！
```

##### 7.11.变量及数组初始化
> 用 =, () 和 {} 均可。
```
int x = 3;
int x(3);
int x{3};
string name("Some Name");
string name = "Some Name";
string name{"Some Name"};
```

##### 7.12.类格式
> 访问控制块的声明依次序是 public:, protected:, private:, 每个都缩进 1 个空格。
```
class MyClass : public OtherClass {
 public:      // 注意有一个空格的缩进
  MyClass();  // 标准的两空格缩进
  explicit MyClass(int var);
  ~MyClass() {}

  void SomeFunction();
  void SomeFunctionThatDoesNothing() {
  }

  void set_some_var(int var) { some_var_ = var; }
  int some_var() const { return some_var_; }

 private:
  bool SomeInternalFunction();

  int some_var_;
  int some_other_var_;
};
```

##### 7.13.构造函数初始化列表
> 构造函数初始化列表放在同一行或按四格缩进并排多行。
```
// 如果所有变量能放在同一行:
MyClass::MyClass(int var) : some_var_(var) {
  DoSomething();
}

// 如果不能放在同一行,
// 必须置于冒号后, 并缩进 4 个空格
MyClass::MyClass(int var)
    : some_var_(var), some_other_var_(var + 1) {
  DoSomething();
}

// 如果初始化列表需要置于多行, 将每一个成员放在单独的一行
// 并逐行对齐
MyClass::MyClass(int var)
    : some_var_(var),             // 4 space indent
      some_other_var_(var + 1) {  // lined up
  DoSomething();
}

// 右大括号 } 可以和左大括号 { 放在同一行
// 如果这样做合适的话
MyClass::MyClass(int var)
    : some_var_(var) {}
```

##### 7.14.操作符
```
// 赋值运算符前后总是有空格.
x = 0;

// 其它二元操作符也前后恒有空格, 不过对于表达式的子式可以不加空格.
// 圆括号内部没有紧邻空格.
v = w * x + y / z;
v = w*x + y/z;
v = w * (x + z);

// 在参数和一元操作符之间不加空格.
x = -5;
++x;
if (x && !y)
  ...
```
