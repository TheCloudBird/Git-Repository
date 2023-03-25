
## Qt简介

## UI文件设计与运行机制

QMainWindow： 主窗口类，具有主菜单、工具栏和状态栏，类似一般程序的主窗口。
QWidget: 是所有具有可视化界面的基类，各种界面组件都支持
QDialog：对话框类，建立基于对话框的界面

1. 通过Qt的设计师设计的界面会生成相应的.ui文件，该文件是由xml语言编写的。  例如ui文件为widget.ui, 通过UIC生成ui_widget.h
2. QT通过UIC(c++程序)读取.ui文件(xml),从而生成C++头文件ui_ui文件名.h
3. 定义了Ui_ui文件名的类，类中包含setUpUi(实现了.ui文件)和retranslate(进行翻译)。还定义了Ui的命名空间，在命名空间中声明了ui文件名的类并继承自Ui_ui文件名的类。  
ui的修改只能通过ui设计师修改，不能修改ui_widget.h

### 可视化UI设计

### 代码化UI设计

### 混合方式UI设计

1. Action: 非常有用的类，可以创建菜单项、工具栏按钮

## Qt核心特点

1. Qt对标准C++进行了扩展，引入了一些新的概念和功能
2. 元对象编译器(Meta-Object Compiler， MOC)是一个预处理器
3. Qt先将Qt的特性程序转换为标准C++程序，再由标准C++编译器进行编译

**使用信号与槽机制，只有添加Q_OBJECT宏， moc才能对类里的信号与槽进行预处理**
**Qt为C++语言增加的特性再Qt Core模块中实现，由Qt的元对象系统实现，包括：信号与槽机制、属性系统、动态类型转换等**

### 元对象系统

1. QObject类是所有使用元对象系统的类的基类
2. 在一个类的private部分声明Q_OBJECT宏
3. MOC为每个Qbject的子类提供必要的代码
4. QObject的子类可以通过QObject::metaObject返回类关联的元对象
~~~C++
 QObject *obj = new QPushButton;
 obj->metaObject()->className(); //返回"QPushButton"
~~~
5. 动态类型转换 使用qobject_cast<> 可以进行QObject子类的动态类型转换
~~~C++
QObject *obj = new QMyWidget;

QWidget *widget = qobject_cast<QWidget*>(obj);
QMyWidget* QMyWidget = qobject_cast<QMyWidget*>(obj)
QLabel* label = qobject_cast<QLabel*>(obj);  //不会报错， label为NULL
~~~
6. QObject是Qt对象模型的核心。QObject不支持拷贝，QObject的拷贝函数和赋值运算符是私有的，并且使用了Q_DISABLE_COPY()宏。
7. QObject在对象树中组织自己。当以另一个对象作为父对象创建QObject时，该对象将自动将自身添加到父对象的子对象列表中。父对象删除时，它将自动删除器子对象。可以使用findChildren()按名称和可选类型查找对象。
 

### 主事件循环

１. QCoreApplication: 为非GUI应用程序提供主事件循环
２.　QuiApplication: 为GUI应用程序提供主事件循环
3.   QAppliction: 为Qt Widgets模块提供主事件循环

 继承关系： QObject -> QCoreApplication -> QGuiAppliction -> QApplication  

QCoreApplication包括主事件循环，处理和分发来自操作系统和其他源的所有事件。它还处理应用程序的初始化与终结，以及系统范围和应用程序范围的设置。
  
### 事件与信号

所有GUI应用程序都是事件驱动的。事件主要由应用程序的用户生成，但也可以通过其他方式生成，例如Internet连接、窗口管理器或计时器。当调用exec方法时，应用程序进入主循环。主循环获取事件并将器发送到对象。

