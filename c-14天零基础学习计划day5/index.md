# C++14天零基础学习计划Day5


# C++14天零基础学习计划Day5

## 第 5 天：C++11/17 核心特性（承上启下）

#### 学习目标

掌握 C++11/17 基础特性，为 C++20 铺垫（这些特性 Qt 已广泛使用）。

#### 核心内容

1. 自动类型推导：auto（Qt 中auto btn = new QPushButton;）、decltype。
2. 范围 for 循环：for (auto& elem : vec)（对比 C 的 for 循环）。
3. 智能指针：std::unique_ptr/std::shared_ptr（替代裸指针，Qt 的 QSharedPointer 类似）。
4. Lambda 表达式：
  	
  		[]() { /* 代码 */ }（Qt 信号槽常用：connect(btn, &QPushButton::clicked, [](){});）。

#### 练习

* 用auto+ 范围 for 遍历std::vector，输出元素。
* 用std::unique_ptr管理一个自定义类对象，对比裸指针的优势。
* 写一个 Lambda 表达式作为 Qt 按钮的点击槽函数。

------
---
---

### 1. 自动类型推导：auto 与 decltype

##### 1.1 auto：让编译器替你推导类型

auto 关键字在 C++11 中获得了全新的含义——`自动类型推导`。编译器根据初始化表达式的类型，自动推断变量的类型。

核心作用：让编译器根据变量的初始化值自动推导类型，避免写冗长的类型名。

###### 基本语法：

```cpp
auto x = 42;           // x 为 int
auto pi = 3.14159;     // pi 为 double
auto str = "hello";    // str 为 const char*
```

###### 在 Qt 中的典型用法：

Qt 中大量使用 new 创建对象，使用 auto 可以避免重复书写冗长的类型名。

```cpp
// 传统方式
QPushButton *btn = new QPushButton("Click me");

// 使用 auto
auto btn = new QPushButton("Click me");   // btn 类型为 QPushButton*
```

当与 Qt 的容器迭代器配合时，auto 更能简化代码：

```cpp
QList<QString> list = {"one", "two", "three"};
// 传统迭代器
for (QList<QString>::iterator it = list.begin(); it != list.end(); ++it) { ... }

// 使用 auto
for (auto it = list.begin(); it != list.end(); ++it) { ... }
```

###### 使用注意：

* auto 变量必须初始化（编译器需要通过初始值推导类型）；
* 不能用于函数参数类型（C++14 支持 auto 返回值，C++20 支持 auto 参数）；
* Qt 中大量用于简化控件指针、容器迭代器的声明。


##### 1.2 decltype：获取表达式的类型

decltype 用于"查询" 一个表达式的类型（而非推导变量类型）。它常用于模板元编程或需要声明与表达式同类型的场景。

###### 基本语法：

```cpp
#include <iostream>
#include <QString>

int main() {
    QString str = "Qt";
    // 推导str的类型，定义同类型的变量
    decltype(str) str2 = "C++11";  
    
    int a = 10;
    decltype(a + 3.14) b = 5.6;   // a+3.14是double，故b为double
    
    std::cout << typeid(b).name() << std::endl;  // 输出double（不同编译器输出可能略有差异）
    return 0;
}
```

###### 在 Qt 中的应用：

decltype 可与 auto 结合，在泛型代码中推导返回类型。例如，在 Qt 的模型/视图编程中，可能需要根据数据源动态决定变量类型。


### 2. 范围 for 循环：简洁遍历容器

C++11 引入了基于范围的 for 循环（range-based for loop），使得遍历容器变得异常简洁。

核心作用：简化容器遍历，替代 C 风格的下标 / 迭代器循环，Qt 容器（QVector/QList 等）完美支持。

###### 基本语法：

```cpp
std::vector<int> vec = {1, 2, 3, 4};
for (auto& elem : vec) {
    elem *= 2;          // 使用引用修改元素
}

for (const auto& elem : vec) {
    qDebug() << elem;   // 只读访问
}
```

###### 对比示例

