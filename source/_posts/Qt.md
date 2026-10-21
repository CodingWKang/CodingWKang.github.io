---
title: Qt学习指南
date: 2026-10-21 20:09:34
tags:
- Qt
category:
- GUI
index_img: /imgs/Qt.png
excerpt: 摘要：Qt是用C++编写的跨平台应用开发框架，提供商业、GPL及LGPL三种许可。其GUI开发支持Qt Designer可视化设计与纯代码两种方式，核心机制为信号与槽和基于Q_OBJECT宏、Q_PROPERTY的元对象系统。项目通过qmake或CMake构建，经MOC、UIC、RCC预编译器将元对象、UI及资源文件转为标准C++后编译链接。框架模块分为Qt Essentials基础模块与Qt Addons附加模块，可广泛用于PC、服务器、移动设备及嵌入式开发。
---

> Qt是一个用C++语言编写的GUI跨平台类库，使用Qt可以为PC、服务器、移动设备甚至单片机开发各种应用。

## 1. Qt简介

Qt的实质上是用C++编写的大型类库，能够为跨平台应用开发提供完整的框架。Qt框架包含大量的类，支持GUI、数据库、网络、多媒体等各种应用的编程。

### 1.1 Qt的许可类型

1.  商业许可。Qt的商业许可需要付费，一版是采取按年付费的方式，商业许可运行开发者不公开项目的源码。
2.  开源许可。GPLv2/GPLv3许可，若用户编程的程序中使用了GPL许可，则用户程序也必须使用GPL许可，即用户代码也必须开源，但是允许商业化销售。LGPLv3许可，如果是用户对使用LGPL许可的Qt代码进行修改后予以发布，则用户发布的代码必须遵循LGPL许可，即必须开源且允许商业化销售。但是若是用户编写的程序仅仅只是以库的形式链接或者调用了LGPL许可的Qt代码，则用户代码可以闭源，也可以商业化销售。

### 1.2 第一个Qt程序

*   main.cpp

```c++
#include "widget.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    Widget w;
    w.show();
    return a.exec();
}
```

*   widget.h

```c++
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

QT_BEGIN_NAMESPACE
namespace Ui {
class Widget;
}
QT_END_NAMESPACE

class Widget : public QWidget
{
    Q_OBJECT

public:
    Widget(QWidget *parent = nullptr);
    ~Widget();

private:
    Ui::Widget *ui;
};
#endif // WIDGET_H
```

*   widget.cpp

```c++
#include "widget.h"
#include "ui_widget.h"

Widget::Widget(QWidget *parent)
    : QWidget(parent)
    , ui(new Ui::Widget)
{
    ui->setupUi(this);
}

Widget::~Widget()
{
    delete ui;
}
```

![CLFEGQDD](/imgs/Qt/2106185-20261015090807906-1981356975.png)

## 2. GUI设计基础

### 2.1 GUI程序结构与运行机制

#### 2.1.1 GUI文件组成

![KD64U4NZ](/imgs/Qt/2106185-20261015090826737-1079961538.png)

新建好一个Widget项目后，该项目包含以下一些文件：

*   Hello.pro是qmake构建系统的项目配置文件，其中存储了项目的各种设置内容。
*   widget.ui是UI文件，这个文件主要是用于窗口界面可视化设计文件。
*   main.cpp是主程序文件，包含main()函数。
*   widget.h是窗口类定义头文件，这个文件用到了widget.ui中的可视化设计界面。
*   widget.cpp是对应于widget.h的源程序文件。

#### 2.1.2 项目配置文件

上文中的Hello.pro就是使用向导创建的Hello项目的项目配置文件，内容如下，具体内容可以参考官方的QMake手册：

```c++
QT       += core gui

greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

CONFIG += c++17

# You can make your code fail to compile if it uses deprecated APIs.
# In order to do so, uncomment the following line.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    
# disables all the APIs deprecated before Qt 6.0.0

SOURCES += \
    main.cpp \
    widget.cpp

HEADERS += \
    widget.h

FORMS += \
    widget.ui

# Default rules for deployment.
qnx: target.path = /tmp/$${TARGET}/bin
else: unix:!android: target.path = /opt/$${TARGET}/bin
!isEmpty(target.path): INSTALLS += target
```

qmake是构建项目的工具软件，qmake的作用是根据项目配置文件中的设置生成Makefile文件，然后C++编译器就可以根据Makefile文件进行编译和链接，对于Qt项目，qmake还会自动为元对象编译器(MOC)和用户界面编译器(UIC)生成构建规则。 qt项目的配置文件是自动生成的，其常见变量的含义如下： ![TN4QUMSS](/imgs/Qt/2106185-20261015090912170-1251881946.png)

#### 2.1.3 窗口文件

Qt项目进行构建后，项目的根目录会生成一个ui\_widget.h文件，与窗口相关的文件主要有4个，如下图所示：![3H3SF8FY](/imgs/Qt/2106185-20261015090856160-181577746.png)

1.  文件widget.h

文件widget.h是定义窗口类的头文件，在创建项目时，选择的窗口基类是QWidget，文件widget.h中定义了一个继承自QWidget的类Widget，如下所示：