### 信号与槽
 Qt具有独特的信号和插槽机制。这种信号和插槽机制是对C++编程语言的扩展； 信号和槽用于对象之间的通信; 槽slot是一种普通的C++函数，当与之相连的信号发出时，调用它。

 #### 几种连接信号和槽的方法。
 1. 使用成员函数指针。  优点：允许编译器检查信号是否与槽的参数兼容。 编译器可以隐式转换参数。(推荐) 如果遇到有重载的函数可以使用qOverload<>宏
    connect(sender,&QObject::destroyed, this, &MyObject::objectDestroyed);
 2. 利用仿函数或lambda表达式作为slot
  connect(sender, &QObject::destroyed, this, [=](){this->m_object.remove(sender);});
 3. 使用SIGNAL()和SLOT()宏 如果参数具有默认值，传递给SIGNAL()宏的签名的参数不得少于传递给SLOT()宏的签名的参数。
    connect(sender, SIGNAL(destroyed(Object*)), this, SLOT(objectDestroyed(Object*)));
    connect(sender, SIGNAL(destroyed(Object*)), this, SLOT(objectDestroyed()));
    connect(sender, SIGNAL(destroyed()), this, SLOT(objectDestroyed()));
     
     connect还可以添加QT::ConnectionType类型的参数，表示信号和槽之间的关系。
     connect的原型： static QMetaObject::Connection connect(const QObject* sender, const char* signal, const QObject* receiver, 
     member, Qt::ConnectionTye = Qt::AutoConnection);
      Qt::AutoConnection(缺省值)：自动确定关联方式。
      Qt::DirectConnection: 信号被发射时，槽立即被执行，槽函数与信号在同一线程内
      Qt::QueuedConnection: 事件循环回到接收者线程后执行槽，槽与信号在不同线程内
      Qt::BlockingQueueConnection: 与Qt::QueuedConnection相似，信号线程会被阻塞直到槽被执行完毕。当槽函数与信号在同一线程，会造成死锁。

    在槽函数里，使用QObject::sender()可以获取信号发射者的指针
    QSpinBox* spinbox=qobject_cast<QSpinBox*>(sender());

 4. 信号和槽还有一种自动关联方式，例如on_pushButton_clicke(),uic自动生成的setupUi()函数中调用connectSlotByName()函数

 #### 自定义信号与槽
  
  1. 信号函数必须无返回值，但可以由输入参数， 信号函数无需实现，只需在某些条件下发射信号。
  2. 需要在发送方的函数中调用信号函数来发送信号。使用关键字 emit
  3. 发送方需要直接或间接继承QObject，必须有Q_OBJECT,信号函数必须有signals:
  4. 接收方可以不用继承QObject，不用有Q_OBJECT,槽函数不用有slot:

### 属性系统

1. 属性的行为类似与类成员,但具有元对象系统的其他特性。
2. 要声明属性，请在继承QObject的类中使用Q_PROPERTY()宏，Q_PROPERTY宏定义一个返回类型为type，名称为name的属性。
3. 可以使用QObject::property和QObject::setProperty读取和写入属性，而不需要知道除属性名称以外的关于类的任何信息。
4. 通过查询类的QObject、QMetaObject、QMetaProperties,可以在运行时发现类的属性

```C++
//属性的操作
QObject* object =    //获取对象指针
const QMetaObject* metaobject = object->metaobject = object->metaObject(); 获取属性的元对象
int count = metaobject->propertyCount(); //获取属性的数量
for (int i = 0; i < count; ++i)
{
  QMetaProperty metaproperty = metaobject->property(i); 获取属性
  const char* name = metaproperty.name(); //属性的名字
  QVariant value = object->property(name); //属性的值

} 

//setProperty可以在运行时为类定义一个新的属性，称之为动态属性

QPushButton* button = new QPushButton;
QObject* object = button;
object->setProperty("flat",true);  //设置一个新的属性并为其赋值                                                                                                                    
bool isFlat = object->property("flat");
```
QSharedPointer是C++中的一个自动共享指针。它的行为与普通指针完全相同，包括const约束的使用。
如果没有其他QSharedPointer对象引用它，当它超出范围时，QSharedPointer将删除它所持有的指针（即回收指向的内存）。
QSharedPointer对象可以从普通指针、另一个QSharedPointer对象或通过将QWeakPointer对象提升为强引用来创建。

创建自动指针
QSharedPointer(MyClass) myinstance = QSharedPointer<MyClass>(new MyClass);
QObject* object = myinstance.get();

## QString

1. QString存储16位QChar（Unicode）字符串
2. QString使用隐式共享（copy-on-write）来提高性能。

### QString常用函数
```
\\\\\
```

## 容器
QList(QVector)、QSet、QMap

## 日期与时间
QDate、QTime、QDateTime

## 界面设计

## Mode/View结构

mode: 与数据通信，并为视图组件提供数据接口
view：是屏幕上的界面组件，视图从数据模型获取每个数据项的模型索引，通过模型索引获取数据
代理：定制数据的界面显示和编辑方式。在标准的视图组件中，代理功能显示一个数据，当数据被编辑时，提供一个编辑器，一般是QLineEdit