```cpp
#include <QVector>
#include <iostream>

int main() {
    QVector<int> vec{10,20,30,40};
    
    // 1. C风格for循环（传统方式）
    for (int i = 0; i < vec.size(); ++i) {
        std::cout << vec[i] << " ";
    }
    std::cout << std::endl;
    
    // 2. C++11范围for循环（值拷贝）
    for (auto elem : vec) {
        elem *= 2;  // 仅修改拷贝，原容器无变化
    }
    
    // 3. C++11范围for循环（引用修改，推荐）
    for (auto& elem : vec) {  // &表示引用，直接操作原元素
        elem *= 2;  // 原容器元素被修改为20,40,60,80
    }
    
    // 4. 只读引用（避免拷贝，提升性能）
    for (const auto& elem : vec) {
        std::cout << elem << " ";  // 输出20 40 60 80
    }
    
    return 0;
}
```

与 C 传统循环对比：

* C 风格（或旧 C++）需要手动管理索引或迭代器，容易出错。
* 范围 for 循环隐藏了迭代细节，代码意图更清晰。

在 Qt 容器中的应用：

Qt 的容器类（如 QList、QVector、QMap 等）都支持范围 for 循环。

```cpp
QMap<QString, int> map = {{"Alice", 25}, {"Bob", 30}};
for (const auto& key : map.keys()) {
    qDebug() << key << ":" << map.value(key);
}

// 直接遍历键值对（C++17 结构化绑定更佳）
for (const auto& [key, value] : map.asKeyValueRange()) {
    qDebug() << key << ":" << value;
}
```

注意：Qt 5 中 QMap 没有直接的范围迭代支持键值对，Qt 6 提供了 asKeyValueRange()；在 C++17 中也可使用结构化绑定简化代码。



### 3. 智能指针：告别手动 delete

核心作用：自动管理内存，避免内存泄漏。

##### 3.1 std::unique_ptr：独占所有权

`unique_ptr`是独占所有权的智能指针，不可拷贝，可移动。它非常适合作为工厂函数的返回类型或表示独占资源。
Qt中替代new/delete管理单个控件/对象。

```cpp
#include <memory>
#include <QPushButton>

int main() {
    // 独占式管理QPushButton对象，超出作用域自动释放
    std::unique_ptr<QPushButton> btn(new QPushButton("独占指针"));
    
    // 错误：unique_ptr不可拷贝
    // std::unique_ptr<QPushButton> btn2 = btn;
    
    // 正确：移动语义（转移所有权）
    std::unique_ptr<QPushButton> btn3 = std::move(btn);
    // 此时btn为空，btn3持有指针
    
    return 0;  // btn3析构，自动delete QPushButton对象
}
```

##### 3.2 std::shared_ptr（共享所有权）

* 多个`shared_ptr`可共享同一个指针，通过引用计数管理；
* 引用计数为 0 时，自动释放内存。

```cpp
#include <memory>
#include <QString>

int main() {
    // 创建共享指针
    std::shared_ptr<QString> str = std::make_shared<QString>("共享指针");
    
    // 拷贝：引用计数+1（此时计数=2）
    std::shared_ptr<QString> str2 = str;
    
    // 重置：引用计数-1（此时计数=1）
    str.reset();
    
    // str2析构：引用计数=0，自动释放内存
    
    return 0;
}
```

Qt 对比：`QSharedPointer`与`std::shared_ptr`用法一致，Qt 为了兼容自身生态封装了该类型。

### 4. Lambda 表达式：就地定义匿名函数

Lambda 表达式允许在函数内部定义匿名函数对象，极大地简化了回调、算法定制和短小函数的书写。

核心作用：定义临时的匿名函数，Qt 信号槽中几乎是标配用法。

###### 4.1 基本语法

```cpp
// 完整语法
[捕获列表](参数列表) 修饰符 -> 返回值类型 { 函数体 }

// Qt6信号槽最简化写法（无捕获、无参数、无返回值）
connect(btn, &QPushButton::clicked, []() {
    qDebug() << "按钮被点击";
});
```