```c++
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

QT_BEGIN_NAMESPACE
//一个命名空间UI，包含了一个类Widget
namespace Ui {
class Widget;
}
QT_END_NAMESPACE

class Widget : public QWidget
{
    Q_OBJECT

public:
    // 构造函数
    Widget(QWidget *parent = nullptr);
    // 析构函数
    ~Widget();

private:
    // 使用UI::Widget类定义的一个对象指针
    Ui::Widget *ui;
};
#endif // WIDGET_H
```

Ui命名空间里的类Widget并不是本文件代码里那个Widget，而是ui\_widget.h里面定义的一个类，是用于描述可视化设计的窗口界面的，所以也称这种Ui::Widget类为窗口UI类。

窗口类Widget中存在一个继承自QWidget的类Widget，包含创建窗口界面，实现窗口上界面组件的交互操作以及一些其他的业务逻辑。Widget类定义的第一行语句插入了一个宏Q\_OBJECT，这个是使用Qt元对象系统的类时必须插入的一个宏。只有插入了这个宏之后，Widget类才可以使用信号与槽、属性等功能。private部分定义了一个指针，这个指针就是使用Ui命名空间里的类Widget定义的，所以ui是窗口UI类的对象指针，指向可视化设计的窗口界面。

2.  widget.cpp

```c++
#include "widget.h"
#include "ui_widget.h"

Widget::Widget(QWidget *parent)
   : QWidget(parent)
   , ui(new Ui::Widget)
{
   ui->setupUi(this);
}

Widget::~Widget()
{
   delete ui;
}
```

源代码开头引入了一个头文件包含语句，文件ui\_widget.h是UI文件widget.ui呗UIC编译后生成的文件，这个文件里定义了窗口UI类。

3.  widget.ui

本文件将是窗口界面的定义文件，是一个XML文件，存储了界面上所以组件的属性设置、布局、信号与槽函数的关联内容。

4.  ui\_widget.h

在构建项目时，UI文件widget.ui会被Qt的UIC编译为对应的C++文件ui\_widget.h。这个文件不会出现在Qt的项目管理树种，因为它只是作为构建项目时的一个中间文件。

```c++
/********************************************************************************
** Form generated from reading UI file 'widget.ui'
**
** Created by: Qt User Interface Compiler version 6.10.2
**
** WARNING! All changes made in this file will be lost when recompiling UI file!
********************************************************************************/

#ifndef UI_WIDGET_H
#define UI_WIDGET_H

#include <QtCore/QVariant>
#include <QtWidgets/QApplication>
#include <QtWidgets/QLabel>
#include <QtWidgets/QWidget>

QT_BEGIN_NAMESPACE

class Ui_Widget
{
public:
   QLabel *label;

   void setupUi(QWidget *Widget)
   {
       if (Widget->objectName().isEmpty())
           Widget->setObjectName("Widget");
       Widget->resize(258, 118);
       label = new QLabel(Widget);
       label->setObjectName("label");
       label->setGeometry(QRect(90, 40, 91, 31));
       QFont font;
       font.setFamilies({QString::fromUtf8("JetBrains Mono")});
       font.setPointSize(16);
       label->setFont(font);

       retranslateUi(Widget);

       QMetaObject::connectSlotsByName(Widget);
   } // setupUi

   void retranslateUi(QWidget *Widget)
   {
       Widget->setWindowTitle(QCoreApplication::translate("Widget", "Widget", nullptr));
       label->setText(QCoreApplication::translate("Widget", "Hello", nullptr));
   } // retranslateUi

};

namespace Ui {
   class Widget: public Ui_Widget {};
} // namespace Ui

QT_END_NAMESPACE

#endif // UI_WIDGET_H
```

5.  窗口的创建与初始化

widget.h里的Widget类，定义了一个Ui::Widget类型的指针变量ui,通过这个指针就能够访问界面上的所有组件。在widget.cpp中构造函数创建了Ui::Widget的类对象ui,运行ui->setupUi(this)进行界面的初始化，后续在Widget类中编写程序时，就可以通过ui指针访问窗口界面上的所有组件。

### 2.2 可视化设计

#### 2.2.1 创建项目和资源文件

在Qt项目中，图标存储在资源文件里，通过新建文件，选择新建Qt资源文件，设置资源文件的文件名，并将其添加到当前项目里。资源文件主要功能是存储图标和图片文件，方便在程序中使用，在创建资源文件里首先需要创建一个前缀，前缀表示资源的分组。

#### 2.2.2 窗口界面设计

双击.ui文件就可打开Qt Designer对窗口界面进行可视化设计。下面有一个.ui文件的主要组件的说明： ![4THXULMM](/imgs/Qt/2106185-20261015091003955-396489687.png)
值得说明的是这里面的界面组件的属性设置：

1.  对象名称就是指窗口上的组件的实例名称，界面上的每个组件都需要有一个唯一的对象名称。
2.  窗口的对象名称会影响窗口UI类的名称。
3.  设置窗体的font属性后，界面上其他组件的默认字体就是窗体的字体。
4.  组件的属性都会具有一些默认值，一个组件的某个属性被修改后，属性编辑器里面对应的属性名称会加粗显示。