### mode类
1. QStringListModel        用于处理字符串列表数据的数据模型
2. QStandardItemModel      标准的基于项数据的数据模型类，每个项数据可以是任何数据类型
3. QFileSystemModel        计算机上文件系统的数据模型类
4. QSortFilterProxyModel   与其他数据模型结合，提供排序和过滤功能的数据模型类
5. QSqlQueryModel          用于数据库SQL查询结果的数据模型类
6. QSqlTableModel          用于数据库的数据表的数据模型类
7. QSqlRelationlTableModel 用于关系型数据表的数据模型类

                       QAbstractListModel    QStringListModel
                       QAbstractProxyModel   QSortFileProxyModel
 QAbstractItemmode     QAbstractTableModel   QSqlQueryModel         QSqlTableModel     QSqlRelationlTableModel
                       QStandardItemModel    
                       QFileSystemModel   

### view类                 

视图组件: 显示数据时，只需要调用数据类的setModel()函数
                      QListView     QListWidget
                      QTableView    QTableWidget
QAbstractItemView     QTreeView     QTreeWidget
                      QColumnView
                      QHeaderView
1. 视图类组件不存储数据
2. 便利类则为组件的每个节点或单元格创建一个项，用项存储数据、格式设置等。
3. 便利类没有数据模型，将界面与数据绑定了。
4. 便利类缺乏对大型数据源进行灵活处理的能力，适用于小型数据的显示与编辑。

### 代理

1. 代理就是在视图组件上为编辑数据提供编辑器
2. 如在表格组件中编辑一个单元格的数据时，缺省是使用一个QLineEdit编辑框。代理负责从数据模型获取相应的数据，然后显示在编辑器里，修改数据后，又将其保存到数据模型中
3. QAbstractItemDelegate是所有代理类的基类

### Model与View的交互

1. 数据模型中存储数据的基本单元都是项（Item），每个项有一个行号、列号，还有一个父项
2. QModelIndex表示模型索引的类。模型索引提供数据存取的一个临时指针
3. 持久模型索引由QPersistenModelIndex类提供
4. 项的角色：在为数据模型的一个项设置数据时，可以赋予其不同项的角色的数据。
5. 模型中的每个项都有一组与其关联的数据元素，每个元素都有自己的角色。视图使用这些角色向模型指示它需要哪种类型的数据。 

### QFileSystemModel

如同Windoes的资源管理器一样。使用QFileSystemModel提供的接口函数，可以创建目录、删除目录、重命名目录，可以获得文件名称、目录名称、文件大小等参数，还可以获取文件的详细信息。
QFileSystemModel* model = new QFileSystemModel(this);
model->SetRootpath(QDir::currentPath());

### QStringListModel

1. setStringList()函数可以初始化数据模型的字符串列表的内容
2. 提供编辑和修改字符串列表数据的函数，如insertRows()、removeRows()、SetData()等

## UI文件设计与运行机制
QMainWindow： 主窗口类，具有主菜单、工具栏和状态栏，类似一般程序的主窗口。
QWidget: 是所有具有可视化界面的基类，各种界面组件都支持
QDialog：对话框类，建立基于对话框的界面

1. 通过Qt的设计师设计的界面会生成相应的.ui文件，该文件是由xml语言编写的。
2. QT通过UIC(c++程序)读取.ui文件(xml),从而生成C++头文件ui_ui文件名.h
3. 定义了Ui_ui文件名的类，类中包含setUpUi(实现了.ui文件)和retranslate(进行翻译)。还定义了Ui的命名空间，在命名空间中声明了ui文件名的类并继承自Ui_ui文件名的类。
### 可视化UI设计

## 布局管理

典型的应用程序由各种widget控件组成，这些可以放置在布局中。我们由两种选择：
1. 绝对定位  控件位于固定的位置，不能改变。
2. 布局管理器  QHBoxLayout(水平布局) QVBoxLayout(垂直布局) QFormLayout(表格布局)  GridLayout(网格布局) 四种布局可以相互嵌套。

### 垂直布局

~~~
auto *vbox = new QVBoxLayout(this);
vbox.addWidget(控件);


~~~~

### 水平布局

~~~
auto hbox = new QHBoxLayout(this);
hbox->addWidget(控件，占空比， 对齐方式);
~~~

### 表格布局

~~~
auto *formLayout = new QFormLayout（this）；
~~~

### 网格布局



