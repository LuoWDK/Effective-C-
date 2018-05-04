## Resource Management
### 资源管理类
> 获得资源后立刻放入管理对象：Resource Acquisition Is Initialization，**RAII**\
> 管理对象运用析构函数确保资源被释放

- 类指针对象：**auto_ptr**
    - 析构函数自动对对象 call delete
    - call coping 函数时，rhs 指针变成 null，lhs 指针获得资源的唯一拥有权
    - 避免多个 auto\_ptr 同时指向同一对象
    - 重载了指针取值操作符：operator->，operator*
```
class Investment {              // root class
public:
    bool isTaxFree() const;
};
Investment* createInvestment(); // 返回指针，指向动态分配的 Investment 对象

void f( )
{
    std::auto_ptr<Investment> pInv1 (createInvestment( ));  // pInv1 指向返回物
    std::auto_ptr<Investment> pInv2 (pInv1);                // 现在 pInv2 指向对象,
                                                            //  pInv1 == null
    pInv1 = pInv2;                                          // 现在 pInv1 指向对象，
                                                            //  pInv2 == null
    bool taxable1 = !(pInv1->isTaxFree());  // 由 operator-> 访问资源
    bool taxable2 = !((*pInv1).isTaxable);  // 由 operator* 访问资源
}       // pInv1 和 pInv2 被销毁
        //  pInv1 的析构函数自动删除所指对象
        //  pInv2==null 没有对象需要删除
```
- RCSP：**tr1::shared_ptr**
    - 引用计数型智慧指针（Reference-Counting Smart Pointer）
    - 类垃圾回收：允许多个对象指向某资源，并在无人指向它时 call delete
    - 避免环状引用
    - 重载了指针取值操作符：operator->，operator*
```
/************* tr1::shared_ptr *****************/
void f( )
{
    std::tr1::shared_ptr<Investment> pInv1 (createInvestment( ));   // pInv1 指向返回物
    std::tr1::shared_ptr<Investment> pInv2 (pInv1);                 // pInv1 和 pInv2 指向同一对象
    pInv1 = pInv2;                                                  // 同上
}       // pInv1 和 pInv2 被销毁，他们所指对象也被销毁
```

### 访问资源管理类的原始资源
- 取得其所管理之资源
    - 显式转换：安全
    - 隐式转换：用户习惯
```
FontHandle getFont();
void releaseFont(FontHandle fh);

class Font {
public:
    explicit Font(FontHandle fh) : f(fh) { }    // 获得资源，pass-by-value
    ~Font( ) { releaseFont(f); }                // 释放资源
    FontHandle get() const { return f; }        // 显式转换
    operator FontHandle() const { return f; }   // 隐式转换
private:
    FontHandle f;
};
```

### new & delete
- new 对应 delete
- new[] 对应 delete[]

### newd 对象 & 智能指针
> 分离资源被创建和资源被转换为资源管理对象

```
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);

processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
        // Error! 由于编译器处理参数的次序未知，
        //  可能在“new Widget”与“call tr1::shared_ptr的构造函数”的之间调用 priority 

std::tr1::shared_ptr<Widget> pw(new Widget);    // 先创建 Widget
processWidget(pw, priority());                  // 再传给 processWidget
```