#### 2.2.3 布局管理

Qt的UI设计具有布局功能，所谓的布局就是指界面上组件的排列方式，使用布局可以使组件有规则地分布，并且使其随着窗口大小变化自动调整大小和相对位置。

1.  界面组件的层次关系

为了将界面上的各个组件的分布设计得更加美观，常会使用一些容器类组件。

2.  布局管理

Qt为UI设计提供了丰富的布局管理功能。在Qt Designer里面，组件面板里有Layouts和Spacers两个分组。常用组件如下：

![ZQPLT5MS](/imgs/Qt/2106185-20261015091022569-636087821.png)

Qt Designer有一个工具栏，用于使得界面进入不同的设计状态，或进行布局设计，工具栏按钮功能如下：

![MURTHEWW](/imgs/Qt/2106185-20261015091057053-1646772154.png)

![L6EET9VV](/imgs/Qt/2106185-20261015091110855-574308017.png)

使用工具栏上的布局按钮时，只需要在窗体上选中所需设计布局的组件，然后点击某个布局按钮，使用Ctrl键可进行多选，并且选择了某个容器组件相当于选择了其内部的所有组件。

3.  伙伴关系与Tab顺序

点击Qt Designer工具栏上的Edit Buddies按钮可以进入伙伴关系编辑状态。伙伴关系是指界面上一个标签和一个具有输入焦点的组件相关联。利用伙伴关系可以在程序运行时通过快捷键将输入焦点切换到某个组件上。点击Qt Designer工具栏上的Edit Tab Order按钮进入Tab顺序编辑状态，Tab顺序是指在程序运行时，按下键盘上的Tab键时输入焦点的移动顺序。

#### 2.2.4 信号与槽简介

信号与槽是Qt编程的基础，也是Qt的一大创新，有了信号与槽的编程机制，在Qt中处理界面上各个组件的交互操作就变得比较直观和简单。 信号就是在特定情况下被发射的通知。如：QPushButton比较常见的信号就行点击鼠标发射的clicked()信号，GUI程序设计的主要工作就是对界面上个组件的信号进行响应，只需要知道什么情况下发射哪些信号，合理地去响应和处理信号就可以了。

槽是对信号进行响应的函数。槽就是函数，所以也称为槽函数，槽函数与一般的C++函数一样，可以具有任何参数，也可以被直接调用。槽函数可以与信号关联，当信号被发射时，关联的槽函数就会自动运行。 信号与槽关联是用函数QObject::connect()实现的，基本格式如下：

```c++
QObject::connect(sender,SIGNAL(signal()),receiver,SLOT(slot()));
```

connect()是QObject类的一个静态函数，而QObject()是大部分Qt类的基类，在实际调用时可以省略QObject::,其中sender是发射信号的对象的名称，signal()是信号，信号可以看作是特殊的函数，需要带有括号，有参数时还需要指明各参数类型，receiver是接收信号的对象的名称，slot()是槽函数，需要带有括号，有参数时还需要指明各参数类型。 SIGNAL和SLOT是Qt的宏，分别用于指明信号和槽函数，并将它们的参数转换为相应的字符串。对于信号与槽：

1.  一个信号可以连接多个槽函数，如：

```c++
connect(spinNum,SIGNAL(valueChanged(int)),this,SLOT(addFun(int));
connect(spinNum,SIGNAL(valueChanged(int)),this,SLOT(updateStatus(int));
```

当一个信号与多个槽函数关联时，槽函数按照建立连接时的顺序依次进行。当信号与槽函数有参数时，在connect()里要指明各参数类型，但是不用指明参数名称。

2. 多个信号可以连接到同一个槽函数。这里三个按钮的clicked()信号关联到相同的自定义槽函数do\_setFontColor()。

```c++
connect(ui->radioBlack,SIGNAL(clicked()),this,SLOT(do_setFontColor());
connect(ui->radioRed,SIGNAL(clicked()),this,SLOT(do_setFontColor());
connect(ui->radioBlue,SIGNAL(clicked()),this,SLOT(do_setFontColor());
```

3.  一个信号可以连接到另一个信号。

```c++
connect(spinNum,SIGNAL(valueChanged(int)),this,SLOT(refreshInfo(int));
```

这样当发射一个信号时，也会大蛇另外一个信号，以实现某些特殊的功能。 

4. 严格情况下，信号与槽的参数个数与类型需要一致，至少信号的参数不能少于槽的参数，参数如果不匹配，会出现编译错误或运行错误。 

5. 在使用信号与槽的类中，必须在类的定义中插入宏Q\_OBJECT。

6. 当一个信号被发射时，与其关联的槽函数通常被立即执行，只有当信号关联的所有槽函数运行完毕后，才运行发射信号处后面的代码。

函数connect()有多种参数形式，有一种常用的形式是不使用SIGNAL和SLOT宏，而是使用函数指针。如：

```c++
connect(btnClose,&QPushButton::clicked,Widget,qOverload<>(&QWidget::close));
```

#### 2.2.5 信号与槽的使用

1.  信号与槽

编辑器的使用Qt的界面组件都是从QWidget继承而来，都支持信号与槽功能，每个类都有一些内置的信号和槽函数。可以在Action编辑器里设置组件的内置信号与其他组件的公有槽函数关联。

