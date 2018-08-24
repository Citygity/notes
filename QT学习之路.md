# Qt学习之路

##1.1Qt5架构

### 1.1.1模块架构

Qt将所有功能模块分为3部分：Qt基本模块（Qt essentials），Qt扩展模块（Qt Add-ons）和开发工具（Qt tools）

Qt基本模块定义了适用于所有平台的基础功能，是Qt的核心，所有Qt应用程序都需要使用该模块中提供的功能。基本模块的基础是Qt core模块，其他所有模块都依赖于该模块。

shadow build：将项目源码和编译生成的文件分别存放

其次，我们来分析一下Qt的基本流程。1~3行是头文件包含，这里有两种头文件，第一种是自定义头文件或者本地头文件，用“ ”来进行表示和包含；第二种是系统头文件，这里就是Qt自带的头文件，直接用<>进行表示和包含就可以。第7行是创建一个QApplication的实例，对于 Qt 程序来说，`main()`函数一般以创建 application 对象（GUI 程序是`QApplication`，非 GUI 程序是`QCoreApplication`。`QApplication`实际上是`QCoreApplication`的子类。），这个对象用于管理 Qt 程序的生命周期，开启事件循环。10~11行是核心代码，也就是我们实际添加的用例代码，这里我创建了一个QLabel，利用构造函数对其进行赋值操作，最后调用show方法将其显示出来。最后一行调用exec，开启事件循环（可以理解成一段无限循环）。 

```
int main(int argc, char *argv[])//参数用来接收命令行参数
{
    QApplication a(argc, argv);新建一个QApplication类对象，用于管理应用程序的资源，任何一个Qt Widgets都需要有一个QApplication类对象
    //MainWindow w;
    QDialog w;
    QLabel label(&w);
    label.setText("hello world!");
    w.show();
    return a.exec();
}

```

```
Qt       += core gui//表明这个项目使用的模块，core模块包含了Qt的核心功能，其他所有模块都依赖于这个模块，gui模块提供了窗口系统集成，没时间处理，OpenGL ES集成，2D图形，基本图像，字体和文本功能。所谓模块就是很多类的集合

greaterThan(Qt_MAJOR_VERSION, 4): Qt += widgets//添加widgets模块，提供了经典的桌面用户界面的UI元素集合

TARGET = calender//生成目标文件的名称，就是生成的exe文件名字
TEMPLATE = app//使用app模版，表明这是个应用程序

DEFINES += Qt_DEPRECATED_WARNINGS

Qt 6.0.0


SOURCES += \   //包含的原文件
        main.cpp \
        mainwindow.cpp

HEADERS += \  //包含的头文件
        mainwindow.h

FORMS += \    //包含的界面文件
        mainwindow.ui
RC_ICONS=myico.ico   //应用程序图标
```

QMainWindow是带有菜单栏和工具栏的主窗口类

QDialog是各种对话框的基类

而他们全都继承自QWidget

### 基础窗口类QWidget

Widget窗口部件，简称不见，是Qt中建立用户界面的主要元素Qt中把没有嵌入到其他部件中的部件称为窗口，一般窗口都是边框和标题栏。

QMainWindow和大量的QDialog子类是最一般的窗口类型，窗口就是没有父部件的部件，所以又称为顶级部件，与其相对的是非窗口部件，也叫做子部件。

QWidget继承自QObject类和QPaintDevice类，QObject是所有支持Qt对象模型的对象的基类，QPaintDevice是所有可以绘制的对象的基类。

Qt中销毁父对象的时候会自动销毁子对象

### 对话框QDialog

QDialog类是所有对话框窗口类的基类，对话框窗口是一个经常用来完成短小任务或者和用户进行简单交互的顶层窗口。按照运行对话框时是否还可以和该程序的其他窗口进行交互，对话框常被分为两类：模态的，非模态的

