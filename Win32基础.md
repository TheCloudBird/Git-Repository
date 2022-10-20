# Win32学习笔记

## 绪论

### 1. 应用程序的分类

 （1）控制台程序 （console）

​       dos程序，本身没有窗口，通过Windows Dos窗口执行

  （2）窗口程序

​          拥有自己的窗口，可以与用户交互

​    （3）库程序

​               存放代码，数据的程序，执行文件可以从中取出代码执行和获取数据

​               库程序又分为静态库程序和动态库程序

​                   静态库程序：扩展名LIB,在编译链接程序时，将代码放入到执行文件中。

​                   动态库程序：扩展名DLL，在执行文件执行时从中获取代码。

### 2. 应用程序的对比

​    入口函数

​           控制台程序 main

​            窗口程序  WinMain

​            静态库程序   无入口函数

​            动态库程序  DllMain

​     文件存在方式

​             控制台程序、窗口程序 ： exe文件

​            静态库程序 ：LIB文件

​            动态库程序：  DLL文件

### 3. win的库和头文件

 (1)windows库

​     kernel32.dll       提供了核心的API， 例如进程、线程、内存管理等

​     user32.dll          提供了窗口、消息等API

​     gdi32.dll            绘图相关的API

​     路径:c:\Windows\System32

(2) 头文件

​    windows.h    所有Windows头文件的集合

​    windef.h       windows数据类型

​    winbase.h    kernel32的API

​    wingdi.h       gdi32的API

​    winuser.h      user32的API

​    winnt.h           UNICODE字符集支持

   路径：c:/Program Files(x86)\Mircrosoft SDKs\windows\v7.0A\Include

### 4. 标准句柄

Windows有三个标准句柄分别为:标准输入句柄(STD_INPUT_HANDLE)、标准输出句柄（STD_OUTPUT_HANDLE）、标准错误句柄(STD_ERROR_HANDLE)

通过GetStdHandle获取这三个句柄

~~~c++
HANDLE GetStdHandle( _In_ DWORD nStdHandle);
~~~



## 一、窗口类（WNDCLASS）
定义：窗口类是一个属性集，是Windows编程中用于创建窗口的模板。窗口类包含了窗口的各种信息的数据结构，每个窗口都具有窗口类，每个窗口都是基于自己 的窗口类来进行创建窗口的。每一个窗口类都有一个名称，使用窗口类时必须注册到操作系统中去。
分类：窗口类可以分为系统窗口类、应用程序全局窗口类、应用程序局部窗口类。
            系统窗口类——系统中已经定义好的窗口类，所有的的应用程序都可以使用 如：BUTTOＮ　EDIT。
            应用程序全局窗口类——由用户自己定义的，当前应用程序所有模块都可以使用。
             程序局部窗口类——由用户自己定义，当前应用程序中本模块中可以使用
 结构：

~~~C++
 typedef struct_WNDCLASS{
　　 UINT style;                 //窗口类的缓冲大小
　　 WNDPROC lpfnWndProc;        //窗口的消息处理函数的指针
　　 int cbClsExtra;             //窗口类的buff的缓冲大小
　　 int cbWndExtra;             //窗口的buff的缓冲大小
　　 HANDLE hInstance;           //当前窗口的实例
　　 HICON hIcon;                //窗口图标句柄
　　 HCURSOR hCursor;            //窗口鼠标句柄
　　 HBRUSH hbrBackground;       //绘制窗口背景的画刷句柄
　　 LPCTSTR lpszMenuName;       //窗口菜单资源ID的字符串
　　 LPCTSTR lpszClassName;      //窗口类的名称
　　}WNDCLASS;
~~~
    注意：1. 不建议使用全局窗口类，容易发生冲突，全局窗口实现的功能，局部窗口类都可以实现
         2. 用户在创建窗口时，首先需要对窗口进行注册，即通过ATOM RegisterClassEx（CONST WNDCLASSEX *Ipwcx):将窗口类的结构写入操作系统中。
##  二、窗口程序的创建流程
   1. 注册窗口：将窗口类的信息写入操作系统的内核中
   2. 创建窗口：在内存中开辟一片空间，将窗口的数据(包括CreateWindows传入的数据和窗口类的数据)写入内存，并返回该内存的地址（窗口句柄）
   3. 显示窗口：按照内存中的窗口数据绘制窗口
   4. 消息循环：获取、翻译、分发窗口中的消息
   5. 窗口的消息处理函数进行消息处理：处理窗口中的消息

    以上对窗口的创建流程进行了概括，接下来对窗口创建过程进行详细的分析。
## 三、窗口创建的原理
   通过以上分析，我们存在两个个问题。第一，在窗口的创建之前为什么要进行注册？第二，窗口创建和注册之间有什么关系？
###   在窗口的创建之前为什么要进行注册？
   想创建什么样的窗口呢？那么，创建什么样的窗口呢？创建前，Window系统可不知道你要的是什么类型的窗口啊（比如标题栏上显示什么图标，鼠标形状是什么，窗口背景颜色等等）。这些类型信息应在你创建前事先告诉Window系统。可以采用这种方法：就是我们事先写一份要创建窗口的类型申请表，提交（注册）给Window系统。然后在创建时，可以让Windows按这个申请表来创建你所要的窗口了。也就是说我们还应该先提交一个申请表，申请成功后再根据这个表创建一个窗口。