2.  为组件信号生成槽函数原型与框架

在窗体上通过右键某个组件选择菜单项Go to slot，会出现对话框，其中会显示当前组件所有可用的信号。在对话框中选择所需信号，点击OK后在对应的类的private slot部分会自动增加函数声明，函数名是根据发射信号的对象名和信号名自动命名的。同时在对应的源码中会自动生成这个函数的代码框架。命名规则就是：

```c++
void on_<对象名>_<信号名>(<信号参数>)
```

3.  自定义槽函数

合适的设置自定义槽函数可用减少公用槽函数的使用次数，比如有三个单选按钮设置文字颜色，一次只有一个单选按钮被选中，设置一个自定义槽函数，可以让三个按钮的clicked()信号都与自定义槽函数进行关联。

```c++
void do_setFontColor();  // 设置文字颜色的自定义槽函数
```

右键该函数，在弹出的快捷菜单中选择Refactor->Add Definition in dialog.cpp,就可以在文件dialog.cpp中自动生成该函数的代码框架。

```c++
void Dialog::do_setFontColor(){
  QPalette plet=ui->plainTextEdit->palette();
  if(ui->radioBlue->isChecked())
    plet.setColor(QPalette::Text,Qt::blue);
  else if(ui->radioRed->isChecked())
    plet.setColor(QPalette::Text,Qt::red);
  else if(ui->radioBlack->isChecked())
    plet.setColor(QPalette::Text,Qt::black);
  else
    plet.setColor(QPalette::Text,Qt::black);
  ui->plainTextEdit->setPalette(plet);
}
```

由于这个槽函数是自定义的，因此不会与界面上三个单选按钮的clicked()信号自动关联，所以需要在构造函数中手动进行关联：

```c++
Dialog::Dialog(QWidget *parent): QDialog(parent),ui(new Ui::Dialog)
{
ui->setupUi(this);
connect(ui->radioBlack,SIGNAL(clicked()),this,SLOT(do_setFontColor()));
connect(ui->radioRed,SIGNAL(clicked()),this,SLOT(do_setFontColor()));
connect(ui->radioBlue,SIGNAL(clicked()),this,SLOT(do_setFontColor()));
}
```

#### 2.2.6 为应用程序设置图标

项目被构建后生成的可执行程序文件具有默认的图标，如果需要为应用程序设置图标。

1.  将一个后缀为".ico"的图标文件复制到项目根目录下。
2.  在项目配置文件（.pro）里用RC\_ICONS设置图标文件名重新构建项目，生成的可执行文件以及窗口的图标就会换成设置的图标。

#### 2.2.7 Qt构建过程

一个qmake构建系统的Qt项目，除了有项目配置文件，还有用Qt C++编写的头文件和源程序文件，以及窗口UI文件和资源文件。在构建项目的过程中，这3类文件会被分别编译为标准C++程序文件，然后被标准C++编译器编译为可执行文件或库。Qt项目的构建过程可以用以下的基本过程来描述。

1.  元对象系统和MOC

Qt对C++标准进行了扩展，引入了元对象系统(MOS),所有从QObject继承的类都可以利用元对象系统提供的功能。元对象系统支持属性、信号与槽、动态类型转换等。

![QYYGSWAW](/imgs/Qt/2106185-20261015091138148-963775375.png)

2.  UI文件和UIC

在构建项目时，可视化设计的窗口UI文件会被用户界面编译器(UIC)转换为一个C++源程序文件。如widget.ui被转换为ui\_widget.h文件。

3.  资源文件与RCC

Qt项目中的资源文件(.qrc)会被资源编译器(RCC)转换为C++程序文件。如：res.qrc经过RCC编译后生成的文件是qrc\_res.cpp。

4.  标准C++编译器

标准C++编译器就是开发套件中的编译器，使用MOC、UIC和RCC编译各原始文件的过程称为预编译过程，预编译之后生成的是标准C++语言的程序文件，它们被标准C++编译器编译和连接，最终生成可执行文件。

### 2.3 代码化UI设计

在实际GUI设计过程中，某些界面效果的可视化设计是无法实现的，而这部分就需要使用纯代码的方式去创建界面。

1.  dialog.h

```c++
#ifndef DIALOG_H
#define DIALOG_H

#include <QCheckBox>
#include <QDialog>
#include <QPlainTextEdit>
#include <QPushButton>
#include <QRadioButton>
class Dialog : public QDialog
{
    Q_OBJECT

public:
    Dialog(QWidget *parent = nullptr);
    ~Dialog();

private:
    QCheckBox *chkBoxUnder;
    QCheckBox *chkBoxItalic;
    QCheckBox *chkBoxBold;
    QRadioButton *radioBlack;
    QRadioButton *radioRed;
    QRadioButton *radioBlue;
    QPushButton *btnOK;
    QPushButton *btnCancel;
    QPushButton *btnClose;
    QPlainTextEdit *txtEdit;
    void iniUI();
    void iniSingnalSlots();
private slots:
    void do_chkBoxUnder(bool checked);
    void do_chkBoxItalic(bool checked);
    void do_chkBoxBold(bool checked);
    void do_setFontColor();
};
#endif // DIALOG_H
```

