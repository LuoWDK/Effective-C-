## Accustoming Yourself to C++
### 用const，enum，inline 替代 #define
> 编译器替代预处理器

- 普通常亮：**const对象** 或 **enums** 替代
```
/** 常量 */
#define ASPECT_RATIO 1.653  // 预处理器将 ASPECT_RATIO替换为 1.653，使得编译器看不到ASPECT_RATIO；
                            //  那么编译的错误信息只会显示 1.653，而不是 ASPECT_RATIO；
                            //  最终导致调试器追踪困难。

// const对象替代
const double AspectRatio = 1.653;


/** class专属常量 */
#define NumTurns 5          // define没有封装性，无法定义class的专属常量
class GamePlayer {
private:
    int scores[NumTurns];
};

// static const替代
class GamePlayer {
private:
    static const int NumTurns;  // NumTurns的声明式：整数常量可"in-class初值设定"
    int scores[NumTurns];
};
const int GamePlayer::NumTurns = 5;     // NumTurns的定义式：必须在实例化GamePlayer对象之前运行

// enum替代
class GamePlayer {
private:
    enum { NumTurns = 5 };  // enum hack：另 NumTurns 成为 5 的几号名称
    int scores[NumTurns];
};
```

- 形似函数的宏（macros）：**inline函数** 替代
```
/** 形似函数的宏 */
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))    // 用 max(a, b) 调用 f

int a = 5, b = 0;
CALL_WITH_MAX(++a, b);      // a被累加两次
CALL_WITH_MAX(++a, b+10);   // a被累加一次

// inline替代
template <typename T>
inline void callWithMax(const T& a, const T& b)         // Pass by reference-to-const
{
    f(a > b ? a : b);                                   // ++、--没有影响
}
```

### const
- const 语法
    - **T const \***：被指物是常量
    - **T\* const**：指针自身是常量
    - **T const \* const**：被指物&指针都是常量
```
T const * pc;        // non-const ptr, const data
T* const pc;         // const ptr, non-const data
const T* const pc;   // const ptr, const data
```
- const 迭代器
    - **const std::vector<int>::iterator**：类似 T* const
    - **std::vector<int>::const_iterator**：类似 const T*
```
std::vector<int> vec;

const std::vector<int>::iterator iter = vec.begin();    // T* const
*iter = 10;                                             // *iter可变
++iter;                                                 // Error!!! iter is const

std::vector<int>::const_iteratot cItre = vector.begin();    // const T*
*cIter = 10;                                                // Error!!! *cIter不可变
++cIter;                                                    // cIter可变
```
- const 函数
```
const Rational operator* (const Rational& lhs, const Rational& rhs);

Rational a, b, c;
if(a * b = c)       // Error!!!
{                   //  a * b返回的const对象不能再赋值
}
```
- const 成员函数
    - **函数重载**：const 对象 & non-const 对象
```
class TextBlock {
public:
    const char& operator[] (std::size_t position) const_iteratot    // for const对象
    {                                                               //  不可改变此对象
        /** A lot code here */                                      //  不可改变返回值
        return text[position];
    }
    char& operator[] (std::size_t position)         // for non-const 对象
    {                                               //  可改变此对象 & 函数的返回值
        return const_cast <char&> (                 // 将 op[] 返回值的 const除掉
            static_cast<const TextBlock&> (*this)   // 为 *this 加上 const
            [position]                              // call const op[]，避免代码重复
        );
    }
private:
    std::string text;
};
```

### 初始化
- 初始化对象
    - **伪初始化**(pseudo-initialization)：
        - 先 call default构造函数初始化参数
        - 再 call copy assignment操作符将参数复制给成员变量
    - **成员初值列**(member initialization list)：
        - Call copy 构造函数初始化成员变量
```
class House {
public:
    House(const std::string& name, const std::string& address);
private:
    std::string theName;
    std::string theAddress;
};
// 伪初始化
House::House(const std::string& name, const std::string& address)
{
    theName = name;         // 先 call default构造函数初始化 theName
                            //  再 call copy assignment操作符将 name 复制给 theName
    theAddress = address;   // 同上类似
}
// 成员初值列
House::House(const std::string& name, const std::string& address)
    : theName(name),        // Call copy构造函数初始化 theName
    theAddress(address)     // 同上类似
{ }
```
- 跨编译单元的初始化次序问题：
    - 用local static对象替换non-local static 对象
```
/////////////////// 跨编译单元使用 non-local static 对象 ////////////////////
/** file_system.h */
class FileSystem {
public:
    std::size_t numDisks() const;
};
extern FileSystem tfs;                  // non-local static 对象
/** directory.h */
class Directory {
public:
    Directory( params );
};
Directory tempDir( params );
/** directory.cpp */
Directory::Directory( params )
{
    /* some code */
    std::size_t disk = tfs.numDisks();  // Maybe Error!!!
                                        //  使用tfs，但不保证已初始化
    /* some code */
}

/////////////////// 跨编译单元使用 local static 对象 ///////////////////
/** file_system.h */
class FileSystem { ... };   // 同上
FileSystem& tfs()           // 此函数替代 tfs 对象；
{                           //  首次call 此函数时，对 fs 初始化；
                            //  后面call 此函数时，直接使用 fs。
    static FileSystem fs;   // 定义并初始化 local static 对象
    return fs;              // 返回一个 reference 指向上述对象
}
/** directory.h */
class Directory { ... };    // 同上
Directory& tempDir()        // 此函数替代 tempDir 对象；
{                           //  首次call 此函数时，对 td 初始化；
                            //  后面call 此函数时，直接使用 td。
    static Directory td;    // 定义并初始化 local static 对象
    return td;              // 返回一个 reference 指向上述对象
}
/** directory.cpp */
Directory::Directory( params )              // 同上，但 reference to tfs 改成 tfs()
{
    /* some code */
    std::size_t disk = tfs().numDisks();
    /* some code */
}
```
