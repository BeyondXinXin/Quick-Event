# Quick Event

#### 介绍
基于QT-5.6封装的一套控制与界面完全分离的代码；利用QT的元对象属性，实现控制类的自动实例化；VS-2015编译通过，使用了C++11语法；使用该模型可以更加专注与自己模块或功能点的开发，更便于调试和扩展；完全分离的控制和界面，让重构变的非常轻松；

#### 软件架构
1.  界面与控制逻辑完全分离
2.  事件管理
3.  线程管理
4.  控制类装填


#### 使用说明

######  发布和订阅
1.通过继承自QuickWork或引入QUICK_EVENT(parent class name)宏来让你的自己定义的类具有发布和订阅事件的能力；

```
//1.继承自QuickWork
class UserWork : public QuickWork
{
    Q_OBJECT
public:
    Q_INVOKABLE explicit UserWork(QObject *parent = nullptr);
    ...
};
//2.使用QUICK_EVENT宏
class Dialog : public QDialog
{
    Q_OBJECT
    QUICK_EVENT(QDialog)
public:
    explicit Dialog(QWidget *parent = nullptr);
    ...
};
```
注:通过QUICK_EVENT宏的方式类也必须时QObject的子类

2.订阅消息
QuickApplication提供了subscibeEvent方法用来订阅一个消息

```
//订阅show_time消息
Dialog::Dialog(QWidget *parent) :
    QDialog(parent),
    ui(new Ui::Dialog)
{
    ui->setupUi(this);
    QuickApplication::subscibeEvent(this, "show_time");
    ...
}
```

3.发布消息
QuickApplication提供了postEvent方法用来发布一个消息

```
TestWork::TestWork(QObject *parent) : QuickWork(parent)
{
    QTimer::singleShot(2000, this, [this]() {
        auto time = QDateTime::currentDateTime().toString();
        QuickApplication::postEvent(nullptr, time, "show_time");
        ...
    });
}
```
4.接受并处理消息
引入发布和订阅功能的类通过实现event_ + 消息名称的槽函数即可接受并处理postEvent推送消息

```
public slots:
    void event_show_time(QSharedPointer<QVariant> ptr);

...

void Dialog::event_show_time(QSharedPointer<QVariant> ptr)
{
    box->setText(ptr->value<QString>());
    box->show();
}

```
######  控制类装填
1. QuickWork及其子类允许被反射，通过QUICK_AUTO(class name)宏和QuickController可以实现控制类的自动实力化

QUICK_AUTO(class name)作用

(1).向QT元对象系统注册自己类型

(2).在mian方法之前将自己类名注册到需要反射类列表中，QuickController对象创建后会反射出所有已注册的类，并做线程归属处理

原理：C/C++无法在main之前执行复杂的操作，通过在.h文件中定义static变量且通过函数方式赋值，可以main之前执行一段代码
，利用static在类外修饰变量表示该变量仅对于文件内部可见的原理，不会产生编译错误；[当然对C++11支持良好编译器可以使用constexpr]
但是这个操作可能被执行多次，所以反射类列表使用了Set容器防止重复插入；

######  线程管理
1. QObject及其子类具有线程归属，QuickWork及其子类可以通过设置moveType_来决定被反射后移动大到线程的位置；

```
    typedef enum {
        MainThread = 0, //反射在主线程中且不移动，start函数不能为死循环否者QT事件循环也将被卡死；
        WorkThread,     //反射在工作线程中，区别与主线程；
        NewThread       //反射在新的线程中，会被每个反射出对象创建一个新的线程；
    } MoveThreadType;
```
注:继承自QuickWork后可以覆盖QuickWork::start函数，start函数一定会在被移动到的线程中调用；

#### 参与贡献

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request


#### 码云特技

1.  使用 Readme\_XXX.md 来支持不同的语言，例如 Readme\_en.md, Readme\_zh.md
2.  码云官方博客 [blog.gitee.com](https://blog.gitee.com)
3.  你可以 [https://gitee.com/explore](https://gitee.com/explore) 这个地址来了解码云上的优秀开源项目
4.  [GVP](https://gitee.com/gvp) 全称是码云最有价值开源项目，是码云综合评定出的优秀开源项目
5.  码云官方提供的使用手册 [https://gitee.com/help](https://gitee.com/help)
6.  码云封面人物是一档用来展示码云会员风采的栏目 [https://gitee.com/gitee-stars/](https://gitee.com/gitee-stars/)