Dialog类的private部分声明了界面上的各个组件的指针变量，这些界面组件都需要在Dialog类的构造函数里创建并在窗口上布局。iniUI()函数创建所有界面组件并完成布局和属性设置，函数iniSignalSlots()完成所有的信号与槽的关联。

private slots部分声明了4个自定义槽函数，需要在函数iniSignalSlots()里与其相应的信号连接。三个checkBox的槽函数如下：

```c++
void Dialog::do_chkBoxUnder(bool checked)
{
    QFont font = txtEdit->font();
    font.setUnderline(checked);
    txtEdit->setFont(font);
}

void Dialog::do_chkBoxItalic(bool checked)
{
    QFont font = txtEdit->font();
    font.setItalic(checked);
    txtEdit->setFont(font);
}

void Dialog::do_chkBoxBold(bool checked)
{
    QFont font = txtEdit->font();
    font.setBold(checked);
    txtEdit->setFont(font);
}

void Dialog::do_setFontColor()
{
    QPalette plet = txtEdit->palette();
    if (radioBlue->isChecked())
        plet.setColor(QPalette::Text, Qt::blue);
    else if (radioRed->isChecked())
        plet.setColor(QPalette::Text, Qt::red);
    else if (radioBlack->isChecked())
        plet.setColor(QPalette::Text, Qt::black);
    else
        plet.setColor(QPalette::Text, Qt::black);
    txtEdit->setPalette(plet);
}
```

2.  界面组件的创建与布局

函数iniUI()实现界面组件的创建与布局，下面是函数iniUI()的完整代码:

```c++
void Dialog::iniUI()
{
    chkBoxUnder = new QCheckBox("Underline");
    chkBoxItalic = new QCheckBox("Italic");
    chkBoxBold = new QCheckBox("Bold");
    QHBoxLayout *HLay1 = new QHBoxLayout();
    HLay1->addWidget(chkBoxUnder);
    HLay1->addWidget(chkBoxItalic);
    HLay1->addWidget(chkBoxBold);
    radioBlack = new QRadioButton("Black");
    radioBlack->setChecked(true);
    radioRed = new QRadioButton("Red");
    radioBlue = new QRadioButton("Blue");
    QHBoxLayout *HLay2 = new QHBoxLayout;
    HLay2->addWidget(radioBlack);
    HLay2->addWidget(radioRed);
    HLay2->addWidget(radioBlue);
    btnOK = new QPushButton("确定");
    btnCancel = new QPushButton("取消");
    btnClose = new QPushButton("退出");
    QHBoxLayout *HLay3 = new QHBoxLayout;
    HLay3->addStretch();
    HLay3->addWidget(btnOK);
    HLay3->addWidget(btnCancel);
    HLay3->addStretch();
    HLay3->addWidget(btnClose);
    txtEdit = new QPlainTextEdit();
    txtEdit->setPlainText("Hello world\n手工创建");
    QFont font = txtEdit->font();
    font.setPointSize(20);
    txtEdit->setFont(font);
    QVBoxLayout *VLay = new QVBoxLayout(this);
    VLay->addLayout(HLay1);
    VLay->addLayout(HLay2);
    VLay->addWidget(txtEdit);
    VLay->addLayout(HLay3);
    setLayout(VLay);
}
```

函数iniUI()按顺序实现如下操作：

*   创建3个QCheckBox组件，然后创建1个水平布局HLay1,将3个复选框添加到这个水平布局里
*   创建3个QRadioButton组件和水平布局HLay2，将这3个单选按钮添加到这个水平布局里。
*   创建3个QPushButton组件和水平布局HLay3,将这3个按钮添加到这个水平布局里。
*   创建一个QPlainTextEdit组件，设置其文字内容和字体
*   创建一个垂直布局VLay，将前面创建的3个水平布局和文本框依次添加到此布局里，设置垂直布局VLay为主窗口的布局。

3.  信号与槽的关联

Dialog类中定义的槽函数都是自定义函数，不会和组件的信号自动关联，函数iniSignalSlots()用于建立信号与槽的关联。

```c++
void Dialog::iniSingnalSlots()
{
    connect(radioBlue, SIGNAL(clicked()), this, SLOT(do_setFontColor()));
    connect(radioRed, SIGNAL(clicked()), this, SLOT(do_setFontColor()));
    connect(radioBlack, SIGNAL(clicked()), this, SLOT(do_setFontColor()));
    connect(chkBoxUnder, SIGNAL(clicked(bool)), this, SLOT(do_chkBoxUnder(bool)));
    connect(chkBoxBold, SIGNAL(clicked(bool)), this, SLOT(do_chkBoxBold(bool)));
    connect(chkBoxItalic, SIGNAL(clicked(bool)), this, SLOT(do_chkBoxItalic(bool)));
    connect(btnOK, SIGNAL(clicked()), this, SLOT(accept()));
    connect(btnCancel, SIGNAL(clicked()), this, SLOT(reject()));
    connect(btnClose, SIGNAL(clicked()), this, SLOT(close()));
}
```

### 2.4 CMake构建

