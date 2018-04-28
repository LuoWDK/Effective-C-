# Terminology
### 声明式（Declaration）
> 对象&类：告诉编译器名称和类型\
> 函数：告诉编译器函数的参数和返回类型

```
extern int x;                       // 对象声明式
std::size_t numDigits(int number);  // 函数声明式
class Widget;                       // class声明式
```

### 定义式（Definition）
> 对象：编译器为此对象拨发内存的地点\
> function：提供代码本体\
> class：列出成员

```
int x;                              // 对象定义式
std::size_t numDigits(int number)   // 函数定义式
{
    /* Code here*/
}
class Widget                        // class定义式
{
    /* Members here */
}
```

### 初始化（Initialization）
> 给予对象初值
- **构造函数**：初始化用户自定义class的对象
    - default：或没有参数，或每个参数都有缺省值
    - explicit：阻止隐式（implicit）类型转换，允许显式类型转换
- **copy构造函数**：以同型对象初始化自我对象
- **copy assignment操作符**：从另一个同型对象中拷贝其值到自我对象

```
void doSomething(Widget wObject);

class Widget {
public:
    Widget();                                   // default构造函数
    explicit Widget(int x = 0, bool b = true);  // explicit的default构造函数
    explicit Widget(char c);                    // explicit的构造函数
    Widget(const Widget& rhs);                  // copy构造函数
    Widget& operator=(const Widget& rhs);       // copy assignment操作符
};
Widget w1;          // call default构造函数
Widget w2(w1);      // call copy构造函数
w1 = w2;            // call copy assignment操作符
Widget w3 = w2;     // call copy构造函数
doSomething(Widget(28));    // Cast: call Widget(int x = 0, bool b = true) 将int显式转换为Widget，
                            // Pass-by-value: call copy构造函数将 Widget(28)复制到 wObject体内，
                            // Call doSomething(Widget).
doSomething(28);    // Error!!!
                    // Function doSomething(Widget)的参数是 Widget object，而不是int object.
```

### 命名习惯
> lhs：left-hand side\
> rhs：right-hand side\
> pt：pointer to T

```
const Rational operator* (const Rational& lhs, const Rational& rhs);
class Airplane;

a * b;          // a = lhs, b = rhs
Widget* pw;     // pw = "ptr to Widget"
Airplane* pa;   // pa = "ptr to Airplane"
```