　　我们在使用microsoft平台SDK或者MFC编程时，在创建窗口类后都要先用RegisterClass函数来注册窗口类，这个函数需要一个指向窗口类结构的指针。那么RegisterClass这个windows API函数到底做了什么呢，关于这个函数的源码微软是不会给出来的，因为它只是提供一个系统编程接口，网上也找不到相关说明，只是粗略介绍需要将类注册给系统，但从msdn的atom table说明中我们发现这样一段说明。The system uses atom tables that are not directly accessible to applications. However, the application uses these atoms when calling a variety of functions. For example, registered clipboard formats are stored in an internal atom table used by the system. An application adds atoms to this atom table using。答案有了，在我们构造一个窗口类结构后，我们需要将这个类结构指针加入到system atom table 即SAT中，这样系统就可以通过查找这张表来找到用户自定义的窗口类，window预定义的窗口类指针也在SAT中。SAT实际上实现了一种用于查询的映射，atom实际类型是short，即16位数据。只有系统才可直接访问这张表，但在调用某些api函数时，如Registerclass，可以告知系统来存取这张表。当然，还有本地原子表和全局原子表，这些表应用程序是可以直接访问的。 
  转自: [https://blog.csdn.net/zxj2018/article/details/6636067](https://blog.csdn.net/zxj2018/article/details/6636067)

  ### 窗口创建和注册之间有什么关系？
   通过分析，我们惊奇的发现在创建窗口和构造WNDCLASS的过程中有两个相同参数lpszClassName(窗口名称)和hInstance(窗口实例)，创建窗口和注册窗口通过这两个参数联系起来的。
  CreateWindow函数的执行过程：
  1. CreateWindow首先会根据窗口类的名称在应用程序局部窗口类中查找窗口类，如果找到执行2，若未找到执行3。
  2. 比较局部窗口类与创建窗口类传入的hinstance是否相同，如果相同在同一模块下创建该窗口。否则执行3。
  3. 在应用程序全局窗口类中查找，若查找到执行4，否则执行5。
  4. 使用窗口信息创建窗口并返回。因为要创建的是全局窗口所以不需要比对hInstance。
  5. 在系统窗口类中查找，若找到创建窗口，否则创建窗口失败，返回NULL。

  **CreateWindow找到相应的窗口类后会在内存中开辟一个空间来存储窗口类的信息并返回该内存地址的指针(句柄)。**
## 四、消息简介
 概念：
 消息（Message）指的就是Windows 操作系统发给应用程序的一个通告，它告诉应用程序某个特定的事件发生了。比如，用户单击鼠标或按键都会引发Windows 系统发送相应的消息。最终处理消息的是应用程序的窗口函数，如果程序不负责处理的话系统将会作出默认处理。
 在windows的平台下消息由窗口句柄、消息ID、消息两个附带参数（wParam和lParam）、消息的时间和消息产生时鼠标的位置五个部分组成。
 消息的组成和MSG的成员对应。
 ~~~ c++
 typedef struct  tagMSG {
 HWND hwnd;      //窗口句柄
 UINT message；  //消息ID
 WPARAM wParam;  
 LPARAM lParam;  //两个参数
 DWORD time;     //消息产生的时间
 POINT pt；      //消息产生时鼠标的位置
}MSG， *PMSG，NERA*  NPMSG， FAR* LPMSG；
 ~~~
作用：
当系统通知窗口工作时，就采用消息的方式通过DispatchMessage(&msg);派发给各个窗口的消息的处理函数。
    **通过消息的作用我们可以知道每一窗口都需要一个消息处理函数.**
分类：
消息可以分为系统消息和用户自定义消息
系统消息： ID范围为 0 —— 0X03FF  
系统消息是由系统定义好的消息，可以在系统中直接使用(负责消息处理或消息发送)
用户自定义消息：ID范围为 0X0400—— 0X7FFF
用户消息是由用户自定义的消息，满足用户自己的需求，由用户自己发出，并响应处理。
自定义消息宏 ＃define　WM_USER 0X0400

## 五、消息循环
~~~c++
MSG msg = { 0 };
while (GetMessage(&msg,0,0,0))
{
 TranslateMessage(&msg);
  DispatchMessage(&msg);
}
~~~
接下来对消息循环的中的各个函数进行详细的解释。
### DispatchMessage与窗口消息处理函数的关系
在消息的作用中我们知道DispatchMessage是将消息分发个不同窗口的消息处理函数，那么它时如何找的窗口的消息处理函数的。
接下我们使用伪代码分析一波。
~~~c++
DispatchMessage(msg) {
 通过窗口句柄获取保存窗口信息的内存，
 然后通过WNDCLASS的lpfnWndProc找到窗口消息处理函数。
 最后调用窗口消息处理函数。
}
~~~
为了能够使系统能够识别和调用窗口消息处理函数，我们遵循windows的规定来定义每个窗口的消息处理函数。
消息处理函数的原型为：

~~~c++
LRESULT CALLBACK WindowProc(
HWND hwnd;       //窗口句柄
UINT uMsg;       //消息ID
WPARAM wParam;   //消息参数
LPARAM lParam;   //消息参数
);
~~~
解释：
 **1. 当系统通知窗口时，会调用窗口消息处理函数，同时将消息ID和消息参数传递给窗口消息处理函数。
2. 在窗口消息处理函数中，若不需要处理消息，可以使用缺省的窗口消息处理函数 DefWindowProc();

3. 窗口处理函数的除了返回值和参数列表，其他都可以和windows的原型不一样.**    
      因此窗口消息处理函数就变为了

      ~~~c++
      LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
      {
      	 switch(message)
      	 {
      		    case 需要处理的消息 ： 
      		    {
      		          处理的内容；
      		     }break；
      		    case WM_DESTROY: //窗口销毁 该消息和消息的处理内容后面介绍
      		    {
      		         PostQuitMessage( 0 );
      		    }break;
      		    default :
      		    {
      		        return DefWindowProc(hWnd, message, wParam, lParam); //处理其他消息
      		    }
          }
          return 0；
      }
      ~~~
### GetMessage的简单解释
GetMessage的作用时获取本窗口消息队列中的消息，但是的功能不仅仅如此，具体功能将在后面的消息队列中进行详细的解释。
函数原型：
~~~
BOOL WINAPI GetMessage(
   _Out_ LPMSG lpmsg,         //存放消息的BUFF即MSG
   _In_Opt_ HWND hWnd, //窗口句柄 只获取该窗口句柄指定窗口的消息，NULL表示获取所有窗口的消息。
   _In_ UINT wMsgFilterMin, //获取消息的最小ID
   _In_ UINT wMsgFiterMax //获取消息的最大ID  这两个参数规定了GetMessage获取消息的ID范围，当这两个参数全部为零时，表示获取所有的消息
)；
~~~
通过消息循环的代码可以知道GetMessage的返回值尤为重要，它直接在影响到程序是否能够结束。
那么GetMessage何时返回true，何时返回false?
官方解释：当GetMessage在消息队列中获取到**WM_QUIT**消息时，**GetMessage**就会返回**false**,否则返回true。
因此我们可以通过在**PostQuitMessage( 0 );**向消息队列中放置**WM_QUIT**消息，这样程序就可以结束。
### TranslateMessage
TranslateMessage的作用是将按键消息翻译成字符消息。但是它只能翻译像24字母和数字这样的可见按键，不可以翻译像F1，F2这样的带有功能的不可见按键。
函数原型：
~~~c++
BOOL WINAPI TranslateMessage(
 _In_ CONST MSG* lpMSG; 需要翻译的消息的地址。
)；
~~~
### 常见的消息
通过对常见消息的了解为后面消息循环的原理做铺垫。
当我们见到一个陌生消息时，我们要从消息的产生时间、消息两个附带的参数（wParam和lParam）和消息的一般用法（功能）。
#### （1）WM_DESTROY
产生时间：窗口被销毁时的消息。
附带消息：wParam: 0  lParam：0
一般用法：常用于窗口被销毁之前做一下善后的处理，例如资源、内存等。
**补个小坑**：在上文的窗口处理函数中，我们捕捉了WM_DESTROY消息，然后使用PostQuitMessage( 0 )将WM_QUIT消息放置到消息队列中，让GetMessage获取后退出程序。
#### (2) WM_SYSCOMMAND 
产生时间：当点击窗口最大化、最小化、关闭时。
附带消息：wParam: 鼠标具体点击位置 例如：SC_CLOSE  lParam：LOWORD(低八位) 水平位置  HIWORD(高八位) 垂直位置
一般用法：常在窗口关闭时，提示用户处理。
#### (3) WM_CREATE
产生时间：窗口创建后但未显示时。
附带消息：wParam:0  lParam：是CREATESTRUCT结构的指针，该指针包含了窗口的所有的信息。
一般用法：常用于初始化窗口的参数、资源等。
#### (4) WM_SIZE
产生时间：窗口大小发生变化后。
附带消息：wParam:wParam: 窗口大小变化的原因 lParam：窗口变化后的大小 LOWORD 宽度  HIWORD 高度
一般用法：常用于窗口大小变化后，调整窗口内部各个元素的布局。
### 消息循环的原理
#### 消息信息的阻塞
**GetMessage** ——GetMessage不仅具有从消息队列中获取消息的能力，还能将在获取的同时将消息移除，当消息队列中没有消息时，该函数还会将程序阻塞（线程挂起，系统无法进行调度）等待下一条消息。因此使用GetMessage获取消息的效率低。
**PeekMessage**——PeekMessage具有和GetMessage相同的功能，但是以查看的方式获取消息队列中的消息，可以不将消息移除，如果系统中没有消息，PeekMessage会返回false，不会阻塞程序。
PeekMessage的原型：
~~~c++
BOOL WINAPI PeekMessage（
   _Out_ LPMSG lpmsg,         //存放消息的BUFF即MSG
   _In_Opt_ HWND hWnd,        //窗口句柄 只获取该窗口句柄指定窗口的消息，NULL表示获取所有窗口的消息。
   _In_ UINT wMsgFilterMin,   //获取消息的最小ID
   _In_ UINT wMsgFiterMax     //获取消息的最大ID  这两个参数规定了GetMessage获取消息的ID范围，当这两个参数全部为零时，表示获取所有的消息
   _In_ UINT wRemoveMsg       //移除标识 值为： PM_REMOVE(获取消息并移除 ,不建议使用)/ PM_NOREMOVE(获取但不移除消息)  
）;
~~~
因此为了提高运行效率，建议使用这样的消息循环代码
~~~c++
while(true){
 if (PeekMessage(&msg,NULL,0,0,PM_NOREMOVE)){
     if(GetMessage(&msg,NULL,0,0)){
         TranslateMessage(&msg);
         DispatchMessage(&msg);
      }else {
          return 0;
      }
 }else {
    //空闲处理
  }
}
~~~
#### 发送消息
发送消息是将消息发送到消息队列中或者将消息直接发送给窗口的消息处理函数。接下来介绍发送消息的两个函数。
**PostMessage**将消息发送到消息队列中，不等待消息结果直接返回。
函数原型：
~~~c++
BOOL WINAPI PostMessage(
   _In_opt_  HWND hwnd,
   _In_  UINT Msg,
   _In_ WPARAM wParam,
   _In_ LPARAM lParam
);
~~~
**SendMessage**将消息直接发送到窗口的消息处理函数（调用消息处理函数），等待消息处理函数处理消息，在这个过程中该函数会阻塞程序的执行，直到获取到消息的处理结果。
函数原型：

~~~c++
BOOL WINAPI SendMessage(
   _In_opt_  HWND hwnd,
   _In_  UINT Msg,
   _In_ WPARAM wParam,
   _In_ LPARAM lParam
   );
~~~
 ## 六、消息队列
 消息队列就是用与存放消息的队列，消息在队列中是先近先出的，所有的窗口都有自己的消息队列。
 只有GetMessage会从消息对列中获取消息，且GetMessage只能在消息队列中获取消息。
 **分类**：
 消息队列分为系统消息队列和程序消息队列。
 程序消息队列是由系统维护的消息队列。存放系统产生的消息。如键盘消息、鼠标消息等。其实window系统中产生的消息首先都需要存放到系统消息队列中，系统消息队列每隔一段时间将消息分发到不同窗口（线程）的程序消息队列中。
程序消息队列是由每个程序进行维护的，每个程序都有属于自己的程序消息对列。

**消息的传递过程**
当消息产生时，系统会将消息放到系统消息队列中，系统消息队列每隔一段时间会通过hInstance(实例句柄)将消息分发到不同窗口（线程）的程序消息队列中。然后每个程序的GetMessage在程序消息队列中获取消息。

### 消息与队列的关系
按照消息与队列关系可以分为队列消息和非队列消息。
**队列消息**：队列消息的发送和获取都是通过队列完成的。
队列消息需要通过**PostMessage**将发送后首先会进入队列，然后通过消息循环，再通过**GetMessage**从消息队列中获取。
常见的队列消息：WM_QUIT、键盘、鼠标、定时器等。
**非对列消息**：非队列消息的发送和获取直接通过调用**SendMessage**和窗口消息处理函数来完成。
非队列消息通过SendMessage发送后，会通过窗口句柄找到窗口的消息处理函数，然后调用窗口处理函数处理消息。
常见的非队列消息：WM_CREATE、WM_SIZE等。
### 深谈GetMessage的执行过程
通过以上分析，我们可以知道GetMessage在消息循环起着非常重要的作用。接下来将深入探讨一下GetMessage的执行过程。
1. GetMessage会在程序消息队列中获取消息，如果有消息，GetMessage就会检查消息是否满足指定条件，如果不满足就不会取出消息，如果满足就会取出消息并返回。
2. 如果程序消息队列中没有消息，GetMessage就会向系统消息队列发出向本程序的消息队列中发送属于本程序的消息的申请。如果系统消息队列中有属于本程序的消息，就会转发到该程序的程序的消息队列中。（此时GetMessage会打断系统消息队列每隔一段时间分发消息的时序。）
3. 系统消息队列中没有属于该程序的消息，GetMessage会检查当前程序窗口中需要重新绘制的区域，如果有GetMessage就会调用PostMessage向系统队列发送WM_PAINT的消息，然后执行2获取WM_PAINT消息。
4. 如果没有需要重新绘制的区域，GetMessage会检查定时器，有没有到时间的，如果有GetMessage就会和步骤3一样通过PostMessage向系统消息队列发送WM_TIME的消息，然后执行2获取WM_TIME消息。（其实步骤3和步骤4是同时执行的）
5. 如果没有到时的定时器，GetMessage就会整理资源和内存。
6. 最后GetMessage就会阻塞程序等待下一条消息。PeekMessage也会执行上述步骤，但是PeekMessage执行到此步骤时会返回false,交出程序的控制权，不会阻塞程序。
7. 若GetMessage获得WM_QUIT,GetMessage就会返回false。

## 七、键盘消息

1. 键盘消息的分类
   WM_ KEYDOWN    -按键被按下时产生
   WM_ KEYUP           -按键被放开时产生
   WM_ SYSKEYDOWN     -系统按键按下时产生如ALT F10
   WM_ SYSKEYUP            -系统按键放开时产生
    附带消息:
   WPARAM:  按键的Virtual Key
   LOARAM:   按键的参数例如按下的次数

2. 字符消息(WM _CHAR)
    TranlateMessage在转换WM_KEYDOWN消息时，对于可见字符可以产生WM_CHAR, 不可见字符无此消息
    附带消息:
    WPARAM     -输入的字符的ASCII字符编码值
    LPARAM      按键的相关参数

3. TranlateMessage的执行过程

   ~~~c++
   TranlateMessage (&nmsg)
   if (nmsg.message != WM_KEYDOWN)//不是按键消息不需要进行处理
   {
       return 0;
   }
   // 根据nmsg. wParam (键码值)可以知道是哪个按键被按下
   if(不可见字符串的按键) return 0;
   查看CapsLock(大写锁定键)是否处于打开的状态
   if(打开){
        //发送消息
        PostMessage (nmsg. hwnd, WM_ CHAR, 大写键码值.... ) ;//大写
   }else{
        PostMessage (nmsg.wnd, WM_CHAR, 小写键码值.... ) ;//小写
   }
   ~~~

## 八、鼠标消息

1. 鼠标消息的分类
    (1)基本鼠标消息
      WM_LBUTTONDOWN     鼠标左键按下
      WM_LBUTTONUP           左键抬起
      WM_RBUTTONDOWN    右键按下
      WM_RBUTTONUP           右键抬起
      WM_MOUSEMOVE          鼠标移动
   附带信息:

      WPARAM  其他按键的状态例如Ctr1/Shift
      LPARAM    鼠标的位置窗口客户区坐标系
                        LOWORD        x坐标            HIWORD     y坐标

   2. 双击消息
      WM_LBUTTONDBLCLK      左键双击
      WM_RBUTTONDBLCLK      右键双击
       附带信息:

      ​     WPARAM   其他按键的状态例如Ctr1/Shift
      ​      LPARAM    鼠标的位置窗口客户区坐标系
      ​                         LOWORD    x坐标              HIWORD       y坐标
         双击消息产生的顺序
      ​       WM_LBUTTONDOWN
      ​       WM_LBUTTONUP
      ​       WM_LBUTTONDBLCLK
      ​       WM_LBUTTONUP
      ​     **在使用双击时需要添加CS_DBLCLKS风格。**

      3. 滚轮消息
         WM _MOUSEWHEEL -滚轮消息
          附带信息：

         ​        WPARAM 

         ​                            LOWORD 其他按键的状态例如Ctr1 / Shift 

         ​                            HIWORD 滚轮的偏移量，通过正负表示滚动的方向。 正:向前滚动负:向后滚动

         ​          LPARAM    鼠标的位置屏幕坐标系

         ​                             LOWORD    x坐标     HIWORD    y坐标

## 九、定时器消息

1. 介绍:
   在程序中创建定时器，当到达时间间隔时，GetMessage就会向系统的消息队列中放置WM_TIME消 息。定时器的精度时毫秒，但准确度很低但误差很小。
      附带信息：

   ​         WPARAM      到时的定时器ID
   ​           LPARAM      定时器处理函数的指针

2. 创建和销毁定时器
    (1)  创建定时器

  ~~~C++
  WINUSERAPI UINT_ PTR WINAPI SetTimer (
         _In_opt_ HWND hWnd, 
         _In_ UINT_PTR nIDEvent, //定时器ID
         _In_ UINT uE1apse, //时间间隔
         _In_ opt_ TIMERPROC lpTimerFunc //定时器的处理函数
   );
  ~~~

  (2) 关闭定时器

  ~~~c++
  WINUSERAPI BOOL WINAPI KillTimer (
        _In_opt_ HWND hWnd,
        _In_ UINT_ PTR uIDEvent // 定时器ID
  );
  ~~~

## 十、菜单资源

1. 菜单的分类
      窗口的顶层菜单、弹出式菜单、系统菜单
      HMENU菜单句柄表示菜单ID表示菜单的菜单项

2. 资源相关
   (1) 资源脚本文件: *. rc
   (2) 资源编译器: RC. EXE
     程序编译的过程

   ​                      CL.EXE(类编译器)

​       *.c/*.cpp ---------------------------------------> *.obj |             LINK.EXE(链接器)

​                           RC.EXE(资源编译器)               | ----------------------------------------------> *.exe

​       *.rc --------------------------------------------->*.res |

3. 添加菜单资源

4. 加载菜单资源
   方式一: 注册时窗口类时设置菜单

   ​               在注册窗口时，WNDCLASS中有一个lpszMenuName的属性，可以将菜单资源给它赋值

   ​              例如：wc.lpszMenuName = MAKEINTRESOURCEW(IDC_FINALWINDOW);

   方式二: 创建窗口时传参设置菜单

   ​               在创建窗口时有一个传递菜单句柄的参数，我们可以通过LoadMenu获取菜单句柄，并传递给CreateWindow函数

   方式三: 在主窗口WM_CTEATE消息中利用SetMenu函数设置菜单

                  ~~~
                  BOOL WINAPI SetMenu(
                  	_In_ HWND hWnd，
                  	_In_opt_ HMENU hMenu;
                  );
                  ~~~

   

   加载菜单资源

~~~c++
HMENU LoadMenu (
  HINSTANCE hInstance,
  LOCTSTR lpMenuName
);
~~~

5. 命令消息(WM COMMAND)处理
   附带消息:
    WPARAM - HIWORD -对于菜单为0
                       LOWORD -菜单ID
   LPARAM -   对于菜单为0

## 十一、图标资源

1. 添加图标
   注意图标的大小，一个图标文件中，可以有多个不同大小的图标

   2. 加载图标

      ~~~c++
      HICON LoadIcon (
         HINSTANCE hInstance,
         LPCTSTR lpIconName
      );
      ~~~

      3.图标设置

      ​      注册窗口类

## 十二、光标资源

1. 添加光标资源
     光标的大小默认是32*32像素，每个光标有HotSpot (热点)

2. 加载资源

   ~~~C++
   HICON LoadCursor (
       HINSTANCE hInstance,
       LPCTSTR lpCur sorName
   );
   ~~~

3. 光标设置
   (1) 在注册窗口时，设置光标
   (2) 使用SetCursor设置光标

   ~~~c++
   WINUSERAPI HCURSOR WINAPI SetCursor
      In_ opt_ HCURSOR hCursor
     );
   ~~~

   该函数必须在WM_ SETCURSOR的消 息下才能使用
   (3) WM_SETCURSOR
             产生时间:光标移动的时候也就是鼠标移动
         附带消息:
        WPARAM  -   当前使用的光标据句柄
         LPARAM  -   HIWORD -当前鼠标消息的ID
                              LOWORD -当前区域的代码HTCLIENT (光标在客户区活动) HTCAPTION (光标在标题栏活动)。

## 十三、字符串资源

1. 添加字符串资源
   添加字符串表，在表中增加字符串

2. 字符串资源的使用

   ~~~c++
   WINUSERAPI int WINAPI LoadString (
       _In_ _opt_ HINSTANCE hInstance,
       _In_ UINT uID, //字符串ID
       _Out_ writes_ to_ (cchBufferMax, return +1) LPWSTR 1pBuffer, //存 放字符串的BUFF 
       _In_  int cchBufferMax //字符串BUFF的长度
    );
   ~~~

## 十四、快捷键资源

1. 添加资源快捷键表，增加命令ID对应的快捷键一般将菜单项与快捷键进行绑定

2. 使用加载快捷键表

   ~~~C++
   WINUSERAPI HACCEL WINAPI LoadAcceleratorsW(
        _In_opt_ HINSTANCE hInstance,
        _In_ LPCWSTR lpTab1eName
   );
   ~~~

   3. 翻译快捷键

   ~~~C++
   WINUSERAPI int WINAPI TranslateAcce leratorW (
      _In_ HWND hWnd,
      _In_ HACCEL hAccTable, //快捷键句柄
      _In_ LPMSG lpMsg //消息
    );
   ~~~

   4. 翻译快捷键的执行过成

   ~~~c++
   int WINAPI TranslateAcceleratorW (hWnd, hAccTable, &nMsg) {
      if (nMsg. Message != WM_ KEYDOWN){ //非按键按键消息
          return 0;
      }
      根据nMsg. wParam(键码值)，获知哪些按键被按下
      拿着键值码到快捷键表中去匹配查找
     if (没找到){
         return 0; 
     } 
     if (找到){
        进行翻译
        SendMessage (hWnd, WM COMMAND, 键值码，..);
        return 1;
      }
   }
   ~~~

   该函数放置到消息循环中去，因此消息循环变为了：

   ~~~c++
   HACCEL hAccelTable = LoadAccelerators (hInstance, MAKE INTRESOURCE (IDC_ WINWINDOWSTWO)) ;
   MSG msg;
   //主消息循环:
   while (GetMessage (&msg, nul1ptr, 0, 0))
   {
       if (!TranslateAccelerator(msg. hwnd, hAccelTable, &msg))
       {
            TranslateMessage (&msg) ;
            DispatchMessage (&msg) ; 
        }
   }
   ~~~

   5. 快捷键也会发送WM COMMAND的消息
         附带消息:
      WPARAM

   ​         HIWORD-   对于菜单为0   对于快捷键为1
   ​          LOWORD   命令ID
   ​     LPARAM    0

## 十五、绘图编程

1. 绘图设备DC (Device Context),绘图上下文/绘图描述表
    （1）通过BeginPaint来获取绘图设备
            函数原型： 

  ~~~c++
  WINUSERAPI HDC WINAPI BeginPaint (
       _In_ HWND hWnd,
       _Out_ LPPAINTSTRUCT 1pPaint
  );
  ~~~

  ​     通过EndPaint来结束绘制
  ​     函数原型：

  ~~~c++
  WINUSERAPI HDC WINAPI EndPaint (
      _In_ HWND hWnd,
      _Out_ LPPAINTSTRUCT lpPaint
  );
  ~~~

  ​    HDC 是绘图设备的句柄
  ​    GDI Windows graphics device interface (Win32提 供的绘图APT)

  2. 颜色: RGB每一个点颜色是3个字节24位保存

     | 位数 | R    | G    | B    | 透明度 |
     | ---- | ---- | ---- | ---- | ------ |
     | 16位 | 5    | 5    | 6    | 无     |
     | 32位 | 8    | 8    | 8    | 8      |

     (1) 颜色的使用
        COLORREF  一  实际是DWORD = unsigned long
               例如COLORREF nColor = 0;
         赋值使用RGB宏
             例如nColor = RGB(0. 0, 255) ;

     ​    获取RGB值
     ​        GetRValue/GetGValue/GetBValue
     ​        例如: BYTE nRed = GetRValue (nColor);

  3. 基本图形绘制
     (1) 使用SetPixe1设置指定点的颜色，就是画一个点

     ~~~c++
     WINGDIAPI COLORREF WINAPI SetPixe1 (
         In_ HDC hde,  //绘图设备句柄
         _In_ int x,
         _In_ int y,
         _In_ COLORREF color
       );//返回原来点的颜色
     ~~~

      (2) 绘制线(直线、弧线)
          MoveToEx -- 修改窗口当前的点，每一个窗口都有自己的当前点，默认在(0, 0)。

     ​     MoveToEx 函数原型:

     ~~~c++
     WINGDIAPI BOOL WINAPI MoveToEx (
          _In_ HDC hdc,
          _In_ int X,
          _In_ int y,
          _Out_opt_ LPPOINT lppt //原来的当前点
     );
     ~~~

     LineTo一从窗口当前点到指定点绘制一条直线
     LineTo函数原型:

     ~~~c++
     WINGDIAPI BOOL WINAPI LineTo(_ In_ HDC hdc,_ In_ int x, _In_ int y);
     ~~~

       (3) 封闭图形:能够使用画刷填充的图形
              Rectangle(矩形) / ELLipse(圆形)

  4. GDI绘图对象
     (1) 画笔
         画笔的作用:线的颜色、线型(实线、虚线)、线粗
                  HPEN:画笔句柄
         画笔的使用

     1. 创建画笔

     ~~~c++
     WINGDIAPI HPEN WINAPI CreatePen(
               _In_ int iStyle,   //画笔的样  PS_ SOILD 实心线
               _In_ int cWidth,  //画笔的粗细
               _In_ COLORREF color //画笔的颜色
     );
     ~~~

     2. 将画笔应用到DC

     ~~~c++
     WINGDIAPI HGDIOBJ WINAPI Select0bject(
            _In_ HDC hde, //绘图设备句柄
            _In_ HGDIOBJ h //GDI绘图对象句柄，画笔句柄
     ); // 返回原来的GDI绘图对象句柄  注意一定要接收一 下
     ~~~

     3. 绘图  
     4.  取出DC中的画笔
         将原来的画笔，使用Seletboeet函数， 放入到设备DC中，就会将我们创建的画笔取出。

     5. 释放画笔

     ~~~c++
     WINGDIAPI BOOL WINAPI DeleteObject (
           _In_ HGDIOBJ ho //需要释放的对象
     );
     ~~~

     ​     **只能删除不被DC使用的画笔，所以在释放前，必须将画笔从DC中取出**
     (2) 画刷
        画刷的作用:为封闭图形填充颜色、图案
     ​           HBRUSH:画刷句柄
     ​    画刷的使用与画笔的使用一 样
     ​         创建画刷     

     ​                CreateSolidBrush     创建实心画刷
     ​                CreateHatchBrush    创建纹理画刷

     5. 位图(. bmp)
        位图相关
        光栅图形      记录图像中每一点的颜色等信息
        矢量图形       记录图像算法、绘图指令
           HBITMAP    位图句柄
        位图使用

        1. 在资源中添加位图资源

        2. 从资源中加载位图LoadBitmap

        3. 创建一个与当前DC相匹配的DC(内存DC)

        ~~~c++
        HDC CreateCompatibleDC(
                HDC hdc //当前DC句柄，可以为NULL (使用屏幕DC)
            );  //返回创建好的DC
        ~~~

        4. 将位图放入匹配的DC中Select0bject

        5. 成像(1:1)

        ~~~c++
        WINGDIAPI BOOL WINAPI BitBlt(
                 _In_ HDC hdc, //目的DC
                 _In_ int x,   //左上x
                 _In_ int y,   //左上Y
                 _In_ int cx,  //宽度
                 _In_ int cy,  //高度
                 _In_ opt_ HDC hdcSrc, //源DC
                 _In_ int x1, //源左上x
                 _In_ int y1,
                 _In_ DWORD rop //成像方法SRCCOPY(原样成像)
            );
        ~~~

        6. 取出位图SelectObject

        7. 释放位图DeleteObject

        8. 释放DC DeleteDC

     6. 文本绘制
        (1)TextOut一将文字绘制在指定坐标的位置
        (2) DrewText
        (3) SetTextColor 设置字符颜色
             SetBkColor设置字体背景颜色
             SetBkMode设置背景显示模式(OPAQUE(不透明 一默认) / TRANSPARENT (透明))

     7. 字体相关
        window常用的字体为TrueType(真实字的点阵)格式的字体文件
          字体名--标识字体类型
           HFONT --字体句柄
        字体的使用
          (1) 创建字体CreateFont
          (2)将字体应用到DC
          (3) 取出DC中的字体
          (4) 释放字体

## 十六、静态库

1. 静态库的特点
        运行不存在   静态库源码被链接到调用程序中    目标程序的归档

2. c语言静态库
   (1) 创建一个静态库项目
   (2) 添加库程序，源文件使用c文件
       使用C语言静态库
       库路径设置:可以使用pragma关键字设置
        #pragma comment (1ib,“库路径)

3. C++静态库的创建
   (1) 创建一个静态库项目
   (2) 添加库程序，源文件使用C++文件

   (3) 使用c++语言静态库
       库路径设置:可以使用pragma关键字设置
            #pragma comment(1ib, ”库路径") C++静 态库需要声明
   注意:**如果在C++文件中去调用c库会调不到，因为C++编 译器在编译时会改变C++函数的名称如Add函数名会改为?Add@YAHHH@Z,若想要调用可以让c++编译不去改变函数的名称**
   方法:在C库函数声明的前面添加extern"C"
   如: extern" C”int Add(int a, int b)

## 十七 、动态库

1. ### 动态库特点

   （1）运行时独立存在   (2)源码不会链接到执行程序 （3） 使用时加载(使用动态库必须使用动态库执行)

2. ### 与静态库的比较

   （1）由于静态库是将代码嵌入到使用程序中，多个程序使用时，会有多分代码，所以代码体积会增大。动态库的代码只需要存在一份，其他程序通过函数地址使用，所以代码体积小。

   （2） 静态库发生变化后，新的代码需要重新链接嵌入到执行程序中。动态库发生变化后，如果库中函数的定义（或地址）未变化，其他使用DLL的程序不需要重新链接。

3.  动态库创建

   （1）创建动态库项目  (2)添加库程序

   （3）库程序导出 -- 将动态库中封装的函数的地址导出 导入到了 .dll的文件中  

   ​          方式一： 声明导出 -- 使用_declspec(dllexport)导出函数

   ​                        例如：_declspec(dllexport) int Add(int a, int b) { return a + b;}

   ​           注意：**动态库编译链接后，也会有LIB文件，是作为动态库函数映射使用，与静态库不完全相同。**

   ​          方式二：模块定义文件.def

   ​                     例如：LIBRARY DLLFunc //库

   ​                               EXPORTS              //库导出表

   ​                               DLL_Mul     @1      //导出函数

4. dll文件和与.dll配套的.lib的结构

   (1).dll主要存储了函数的名称地址以及函数的源码

   |        函数序号        |   函数名称    | 函数地址 |
   | :--------------------: | :-----------: | :------: |
   |           0            | ?Add@@YAHHH@Z | 0xfafba  |
   |           1            | ?Sub@@YAHHH@Z | 0xfafbd  |
   |           2            | ?Mul@@YAHHH@Z | 0xfafbf  |
   |          ...           |      ...      |   ...    |
   | 此部分为所有函数的源码 |               |          |

   (2) .lib主要存储了函数的名称和标号 与 .dll中一致

   | 函数名称 | 序号          | dll文件名称 |
   | -------- | ------------- | ----------- |
   | 0        | ?Add@@YAHHH@Z |             |
   | 1        | ?Sub@@YAHHH@Z |             |
   | 2        | ?Mul@@YAHHH@Z |             |

5. 动态库的使用

   （1）隐式链接(操作系统负责动态库的执行)

   ​        1）头文件和函数原型

   ​             可以在函数原型的声明前，增加_declspec(dllimport)

   ​             例如：_declspec(dllimport) int Add(int a, int b) ;

   ​          2）导入动态库的LIB文件  库路径设置：#pragma comment(lib,"库路径")

   ​          3）在程序中使用函数

   ​          4）隐式链接，dll文件可以存放的路径

   ​                   与执行文件在同一个目录中

   ​                   当前工作目录

   ​                   Windows目录

   ​                   Windows/System32目录

   ​                   Windows/System

   ​                   环境变量PATH指定目录

   （2）显式链接

   ​       1）定义函数指针类型 typedef  

   ​            例如： typedef int(*Add)(int m, int n);

   ​       2)  定义动态库 

   ​         HMODULE LoadLibrary(

   ​               LPCTSTR lpFileName //动态库文件名或全路径

   ​           ); //返回DLL的实例句柄

   ​        3）获取函数绝对地址

   ​             FARPR GetProcAddress(

   ​                       HMODULE hModule, //DLL句柄

   ​                        LPCSTR lpProcName  //函数名称

   ​               );成功返回函数地址

   ​         4）使用函数

   ​         5）卸载动态库

   ​                BOOL FreeLibrary(

   ​                        HMODULE hModule //DLL句柄

   ​               );

6. 动态库中封装类

   (1) 在类名前增加 _declspec(dllexport) 定义

   例如：class _declspec(dllexport) CMath {

   

   };

   (2) 通常使用预编译开关切换类的导入导出定义，

     例如： #ifdef DLLCLASS_EXPORTS

   ​              #define EXT_CLASS _declspec(dllexport)//DLL

   ​              #else

   ​               #define EXT_CLASS _declspec(dllimport)//使用者

   ​               #endif

   ​              Class EX_CLASS CMath{

   ​               .......

   ​              };

   ## 十八、线程

   1. ### 线程基础
   
      Windows线程是可以执行代码的实例。系统是以线程为单位调度程序。一个程序当中可以有多个线程，实现多任务处理，但主线程就有一个。进程开启意味着在内存中开辟空间，线程开启意味着程序的执行。
   
      #### Windows线程的特点：
   
      1. 每个线程都具有1个ID
      2. 每个线程都具有自己的内存栈
      3. 同一进程中的线程使用同一个地址空间。
   
      #### 线程的调度
   
      将cup的执行时间划分成时间片，依次根据时间片执行不同的线程。
   
      线程轮询：线程A->线程B->线程A。。。。
   
   2. ### 创建线程
   
      ~~~c++
      HANDLE　CreateThread(
        LPSECURIYT_ATTRIBUTES lpThreadAttributes,//安全属性(废弃)
        SIZE_T dwStackSize,                      //线程栈的大小
        LPTHREAD_START_ROUTINE lpStartAddress,   //线程处理函数的函数地址
        LPVOID lpzParameer,                      //传递给线程处理函数的参数
        DWORD dwCreatuibFlags,                   //线程创建方式   立即启动(0)和挂起状态(CREATE_SUSPENDED)
        LPDWORD lpThreadId                       //创建成功，返回线程的ID
      );
      ~~~
   
      定义线程处理函数的原型
   
      ~~~c++
      DWORD WINAPI ThreadProc( 
        LPVOID lpParamter  //创建线程时，传递给线程的参数
      );
      ~~~
   
      
   
   3. ### 线程挂起\销毁
   
      #### 挂起
   
      ~~~c++
      DWORD SuspendThread( 
      
          HANDLE hThread //handle to thread
      );
      ~~~
      
      #### 唤醒
      
      ~~~
      DWORD ResumeThread (
         HANDLE hThread 
      );
      ~~~
      
      #### 结束指定的线程
      
      ~~~C++
      BOOL TerminateThread( 
      
            HANDLE hThread,
            DWORD dwExitCode  //exit code
      
      );
      ~~~
      
      结束函数所在的线程
   
      ~~~c++
      VOID ExitThread(
      
         DWORD dwExitCode  //exit code
      
      );
      ~~~
      
   4. ### 线程的相关操作
   
       GetCurrentThreadId  -- 获取当前线程ID
   
       GetCurrentThread  -- 获取当前句柄
   
      等待单个句柄有信号 可等候的句柄(线程句柄)必须具备有信号和无信号两种状态
   
      **当线程结束时，线程才会有信号** 
   
      ~~~c++
      VOID WaitForSingleObject(
         HANDLE handle, //句柄BUFF的地址
      　 DWORD　dwMilliseconds  //等待时间INFINITE(无限)
      );
      ~~~
   
      同时等候多个句柄信号
   
      ~~~c++
      DWORD WaitForMultipleObjects(
         DWORD nCount,                //句柄数量
         CONST HANDLE *lpHandle,      //句柄BUFF的地址
         BOOL bWaitAll,               //等待方式
         DWORD dwMilliseconds         // 等候时间 INFINITE
      );
      bWaitAll -- 等待方式
          TRUE　-- 表示所有句柄都有信号，才结束等待
          FASLE -- 表示句柄只要有1个信号，就结束等待
      ~~~
   
   ## 线程同步
   
   ### 原子锁
   
   
   
   ## 十九、增加console调试程序
   
   需要获取标准输入句柄，然后通过WriteConsole()向窗口打印数据
   
   1.声明全局标准输入句柄  HANDLE　g_stdOutput = 0;
   
   添加console窗口 AllocConsole();
   
   输出信息   WriteConsole
   
   