CMake是Qt6.0引入的构建系统，要使用CMake构建Qt，就需要在向导中选择构建系统CMake。CMakeLists.txt是CMake项目的配置文件，CMake Modules是项目用到的其他一些CMake模块，具体的模块就行一些后缀为".cmake"的文件。CMakeLists.txt的内容如下：

```cmake
cmake_minimum_required(VERSION 3.19) #要求 CMake 最低版本 3.19
project(HelloCMake LANGUAGES CXX) #定义项目名为 HelloCMake，只使用 C++ 语言

find_package(Qt6 6.5 REQUIRED COMPONENTS Core Widgets)# 查找 Qt6，要求版本 ≥ 6.5,添加Core和 Widgets模块

qt_standard_project_setup() # Qt6 提供的便捷宏，提供自动处理 .ui、信号槽 moc、资源文件

qt_add_executable(HelloCMake
    WIN32 MACOSX_BUNDLE
    main.cpp
    dialog.cpp
    dialog.h
    dialog.ui
) #定义可执行程序 HelloCMake，Windows 不弹控制台，macOS 打成 app 包，源文件含 cpp、h、ui

target_link_libraries(HelloCMake
    PRIVATE
        Qt::Core
        Qt::Widgets
) #链接 Qt 的 Core 和 Widgets

include(GNUInstallDirs) # 引入标准安装路径变量，比如 bin、lib 目录

install(TARGETS HelloCMake
    BUNDLE  DESTINATION .
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
) #定义安装规则，app 包放根目录，可执行文件放 bin，动态库放 lib

qt_generate_deploy_app_script(
    TARGET HelloCMake
    OUTPUT_SCRIPT deploy_script
    NO_UNSUPPORTED_PLATFORM_ERROR
)# 生成 Qt 部署脚本，收集运行时依赖库
install(SCRIPT ${deploy_script})# 安装时自动复制
```

## 3. Qt框架

### 3.1 Qt6模块

Qt模块主要分为两大类：

*   Qt Essentials:Qt框架的基础模块，这些模块提供了Qt在所有平台上的基本功能，在安装Qt时，这些基础模块是自动安装的，无需选择。
*   Qt Addons:Qt框架的附加模块，这些是实现一些特定功能的模块。

#### 3.1.1 Qt基础模块

Qt框架的基础模块提供了Qt在所有平台上的基本功能，它们在所有开发平台上都可用。

![EII87KW3](/imgs/Qt/2106185-20261015091204354-764508239.png)

Qt Core模块是Qt框架的核心，其他模块都依赖此模块。Qt GUI模块提供用于开发GUI应用程序的必要的类。

#### 3.1.2 Qt附加模块

Qt附加模块是一些能够是实现特定功能的模块，用户安装Qt时可以选择性地安装这些附加模块。

![JRIVALF5](/imgs/Qt/2106185-20261015091217196-371677718.png)

Qt的模块很多。针对开发各种应用程序提供了丰富的功能。

### 3.2 Qt全局定义

头文件包含Qt框架中的一些全局定义，包括基本数据类型、函数和宏。一般的Qt类的头文件都会包含这个头文件，所有用户程序中无须包含这个头文件就可以使用其中的定义。

#### 3.2.1 数据类型定义

为了确保在各个平台上的各种基本数据类型都有统一确定的长度，Qt为各种常见数据类型定义了类型符号。

![LEM6IBN5](/imgs/Qt/2106185-20261015091229908-514445459.png)

qreal默认表示8字节double类型的浮点数，若使用-qreal float选项进行配置，则表示4字节float类型浮点数。 qfloat16是Qt5.9中增加的一种类型，用于表示16位浮点型，需要注意**qfloat16不是在头文件中定义**，要使用qfloat16,需要包含头文件。

#### 3.2.2 函数

头文件中包含一些常用函数的定义，这些函数大多以模板类型作为输入和输出参数类型。常用的全局函数定义如下：

![XRQZ5IT8](/imgs/Qt/2106185-20261015091240266-1959738749.png)

![6ZVRTN6L](/imgs/Qt/2106185-20261015091249601-424669457.png)

涉及到基础的数学运算函数在头文件中定义。

#### 3.2.3 宏定义

头文件中定义了很多宏，其中包括：

*   QT\_VERSION表示Qt版本。
*   Q\_BYTE\_ORDER表示系统内存中数据的字节序，Q\_BIG\_ENDIAN表示大端字节序，Q\_LIITLE\_ENDIAN表示小端字节序。
*   Q\_DECL\_IMPORT和Q\_DECL\_EXPORT分别用于在使用或设计共享库时导入或导出库的内容。
*   Q\_UNUSED(name)用于声明函数中未被使用的参数，当函数的输入参数在函数对于的代码里未被使用时，需要用这个宏声明，否则会出现编译警告。
*   foreach(variable，container)用于遍历容器。
*   qDebug(const char\* message,...)用于在debugger窗口显示消息。

### 3.3 Qt元对象系统

Qt中引入了元对象系统对标准C++语言进行扩展，增加了信号与槽、属性系统、动态翻译等功能。

#### 3.3.1 元对象系统概述

Qt的元对象系统功能主要在以下3方面：