###### 4.2 C++20 Lambda 核心新特性（Qt6 适配）

C++20 为 Lambda 带来了极大简化和功能增强，Qt6 已完全适配这些特性，下面逐一讲解关键特性及 Qt 场景用法。

###### 4.2.1 省略括号（无参数 Lambda）

特性：无参数的 Lambda 可省略()，代码更简洁（Qt 信号槽无参场景首选）。

```cpp
#include <QApplication>
#include <QPushButton>
#include <QDebug>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    
    QPushButton btn("点击我（C++20简化）");
    btn.show();
    
    // C++11/17写法（需括号）
    connect(&btn, &QPushButton::clicked, []() {
        qDebug() << "C++17写法";
    });
    
    // C++20写法（省略括号，更简洁）
    connect(&btn, &QPushButton::clicked, [] {
        qDebug() << "C++20写法：省略无参括号";
        btn.setText("已点击");
    });
    
    return a.exec();
}
```

###### 4.2.2 模板 Lambda（泛型 Lambda 增强）

特性：C++20 支持显式模板 Lambda（[]<typename T>(T x) { ... }），替代 C++14 的自动类型推导，类型更可控，适合 Qt 容器通用操作。

```cpp
#include <QVector>
#include <QDebug>

int main() {
    QVector<int> intVec = {1,2,3};
    QVector<QString> strVec = {"Qt6", "C++20", "Lambda"};
    
    // C++20模板Lambda：显式指定模板参数
    auto printElem = []<typename T>(const T& elem) {
        qDebug() << elem;
    };
    
    // 遍历Qt容器（通用写法，适配任意类型）
    for (const auto& elem : intVec) printElem(elem);
    for (const auto& elem : strVec) printElem(elem);
    
    // 结合Qt信号槽：泛型处理不同类型参数
    QPushButton btn;
    connect(&btn, &QPushButton::clicked, []<typename T>(T /*checked*/) {
        qDebug() << "泛型处理点击信号";
    });
    
    return 0;
}
```

###### 4.2.3 constexpr Lambda（编译期 Lambda）

特性：Lambda 可标记为constexpr，支持编译期执行，提升 Qt 程序运行效率（如编译期计算常量、校验参数）。

```cpp
#include <QDebug>

// C++20 constexpr Lambda：编译期执行
constexpr auto calcSize = [] (int base) constexpr {
    return base * 2; // 编译期计算，无运行时开销
};

int main() {
    // 编译期确定Qt控件大小（常量表达式）
    constexpr int btnWidth = calcSize(100);
    constexpr int btnHeight = calcSize(50);
    
    QPushButton btn;
    btn.resize(btnWidth, btnHeight); // 100*2=200, 50*2=100
    qDebug() << "按钮大小：" << btn.size();
    
    return 0;
}
```

###### 4.2.4 捕获 this 的方式增强（Qt 类成员中使用）

特性：C++20 支持[this]的简化写法，且新增`[*this]`（捕获当前对象的拷贝），解决 Qt 类中 Lambda 捕获this的生命周期问题。

```cpp
#include <QObject>
#include <QPushButton>
#include <QDebug>

class MyWidget : public QObject {
    Q_OBJECT
public:
    MyWidget(QObject* parent = nullptr) : QObject(parent) {
        btn = new QPushButton("测试this捕获");
        
        // 1. C++11/17：捕获this（引用当前对象）
        connect(btn, &QPushButton::clicked, [this] {
            this->onBtnClicked(); // 调用类成员函数
        });
        
        // 2. C++20：[*this] 捕获对象拷贝（避免悬垂指针）
        // 适用场景：Lambda可能在对象销毁后执行（如异步操作）
        connect(btn, &QPushButton::clicked, [*this] {
            qDebug() << "捕获对象拷贝：" << this->objectName();
        });
    }
    
    void onBtnClicked() {
        qDebug() << "按钮点击，调用成员函数";
    }

private:
    QPushButton* btn = nullptr;
};
```