Qt 支持模态对话框和非模态对话框。其中，Qt 有两种级别的模态对话框：应用程序级别的模态和窗口级别的模态，默认是应用程序级别的模态。应用程序级别的模态是指，当该种模态的对话框出现时，用户必须首先对对话框进行交互，直到关闭对话框，然后才能访问程序中其他的窗口。窗口级别的模态是指，该模态仅仅阻塞与对话框关联的窗口，但是依然允许用户与程序中其它窗口交互。窗口级别的模态尤其适用于多窗口模式 ，更详细的讨论可以看[以前发表过的文章](http://www.devbean.net/2011/03/qdialog_window_modal/)。 

模态对话框就是在没有关闭它之前，不能再与同一个应用程序的其他窗口进行交互。setModal()函数可以修改模态

```
#ifndef MAINWINDOW_H   //IF NOT DEFINE MAINWINDOW_H 防止该头文件被重复引用
#define MAINWINDOW_H   //DEFINE MAINWINDOW_H
//函数体
#endif
//
```

信号与槽的关联方式除了connect()函数，还有自动关联：将关联函数整合到槽命名中，比如将槽重命名为字符on,发射信号的部件对象名和信号名组成。这样就可以取消关联函数去掉。对于不是在Qt设计器中往界面上添加的部件，就要在调用setupUI()函数前定义该部件，而且还要使用setObjectName()函数制定部件的对象名，这样才可以使用自动关联，在编写程序时一般都使用第一种connect方式。

### QFrame类族

QFrame是带有边框的部件的基类，Qt中凡是带有Abstract字样的类都是抽象基类，抽象基类是不能直接使用的，但是可以继承该类 实现自己的类，或者使用它提供的子类。

### QLabel

QLabel部件用来显示文本或者图片

### 布局管理器

1.基本布局管理器（QBoxLayout）

QBoxLayout类可以使子部件在水平方向，或者垂直方向排成一列，它将所有的空间分成一行盒子，然后将每个部件在水平方向或者垂直方向排成一列，它将所有的空间分成一行盒子，然后将每个部件放入一个盒子中。他有两个子类QHBoxLayout和QVBoxLayout分别是水平布局管理器和垂直布局管理器。

2.栅格布局管理器（QGridLayout）

栅格布局管理器QGridLayout类使部件在网格中进行布局，它将所有的空间分隔成一些行和列。

....

### 可扩展窗口

### 应用程序主窗口

#### 主窗口框架

1.菜单栏

2.工具栏

3.中心部件

4.dock部件

5.状态栏



### 事件系统

事件（event）是由系统或者 Qt 本身在不同的时刻发出的。 Qt 中的事件和信号槽却并不是可以相互替代的。信号由具体的对象发出，然后会马上交给由`connect()`函数连接的槽进行处理；而对于事件，Qt 使用一个事件队列对所有发出的事件进行维护，当新的事件产生时，会被追加到事件队列的尾部。前一个事件完成后，取出后面的事件进行处理。但是，必要的时候，Qt 的事件也可以不进入事件队列，而是直接处理。 

总的来说，如果我们**使用**组件，我们关心的是信号槽；如果我们**自定义**组件，我们关心的是事件。因为我们可以通过事件来改变组件的默认操作。比如，如果我们要自定义一个能够响应鼠标事件的`EventLabel`，我们就需要重写`QLabel`的鼠标事件，做出我们希望的操作，有可能还得在恰当的时候发出一个类似按钮的`clicked()`信号（如果我们希望让这个`EventLabel`能够被其它组件使用）或者其它的信号。 

当事件发生时，Qt 将创建一个事件对象。Qt 中所有事件类都继承于`QEvent`。在事件对象创建完毕后，Qt 将这个事件对象传递给`QObject`的`event()`函数。`event()`函数并不直接处理事件，而是按照事件对象的类型分派给特定的事件处理函数（event handler）。 

**当重写事件回调函数时，时刻注意是否需要通过调用父类的同名函数来确保原有实现仍能进行！** 

事件过滤器与事件的发送：要对一个部件使用事件过滤器，那么就要先使用其的installEventFilter()函数为其安装事件过滤器，这个函数表明了监视对象。

### 图形视图框架的结构

图形视图框架提供了一个基于图形项的模型视图编程方法，主要由场景，视图和图形项三部分组成，分别由QGraphicsScene,QGraphView,QGraphicsItem,这三个类来表示。多个视图可以查看一个场景，场景中包含各种各样几何形状的图形项。	

GVF可以管理数量庞大的自定义2D图形项，并且进行交互。使用视图部件可以使这些图形项可视化，视图还支持缩放和旋转。

#### QGraphicsScene 场景

场景是图形项QGraphicsItem对象的容器，拥有以下功能：

- 提供用于管理大量图形项的高速接口
- 传播事件到每一个图形项
- 管理图形项的状态，比如选择和处理焦点
- 提供无变换的渲染功能，主要用于打印

#### QGraphView 视图

视图部件用来使场景中的内容可视化，可以连接多个视图到同一场景来为相同的数据集提供多个视口。

#### QGraphicsItem 图形项

- 鼠标按下，移动，释放，双击，悬停，滚轮和右键菜单事件
- 键盘输入焦点和键盘事件
- 拖放事件
- 分组，使用QGraphicsItemGroup通过parent-child关系来实现
- 碰撞检测

### 图形视图框架的坐标系统和事件处理

GVF有3个有效的坐标系统：图形项坐标，场景坐标和视图坐标。绘图时，场景坐标对应QPainter的逻辑坐标，视图坐标对应设备坐标。

#### 图形项坐标

图形项使用自己的本地坐标系统，坐标通常以他们的中心为原点(0,0)，而这也是所有变换的中心，自定义图形项的时，只需要考虑图形项的坐标系统，QGraphicsView，QGraphicsScene会完成其他所有的变换，而且一个图形项的边界矩形和图形形状都是在图形项坐标系统中的。

图形项的位置是图形项在父形项或者场景中的位置，

#### 场景坐标

场景坐标是所有图形项的基础坐标系统。场景坐标系统描述了每一个顶层图形项的位置，也用于处理所有从视图传到场景上的事件。场景坐标的原点在场景的中心，x和y坐标分别向右和向下增大。

#### 视图坐标

视图的坐标就是部件的坐标，视图坐标的每一个单位对应一个像素，原点(0,0)总在QGraphicsView视口的左上角，而右下角是(宽，高)，所有的鼠标事件和拖放事件最初都是使用视图坐标接收的。

### 界面外观

Qt中的各种风格是继承至QStyle的一组类，QStyle类是一个抽象基类，封装了一个GUI的外观

调色板QPalette类包含了部件各种状态的颜色组。调色板包含3中状态：激活，非激活，失效

**激活** QPalette::Active,用于获得键盘焦点的窗口

**非激活** QPalette::Inactive，用于其他没有获得键盘焦点的窗口

**失效** QPalette::Disabled，用于因为一些原因而不可用的部件（不是窗口）

| 常量                    | 描述                                               |
| ----------------------- | -------------------------------------------------- |
| QPalette::WindowText    | 一个一般的前景颜色                                 |
| QPalette::Base          | 可作QLinrEdit QComboBox的背景色 QToolBar的手柄颜色 |
| QPalette::AlternateBase | 在交替行颜色的视图中作为交替背景色                 |
| QPalette::ToolTipBase   | QToolTip QWhatsThis背景色                          |
| QPalette::ToolTipText   | QToolTip QWhatsThis前景色                          |
| QPalette::Text          | 与Base一起，作为前景色                             |
| QPalette::Button        | 按钮部件背景色                                     |
| QPalette::ButtonText    | 按钮部件背景色                                     |
| QPalette::Window        | 一个一般的背景颜色                                 |

### Qt样式表

可用以下方式读取qss文件

```
    QFile qss(":/qss.qss");
    qss.open(QFile::ReadOnly);
    w.setStyleSheet(qss.readAll());
    qss.close();
```

qss知识总结：https://blog.csdn.net/yansmile1/article/details/52882965

#### 盒子模型

使用样式表时，每一个部件都被看作拥有4个同心矩形的盒子，默认值均为0，这样4个矩形恰好重合。可以把它当成日常中的一个盒子去理解。content就是盒子里装的东西，它有高度（height）和宽度（width）,可以是图片，可以是文字或者小盒子嵌套，在现实中，内容不能大于盒子，内容大于盒子就会撑破盒子，但在css中，盒子有弹性的，顶多内容太大就会撑大盒子，但是不会损害盒子。padding即是填充，就好像我们为了保证盒子里的东西不损坏，填充了一些东西，比如泡沫或者塑料薄膜，填充物有大有小，有软有硬，反应在网页中就是padding的大小了。而再外一层就是border边框，因为边框有大小和颜色的属性，相当于盒子的厚度和它的颜色或者材料。margin外边距，就是我们的盒子与其他的盒子或者其他东西的距离。假如有很多盒子，margin就是盒子堆码直接的距离，可以通风，也美观同时方便取出。 

![img](https://images0.cnblogs.com/blog2015/790006/201507/281730521883310.jpg) 

### 2D画图

Qt中提供了强大的2D绘图系统，可以使用相同的API在屏幕和绘图设备上进行绘制，主要基于QPainter,QPaintDevice,QPaintEngine这3个类，其中QPainter用来执行绘图操作，QPaintDevice提供绘图设备，是一个二维空间的抽象，可以使用QPainter在其上进行绘制；是所有可以进行绘制的对象的基类，子类主要有QWidget,QPixmap,QPicture,QImage等。QPaintEngine提供了一些接口，用于QPainter和QPaintDevice内部，使得QPainter可以在不同的设备上进行绘制；除了创建自定义的绘图设备类型，一般编程中不需要使用该类。

QPainte->QPaintDevice->QPaintEngine(关系图)

### model/view编程

应用程序中往往需要存储大量的数据，并对它们进行处理，然后可以通过各种形式显示给用户，用户需要时还可以对数据进行编辑，Qt中的Model/View架构用来实现大量数据的存储、处理及其显示的。MVC结构如下图。![img](https://images2015.cnblogs.com/blog/811883/201704/811883-20170423150019101-1710764799.jpg) 

 将view和controller结合起来，就形成了Model/view架构，Model/View框架的核心思想是模型（数据）与视图（显示）相分离，模型对外提供标准接口存取数据，不关心数据如何显示，视图自定义数据的显示方式，不关心数据如何组织存储。 引入委托(delegate,也叫代理)的概念，使用它可以定制数据的渲染和编辑方式。

![wKioL1hCXQKDvgm7AABchnskI7U624.png](http://s4.51cto.com/wyfs02/M00/8B/05/wKioL1hCXQKDvgm7AABchnskI7U624.png) 

#### 组成部分

model/view架构中众多类可分为3组：模型，视图和委托。



QToolbar+ qtoolbutton

Qtreewidget换为qlistwidget