*   QObject类是所有使用元对象系统的类的基类
*   必须在一个类的开头部分插入宏Q\_OBJECT,这样这个类才能使用元对象系统的特性
*   MOC为每个QObject的子类提供必要的代码来实现元对象系统的特性构建项目时，MOC会读取C++源文件，当发现类的定义里有Q\_OBJECT宏时，就会为这个类生成另一个包含元对象支持代码的C++源文件，这个生成的源文件连同类的实现文件一起被标准C++编译器编译和连接。

1.  QObject类

QObject类是所有使用元对象系统的类的基类。即如果一个类的父类或上层父类是QObject，它就可以使用信号与槽、属性等特性。

![3GAPMDBS](/imgs/Qt/2106185-20261015091300779-1619168865.png)

元对象系统的特性是通过QObject的一些函数来实现的。

（1）元对象。每个QObject及其子类的示例都有一个元对象，这个元对象是自动创建的。静态变量staticMetaObject就是这个元对象，函数metaObject()返回这个元对象指针，所以有两种方式获取一个对象的元对象：

```c++
QPushButton *btn =new QPushButton();
const QMetaObject *metaPtr=btn->metaObject();
const QMetaObject metaObj=btn->staticMetaObject;
```

(2) 类型信息。QObject的inherits()函数可以判断对象是不是从某个类继承的类的实例。

(3) 动态翻译。函数tr()用于返回一个字符串的翻译版本，在设计多语言界面的应用程序时需要用到tr()函数。

(4) 对象树。对象书指的是表示对象间从属关系的树状结构。对象树中的某个对象被删除时，其子对象也会被自动删除。

(5) 信号与槽。通过在一个类的定义中插入宏Q\_OBJECT，就可以使用Qt扩展的C++语言特性编程。

(6) 属性系统。在类的定义代码中可以使用Q\_PORPERTY定义属性，QObject的setProperty()函数会设置属性的值或定义动态属性，property()函数会返回属性的值。

2.  QMetaObject类

每个QObject及其子类的实例都有一个自动创建的元对象，元对象是QMetaObject类型的实例。元对象存储了了的实例所属类的各种元数据，包括类信息元数据、方法元数据、属性元数据，所以，元对象本质上是对类的描述。

QMetaObject类的主要接口如下所示：

![JDFXV7PJ](/imgs/Qt/2106185-20261015091314014-808165972.png)

![H4GTQLZI](/imgs/Qt/2106185-20261015091323833-1010356924.png)

通过QMetaObject类的这些函数，可以在运行时获取一个QObject对象的类信息和各种元数据。

#### 3.3.2 运行时类型信息

通过使用QObject和QMetaObject提供的接口函数，可以在运行时获得一个对象的类名称以及其父类的名称，判断其是否从某个类继承而来。

(1) 函数QMetaObject::className()。这个函数可以在运行时返回类名称的字符串：

```c++
QPushButton *btn=new QPushButton();
const QMetaObject *meta=btn->metaObject();
QString str=QString(meta->className());
```

(2) 函数QObject::inherits()。这个函数可以判断一个对象是不是继承自某个类的实例，顶层的父类是QObjetc:

```c++
QPushButton *btn=new QPushButton();
bool result=btn->inherits("QPushButton");
result=btn->inherits("QObject");
result=btn->inherits("QWidget");
result=btn->inherits("QCheckBox");
```

如果一个类是多继承的，只有其中有一个父类是包含在内的，那么也会返回true。

(3) 函数QMetaObject::superClass()。这个函数返回该元对象所描述类的父类的元对象，通过父类的元对象可以获取父类的一些元数据：

```c++
QPushButton *btn=new QPushButton();
const QMetaObject *meta=btn->metaObject();
QString str=QString(meta->className());
const QMetaObject *metaSuper=btn->metaObject()->superClass();
QString str2=QString(metaSuper->className());
```

QPushButton的父类是QAbstractButton，所以str是QPushButton，str1是QAbstractButton。

(4) 函数qobject\_cast()。这个函数是头文件中定义的一个非成员函数，对于QObjetc及其子类对象，可以使用函数qobject\_cast()进行动态类型转换，如果自定义的类要支持函数qobject\_cast()，那么自定义的类需要直接或间接继承自QObject,且在类定义中插入宏Q\_OBJECT。

```c++
QObject *btn=new QPushButton();
const QMetaObject *meta=btn->metaObject();
QString str1=QString(meta->className());
QPushButton *btnPush=qobject_cast<QPushButton*>(btn);
const QMetaObject *meta2=btnPush->metaObject();
QString str2=QString(meta2->className());
```

#### 3.3.3 属性系统

1.  属性定义属性是Qt C++的一个扩展的特性，是基于元对象系统实现的，标准的C++语言中没有属性。在QObject的子类中，可以使用Q\_PROPERTY定义属性，其格式如下：

```c++
Q_PROPERTY(type name
          (READ getFunction [WRITE setFunction]) |
          MEMBER memberName [(READ getFunction | WRITE setFunction)])
          [RESET resetFunction]
          [NOTIFY notifySignal]
          [REVISION int | REVISION(int[,int])]
          [DESIGNABLE bool]
          [SCRPTABLE bool]
          [STORED bool]
          [USER bool]
          [BINDABLE bindableProperty]
          [CONSTANT]
          [FINAL]
          [REQUORED])
```