###### 4.2.5 Lambda 作为非类型模板参数

特性：C++20 允许 Lambda 作为模板参数，可用于 Qt 自定义模板类 / 函数的通用回调。

```cpp
#include <QDebug>

// 模板函数：接收Lambda作为非类型参数（C++20新特性）
template <auto Func>
void executeQtCallback() {
    Func(); // 执行Lambda
}

int main() {
    // Lambda作为模板参数，实现通用回调
    executeQtCallback<[] {
        qDebug() << "Lambda作为Qt模板参数（C++20）";
    }>();
    
    return 0;
}
```

###### 4.3 Qt6 + C++20 Lambda 实战场景

###### 4.3.1 Qt6 信号槽（带参数 + 简化写法）

```cpp
#include <QApplication>
#include <QSlider>
#include <QLabel>
#include <QVBoxLayout>
#include <QWidget>

int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    
    QWidget w;
    auto layout = new QVBoxLayout(&w);
    auto slider = new QSlider(Qt::Horizontal);
    auto label = new QLabel("值：0");
    layout->addWidget(slider);
    layout->addWidget(label);
    
    // C++20 Lambda：省略括号（无参）/ 自动推导参数类型
    connect(slider, &QSlider::valueChanged, [&label](int val) {
        label->setText(QString("值：%1").arg(val));
    });
    
    w.show();
    return a.exec();
}
```

###### 4.3.2 Qt 容器算法（C++20 constexpr Lambda）

```cpp
#include <QVector>
#include <algorithm>
#include <QDebug>

int main() {
    QVector<int> vec = {5, 2, 8, 1, 9};
    
    // C++20 constexpr Lambda：排序规则
    constexpr auto compare = [] (int a, int b) constexpr {
        return a < b; // 升序排序
    };
    
    // 编译期确定排序逻辑，运行时执行
    std::sort(vec.begin(), vec.end(), compare);
    
    qDebug() << "排序后：" << vec; // 输出：1,2,5,8,9
    
    return 0;
}
```

###### 4.3.3 Qt 异步操作（Qt6 QFuture + C++20 Lambda）

```cpp
#include <QCoreApplication>
#include <QFuture>
#include <QtConcurrent>
#include <QDebug>

int main(int argc, char *argv[]) {
    QCoreApplication a(argc, argv);
    
    // C++20 Lambda作为异步任务（QtConcurrent）
    auto future = QtConcurrent::run([] {
        // 模拟耗时操作
        for (int i = 0; i < 5; ++i) {
            qDebug() << "异步任务：" << i;
            QThread::msleep(500);
        }
        return 42; // 返回值
    });
    
    // 等待任务完成并获取结果
    future.waitForFinished();
    qDebug() << "异步任务结果：" << future.result();
    
    return a.exec();
}
```

###### 注意事项

* 生命周期：在 Qt 信号槽中捕获`this`时，若 Lambda 可能在对象销毁后执行（如异步操作），优先使用`[*this]`（C++20）或QPointer避免悬垂指针；
* 性能：constexpr Lambda仅在编译期可计算场景使用，运行时 Lambda 无需强制标记constexpr；
* 兼容性：若需兼容旧版本 Qt/C++，可降级使用 C++17 写法（仅省略 C++20 特有特性即可）。



#### 总结

1. `auto/decltype`：简化类型声明，Qt 中常用于控件指针、容器变量的定义，decltype 用于查询表达式类型；
2. `范围 for 循环`：替代 C 风格循环，Qt 容器遍历更简洁，优先使用auto&（修改）或const auto&（只读）；
3. `智能指针`：unique_ptr（独占）/shared_ptr（共享）替代裸指针，避免内存泄漏，Qt 的 QSharedPointer 与 std::shared_ptr 原理一致；
4. `Lambda 表达式`：Qt 信号槽核心用法，通过捕获列表控制外部变量访问，简化回调函数编写。

这些特性是 C++20 的基础，也是 Qt 开发的必备技能，掌握后能大幅提升代码质量和开发效率。
