## Constructors, Destructors, and Assignment Operators
### 编译器自动生成
- **default构造函数**：
    - 未声明任何构造函数
- **析构函数**：
    - 未声明析构函数
    - base class为virtual，则为virtual；否则，为non-virtual
- **copy构造函数**：
    - 拷贝源对象的每个non-static成员变量
- **copy assignment操作符**：
    - 不含reference成员
    - 不含const成员
    - base class的copy assignment操作符不为private

### 拒绝编译器版copy构造函数 & copy assignment操作符
- 阻止编译器自动生成：
    - 将它们声明为 private；
- 阻止 member 函数和 friend 函数 call 它们：
    - 不定义它们。

```
class Uncopyable {
protected:
    Uncopyable() {}                             // default构造函数
    ~Uncopyable() {}
private:
    Uncopyable(const Uncopyable&);              // copy构造函数
    Uncopyable& operator=(const Uncopyable&);   // copy assignment操作符
};

class HomeForSale: private Uncopyable {         // class不再声明 copy 构造函数
    /** code here */                            //  或 copy assignment构造函数
};
```

### virtual析构函数
- 需要 virtual 析构函数的 class
    - 带多态性质的(polymorphic) base class
    - 带 virtual 函数的的 class

### 析构函数抛出异常
- 先给用户处理异常的机会
- 再确保析构函数不吐出异常：
    - 若抛出异常，则应捕捉，然后吞下或结束程序

```
class DBConnection {                // 负责连接数据库的 class
public:
    static DBConnection create();   // 此函数返回 DBConnection 对象
    void close();                   // 关闭连接，失败则抛出异常
};

class DBConn {                      // 管理 DBConnection 对象的 class
public:
    void close()                    // 供客户关闭连接
    {
        db.close();
        close = true;
    }
    ~DBConn()                       // 确保数据库连接总会被关闭
    {
        if (!closed) {              // 给客户处理异常的机会
            try {                   // 若客户未关闭连接，则关闭连接
                db.close();
            }
            catch (...) {           // 若关闭连接失败
                /** 记录下来并结束程序 **/
                /** 或吞下异常 **/
            }
        }
    }
};
```

### 构造和析构过程不 call virtual 函数
> 在 derived class 对象的 base class 构造期间，对象类型是 base class 而不是 derived class，virtual 函数会被编译器解析至 base class。

```
/******************** 在析构过程 call virtual 函数 *****************************/
class Transaction {                             // 所有交易的 base class
public:
    Transaction();
    virtual void logTransaction() const = 0;    // 因类型而变化的日志记录
};
Transaction::Transaction()
{
    /* code here */
    logTransaction();       // 此处不是 virtual 的 logTransaction，。
}                           //  而是 Transaction 的 logTransaction。

class BuyTrasaction: public Transaction {       // derived class 1
public:
    virtual void logTransaction() const;
};
class SellTrasaction: public Transaction {      // derived class 2
public:
    virtual void logTransaction() const;
};

/******************** 通过信息传递，阻止析构调用 virtual *******************/
class Transaction {
public:
    explicit Transaction(const std::string& logInfo);
    void logTransaction(const std::string& logInfo) const;  // non-virtual
};
Transaction::Transaction(const std::string& logInfo)
{
    logTransaction(logInfo);                            // non-virtual call
}

class BuyTrasaction: public Transaction {
public:
    BuyTransaction( parameters ): Transaction(createLogString( parameters ))
    {                                                   // 将信息传给 base class 的构造函数
        /* code here */
    }
private:
    static std::string createLogString( parameters );   // 传值辅助函数
};
```

### *reference to* *this
- 令赋值操作符返回：
    - 所有内置类型和标准程序库提供的类型（string、vector、complex、tr1::shared_ptr）都遵守此协议

```
class Widget {
public:
    Widget& operator=(const Widget& rhs)
    {
        /* code here */
        return *this;       // 返回指向当前对象的 reference
    }
    Widget& operator+=(const Widget& rhs)
    {
        /* code here */
        return *this;       // 返回指向当前对象的 reference
    }                       //  +=、-=、*= 同理
};
```

### 自我复制
> 对象被赋值给自己

```
class Bitmap { };
class Widget {
private:
    Bitmap* pb;
};

Widget& Widget::operater=(const Widget& rhs)    // 不安全的 copy assignment 操作
{
    delete pb;                                  // Error!pb 和 rhs 可能是同一对象
    pb = new Bitmap(*rhs.pb);                   //  则此处 rhs 的 bitmap 副本为空
    return *this;
}
```
- **验证测试**（identity test）
```
Widget& Widget::operater=(const Widget& rhs)
{
    if (this == &rhs)           // 验证测试
        return *this;           // 若是自我复制，不做赋值操作
    delete pb;
    pb = new Bitmap(*rhs.pb);
    return *this;
}
```
- **调整语句顺序**
```
Widget& Widget::operater=(const Widget& rhs)
{
    Bitmap* pOrig = pb;         // 拷贝原始 bitmap
    pb = new Bitmap(*rhs.pb);   // pb 指向 rhs 的 bitmap 副本
    delete pOrig;               // 删除原始 bitmap
    return *this;
}
```
- **copy-and-swap**
```
class Widget {
    void swap(Widget& rhs);                     // 交换 *this 和 rhs 的数据
};
Widget& Widget::operator=(const Widget& rhs)    // pass by reference
{
    Widget temp(rhs);                           // 为 rhs 保留副本
    swap(temp);                                 // 将 *this 和副本交换
    return *this;
}
Widget& Widget::operator=(Widget rhs)           // pass by value
{
    swap(rhs);                                  // 将 *this 和 rhs 交换
    return *this;
}
```

### copying 函数
> 既 copy 构造函数和 copy assignment 函数

- 确保复制“对象内所有成员变量”及“base class 成分”
- 使用 init 函数消除 copying 函数之间的代码重复