宏Q\_PROPERTY定义一个值类型为type，名称为name的属性，用READ、WRITE关键字分别定义属性的读取、写入函数。宏Q\_PROPERTY定义属性的关键字的含义如下：

*   READ:指定一个读取属性值的函数，没有MEMBER关键字时必须设置为READ。

```c++
class Circle : public QObject
{
    Q_OBJECT
    Q_PROPERTY(double radius READ radius)
public:
    explicit Circle(double r, QObject *parent = nullptr)
        : QObject(parent), m_radius(r) {}

    double radius() const { return m_radius; }

private:
    double m_radius;
};
// 用法
Circle c(5.0);
qDebug() << c.property("radius").toDouble();  // 5
```

*   WRITE:指定一个设置属性值的函数，只读属性没有WRITE配置。

```c++
class Rectangle : public QObject
{
    Q_OBJECT
    Q_PROPERTY(int width READ width WRITE setWidth)
public:
    int width() const { return m_width; }
    void setWidth(int w) { m_width = w; }

private:
    int m_width = 0;
};

// 用法
Rectangle r;
r.setProperty("width", 100);
qDebug() << r.property("width").toInt();  // 100
```

*   MEMBER:指定一个成员变量与属性关联，使之成为可读可写的属性，指定后无须再设置READ和WRITE。

```c++
class Student : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString name MEMBER m_name)
    Q_PROPERTY(int score MEMBER m_score)
public:
    QString m_name;
    int m_score = 0;
};

// 用法
Student s;
s.setProperty("name", "Tom");
s.setProperty("score", 95);
qDebug() << s.m_name << s.m_score;  // Tom 95
```

*   RESET:可选，用于指定一个设置属性默认值的函数。

```c++
class Volume : public QObject
{
    Q_OBJECT
    Q_PROPERTY(int level READ level WRITE setLevel RESET resetLevel)
public:
    int level() const { return m_level; }
    void setLevel(int v) { m_level = v; }
    void resetLevel() { setLevel(50); }   // 默认音量 50

private:
    int m_level = 50;
};

// 用法
Volume v;
v.setLevel(80);
qDebug() << v.level();            // 80
v.resetLevel();
qDebug() << v.level();            // 50
```

*   NOTIFY:可选，用于设置一个信号，当属性值变化时发射此信号。

```c++
class Counter : public QObject
{
    Q_OBJECT
    Q_PROPERTY(int value READ value WRITE setValue NOTIFY valueChanged)
public:
    int value() const { return m_value; }
    void setValue(int v) {
        if (m_value == v) return;
        m_value = v;
        emit valueChanged(m_value);
    }

signals:
    void valueChanged(int newValue);

private:
    int m_value = 0;
};

// 用法
Counter c;
QObject::connect(&c, &Counter::valueChanged,[](int v){ qDebug() << "changed to" << v; });
c.setValue(10);   // 输出: changed to 10
```

*   DESIGNABLE:表示属性是否在Qt Designer的属性编辑器里可见，默认为true。

```c++
class MyWidget : public QWidget
{
    Q_OBJECT
    Q_PROPERTY(QColor bgColor READ bgColor WRITE setBgColor DESIGNABLE true)
    Q_PROPERTY(int internalId READ internalId DESIGNABLE false)
public:
    QColor bgColor() const { return m_bg; }
    void setBgColor(const QColor &c) { m_bg = c; }
    int internalId() const { return 9527; }
private:
    QColor m_bg;
};

// 效果：Qt Designer 里能看到 bgColor，看不到 internalId
```

*   USE:表示这个数不是用户可编辑的属性，默认为false,通常一个类只有一个USER设置为true的属性。

```c++
class MyButton : public QWidget
{
    Q_OBJECT
    Q_PROPERTY(QString text READ text WRITE setText USER true)
public:
    QString text() const { return m_text; }
    void setText(const QString &t) { m_text = t; }
private:
    QString m_text;
};

// 效果：QAbstractButton::checked 就是 USER true 的典型例子
```

*   CONSTANT:表示属性值是一个常数，对于一个对象实例，READ指定的函数返回值是常数，具有CONSTANT关键字的属性不能有WRITE和NOTIFY关键字。

```c++
class AppInfo : public QObject
{
    Q_OBJECT
    Q_PROPERTY(QString version READ version CONSTANT)
    Q_PROPERTY(QString vendor READ vendor CONSTANT)
public:
    QString version() const { return "2.3.1"; }
    QString vendor() const { return "MyCompany"; }
};

// QML 中可读但不可改，也不能有 NOTIFY
```

*   FINAL:表示所定义的属性不能被子类重载。

```c++
class Base : public QObject
{
    Q_OBJECT
    Q_PROPERTY(int id READ id WRITE setId FINAL)
public:
    int id() const { return m_id; }
    void setId(int v) { m_id = v; }
private:
    int m_id = 0;
};

// 下面这样写会编译报错：
// class Derived : public Base {
//     Q_OBJECT
//     Q_PROPERTY(int id READ id2 ...)
// };
```

