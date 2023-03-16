# MFC笔记

## 绪论

### MFC的概念和作用

### 常用的头文件

afx.h  - 将各种MFC头文件包含在内

afxwin.h - 包含了各种MFC窗口类。包含了afx.h和windows.h

afxext.h - 提供了扩展窗口类的支持，例如工具栏、状态栏。

### MFC的程序的分类

#### MFC控制台程序

和Win32的控制台程序的差别

1. main函数不同与普通的控制程序
2. CWinApp theApp;多了一个全局对象。

经验之谈：

   以Afx开头可以确定为MFC库中的全局函数

   以::可以确定为win32的API函数

#### MFC窗口程序

   MFC的窗口程序分为三类，分别为单文档视图架构程序，多文档试图架构程序，对话框应用程序

##### 单文档视图架构程序

CWinAPP   -------      应用程序类，负责管理应用程序的流程

CFrameWnd  -------   框架窗口类，负责管理框架窗口

CView           -------   视图窗口类，显示数据

CDocument   -------  文档类，负责管理数据

##### 多文档视图架构程序

CWinAPP   -------      应用程序类，负责管理应用程序的流程

CMDIFrameWnd  -------  多文档主框架窗口类

CMDIChildWnd  -------  多文档子框架窗口类

CView           -------   视图窗口类，显示数据

CDocument   -------  文档类，负责管理数据

##### 对话框应用程序

CWinAPP   -------      应用程序类，负责管理应用程序的流程

CDialog     -------       对话框窗口类

#### MFC的库程序

使用MFC制作静态库程序

MFC的动态库

​     使用静态的MFC库制作自己的动态库

​     使用动态的MFC库制作自己的动态库

### MFC库中相关的类介绍

1. CObject类

   MFC类库中绝大部分类的父类，提供了MFC类库中一些基本的机制。包括对运行时类信息的支持、对动态创建的支持、对序列化的支持

2. CWinAPP类  应用程序类，封装了应用程序、线程等信息

3. CDocument类  文档类，管理数据

4. Frame Windons 框架窗口类，封装了窗口程序组成的各种框架窗口

5. CSplitterWnd 用来完成拆分窗口的类

6. Control Bar  控件条类

7. Dialog Boxs  对话框类，封装了各种对话框，通用的对话框

8. views  视图类，封装了各种显示窗口

9. Controls 控件类，封装了各种常用的控件

10. Exceptions 异常处理类，封装了MFC中的各种异常

11. File 文件类，各种文件的I/O操作等

12. 绘图类  包括CDC类和CGdiobject类

13. 数据集合类  CArray/Clsit/CMap,封装了相应的数据结构的管理

14. 非CObject类的子类   提供了各种数据结构相关的管理，CPoint、CTime、CString

## 第一个简单的MFC程序

为了能够更好的了解MFC的执行过程，我们将使用Win32创建项目，然后修改为MFC的项目。因为win32的项目和MFC的项目取决于能否使用MFC库

项目创建过程

1. 创建win32项目，在项目属性-->高级，将MFC的使用改成使用MFC库

2. 添加CPP文件，包含#include <afxwin.h> 

3. 书写代码

   （1） 定义自己的框架类，派生自CFrameWnd类
   (2)   定义自己的应用程序类，派生自CWinApp类，并定义构造函数以及重写InitInstance虚函数，在程序中创建并显示窗口。
   ~~~c++
   CMyWinApp::CMyWinApp()
   {
       
   }
   BOOL CMyWinApp: : InitInstance ()
   {
       CMDIFrameWnd* pFrame = new CMDIFrameWnd(); //创建框架类
       pFrame->Create(NULL,_T("MFCBase"));        //创建窗口
       m_pMainWnd = pFrame;                       //将框架类对象保存到theApp的属性中
       pFrame->ShowWindow(SW_SHOW);               //显示窗口
       pFrame->UpdateWindow();
       return TRUE;
   }
   ~~~
   
   (3) 创建全局变量  CMyWinApp的对象    
   
   ## MFC程序的启动
   
   ### 入口函数
   
   MFC的入口函数和win32窗口程序相同，都是从WinMain入口。但是MFC库已经实现了WinMain函数，所以在程序中不需要实现。
   
   因此，在Win32中WinMain由程序员自己实现，那么流程是程序员安排的，但到了MFC中，由于MFC库实现了WinMain，也就意味着MFC负责安排程序的流程。
   
   ### 执行流程
   
   通过程序断点调试并通过伪代码的方式，获取MFC的执行流程。
   
   **在MFC中存在三个全局变量分别为 AFX_MODULE_STATE  pModuleState 当前程序模块状态信息 、AFX_MODULE_THREAD_STATE*  pModuleThread当前模块线程状态信息、 AFX_THREAD_STATE*  pThreadState 当前线程状态信息**
   
   1. 全局对象 theApp的构造过程。
   
      ~~~c++
      //theApp基类构造函数
      CWinApp::CWinApp()
      {
          /*  该行代码，获取全局变量的地址    在MFC中有三个比较重要的全局变量 
              第一个全局变量: AFX_MODULE_STATE  pModuleState 当前程序模块状态信息
          */
         AFX_MODULE_STATE* pModuleState = _AFX_CMDTARGET_GETSTATE();  //_AFX_CMDTARGET_GETSTATE();是一个宏，指代着AfxGetModutate();
          /*
           获取第二个全局变量   AFX_MODULE_THREAD_STATE* pThreadState 当前程序线程状态信息 
          */
         AFX_MODULE_THREAD_STATE* pThreadState = pModuleState->m_thread;
      
         ASSERT(AfxGetThread() == NULL);//判断当前线程中的m_pCurrentWinThread是否为NULL
          /*
            m_pCurrentWinThread为空
            将theApp的保存到全局变量pThreadState的成员变量m_pCurrentWinThread中
             moduleState、 pThreadState、theApp的存储关系
             pModuleState->m_thread(pThreadState)->m_pCurrentWinThread(theApp)
          */
         pThreadState->m_pCurrentWinTread = this; //该this指代theApp;
         ASSERT(AfxGetThread() == this);//判断是否赋值成功
         //AfxGetThread内部执行过程见下面
          
          //这两个是theApp的属性
          this -> m_hThread = ::GetCurrentThread();
          this -> m_nThreadID = ::GetCurrentThread();
          
          /*
           此时pModuleState、 pThreadState、theApp的存储关系
              pModuleState->m_pCurrentWinApp(theApp)
              pModuleState->m_thread(pThreadState)->m_pCurrentWinThread(theApp)
          */
          pModuleState -> m_pCurrentWinAPP = this;
          ASSERT(AfxGetAPP() == this);
          //AfxGetApp的内部执行过程见下面
      }  
      ~~~
   
      **AfxGetThread内部执行过程**
   
      ~~~c++
       AfxGetThread(){
              //获取全局变量  AFX_MODULE_THREAD_STATE* pThreadState 当前程序线程状态信息 
              AFX_MODULE_THREAD_STATE* pState = AfxGetModutateThreadState();
              CWinThread* pThread = pState -> m_pCurrentWinThread; //这是theApp；
              return pThread;   //返回的是theApp的地址
           }
      ~~~
   
         **AfxGetApp的内部执行过程**
   
      ~~~c++
      //AfxGetApp的内部执行过程见下面
      AfxGetAPP(){
          return afxCurrentWinApp;  //afxCurrentWinApp是一个宏指代着AfxGetModutate()-> m_pCurrentWinApp; //返回theApp  
      }
      ~~~
   
      总结：当程序启动时，构造theApp对象，调用父类CWInApp的构造函数。
   
      ​          主要功能是将theApp对象保存到两个MFC的全局变量中。
   
      ​           AFX_MODULE_STATE  moduleState 当前程序模块状态信息  
   
      ​           AFX_MODULE_THREAD_STATE* pThreadState 当前程序模块线程状态信息 
   
      ​     补充：  综上所知，全局变量AFX_MODULE_STATE  pModuleState 和  AFX_MODULE_THREAD_STATE* pModuleThread存在一一对应的关系，即pModuleThread->m_moduleState =   pModuleState 、 pModuleState->m_thread =    pModuleThread
   
   2. MFC程序执行流程
   
       通过调用堆栈获取InitInstance()的调用过程，找到MFC的main函数，MFC的main函数是_tWinMain(),它调用了AfxWinMain
   
      **AfxWinMain的执行过程**
   
      ~~~c++
      int AFXAPI AfxWinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance, _In_ LPTSTR lpCmdLine, int nCmdShow){
           //首先获取了theApp  pThread和pApp都是全局变量theApp
          CWinThread* pThread = AfxGetThread();
          CWinApp* pApp = AfxGetApp();
          /*
            由pApp->InitApplication()可知，InitApplication()是theApp的成员函数，且InitApplication()是虚函数，我们没有对InitApplication()进行重       写，因此它会执行父类的InitApplication()进行初始化
          */
         if (pApp != NULL && !pApp->InitApplication()) goto InitFailure;
          /*
            接下来又对实例进行初始化，InitInstance()是theApp的成员函数，并且我们进行了重写，因此执行我们的代码，在InitInstance()中，我们创建了Frame       的对象并通过Frame创建了窗口，并将窗口进行显示。最后我们还将Frame对象保存到theApp的成员属性m_pMainWnd中
          */
         if (!pThread->InitInstance())
         {
             。。。。。。
         }
          //进行消息循环 该Run函数用了CWinThread::Run()函数
          nReturnCode = pThread->Run();
      }
      ~~~
      
       **CWinThread::Run()内部执行过程**
      
      ~~~C++
      //CWinThread::Run()内部执行过程
          int CWinThread::Run()
          {
             //获取MFC中的第三个全局变量 该变量中存放着消息
             AFX_THREAD_STATE*  pState = AfxGetThreadState();
             BOOL bIdle = TRUE;
             //死循环
             for(;;)
             {
               //当消息队列中没有消息时进入循环
               while(bIdle && !::PeekMessage(&(pState->msgCur),NULL,NULL,NULL,PM_NOREMOVE))
               {
                   //OnIdle()用来进行空闲处理 它是theApp的成员虚函数，可以进行重写
                  if(this->OnIdle(this -> lIdleCount++)) this->bIdle = FALSE;
               }
                 
               do
               {
                 //当PumpMessage()返回false时就可以退出死循环，就是当GetMessage获取到WM_QUIT时才会退出
                 if(!PumpMessage()) return ExitInstance();  //程序结束之前进行一些处理
                      // PumpMessage()内部直接调用了全局函数AfxInternalPumMessage();
                       // AfxInternalPumMessage();的内部执行见下面    
               }while(::PeekMessage(&(pState->msgCur),NULL,NULL,NULL,PM_NOREMOVE));//有消息时不断循环
            }
       } 
            
      ~~~
      
      **AfxInternalPumMessage();的内部执行**
      
      ~~~C++
         BOOL  AfxInternalPumMessage()
         {
         // GetMessage获得WM_QUIT消息时才会进入
             if(!::GetMessage(&(pState->msgCur),NULL,NULL,NULL))
             {
               return FALSE;
             }
             if(pState->m_msgCur.message != WM_KICKIDLE && !AfxPreTranslateMessae(&(pState->m_msgCur)))
             {
                 //消息翻译
               ::TranslateMessage(&(pState->m_msgCur));
                 //消息分发
               ::DispatchMessage(&(pState->m_msgCur));
             }
            return true;
         } 
      ~~~
      
      
      
      总结：进入入口函数WinMain
      
      ​          首先获取应用程序类对象theApp，
      
      ​          利用theApp调用InitApplication，初始化当前应用程序的数据
      
      ​          利用theApp调用InitInstance初始化程序，在函数中我们创建窗口并显示
      
      ​          利用theApp调用CWinApp的Run函数进行消息循环
      
      ​          如果没用消息，利用theApp调用OnIdle虚函数进行空闲处理
      
      ​          程序退出利用theApp地址调用ExitInstance虚函数实现退出前的善后处理工作
      
      通过以上可以知道theApp控制着程序的流程
      
      现在我们见到的CWinApp的成员
      
      ​     成员虚函数
      
      ​          InitApplication、InitInstance、Run、OnIdle、ExitInstance
      
      ​     成员变量 
      
      ​          m_pMainWnd      当前应用程序主窗口 
      
      ## MFC窗口和消息
      
      ### 钩子函数简介
      
      #### 简介
   
      钩子函数是Windows消息处理机制的一部分，通过设置“钩子”，应用程序可以在系统级对所有消息、事件进行过滤，访问在正常情况下无法访问的消息。钩子的本质是一段用以处理系统消息的程序，通过系统调用，把它挂入系统。
   
      WINDOWS的钩子函数可以认为是WINDOWS的主要特性之一。利用它们，您可以捕捉您自己进程或其它进程发生的事件。通过“钩挂”，您可以给WINDOWS一个处理或过滤事件的回调函数，该函数也叫做“钩子函数”，当每次发生您感兴趣的事件时，WINDOWS都将调用该函数。
      
      #### 创建钩子
      
      ~~~c++
      HHOOK SetWindowsHookEx(
         int idHook,       //钩子类型（WH_CBT）对窗口创建感兴趣
          HOOKPROC lpfn,   //钩子处理函数
          HINSTANCE hMod,  //应用程序句柄
          DWORD dwThreadId //线程ID
      );
      ~~~
      
      #### 钩子处理函数
      
      ~~~C++
      LRESULT CALLBACK CBTProc(
         int nCode,      //钩子码(HCBT_CREATEWND) 对应WH_CBT
         WPARAM wParam,  //刚刚创建成功的窗口句柄
         LPARAM lParam
      );
      ~~~
      
      ### MFC的窗口创建
      
      #### 更改窗口处理函数
      
      ~~~c++
      LONG_PTR SetWindowLongPtr(
        HWND hWnd,         
        int nIndex,            //GWLP_WNDPROC
        LONG_PTR dwNewLong     //新的窗口处理函数名(函数地址)
      );
      ~~~
      
      #### MFC窗口创建的过程
      
      ~~~c++
      CMyFrameWnd* pFrame = new CMyFrameWnd();
      PFrame->Create(NULL,"MFCBase");
      //在程序中我们只传递两个值，其他的参数都由默认值
      PFrame->Create(LPCTSTR　lpszClassName,
                    LPCTSTR lpszWindowName,
                    DWORD dwStyle,
                    const RECT& rect,
                    CWnd* pParentWnd,
                    LPCTSTR lpszMenuName,
                    DWORD dwExStyle
                    CCreateContext* pContext)
      {
        //首先进行加载菜单
          HMENU hMenu = NULL;
          if (lpszMenuName != NULL)
          {
             //加载菜单
          }
          //然后进入CreateEx
           //CreateEX内部执行见下面
          if (!CreateEx(dwExStyle,lpszClassName,lpszWindowName,dwStyle,.......,pParentWnd->GetSafeHwnd(),hMenu,(LPVOID)pContext))
          {
              //创建失败
              return FALSE;
          }
          return TRUE;
      }
   
   ​            **CreateEx内部执行**
   
   ~~~C++
   BOOL CWnd::CreateEx(...,lpszClassName(NULL),....)
         {
           //创建CREATESTRUCT的结构体并赋值   与win32相同
           CREATESTRUCT cs;
           ...
           cs.lpszClass = lpszClassName;(NULL) //窗口的名字为空无法创建窗口
           ...
           cs.hInstance = AfxGetInstanceHandle(); //该函数可以获得当前窗口的实例
           ...
           //通过PreCreateWindow(cs);对lpszClass赋默认值并且进行窗口注册
               //PreCreateWindow()内部执行在下面
           if(!PreCreateWindow(cs))
           {
              //注册失败
              PostNcDestroy();
              return FALSE;
           }
           //在上面对窗口类进行赋值时，将窗口的消息处理函数设置为DefWindowProc(默认的窗口处理函数),我们无法对消息进行处理，接下来，我们需要更改窗口的消息处理函数
          //更改窗口的消息处理函数，AfxHookWindowCreate(this);的执行过程在下面。并且将Frame对象存储到全局变量AFX_THREAD_STATE* pThreadState中
           AfxHookWindowCreate(this);//此处的this为Frame
       //接下来进行窗口的创建
       HWND hWnd = CreateWindowEx(...);
       //在窗口创建成功后就会产生WM_CREATE(窗口创建成功)的消息，此时就会上钩子函数获得该消息，并通过钩子处理函数改变窗口的消息处理函数
       //钩子处理函数(_AfxCbtFilterHook)的执行过程在下面
         }
   ~~~

**PreCreateWindow()内部执行**

~~~c++
BOOL CFrameWnd::PreCreateWindow(CREATESTRUCT& cs)
{
    if(cs.lpszClass == NULL)
    {
        //AfxDeferRegisterClass实现了窗口的注册功能
        //AfxDeferRegisterClass(AFX_WNDFRAMEORVIEM_REG)执行过程在下面
        VERIFY(AfxDeferRegisterClass(AFX_WNDFRAMEORVIEM_REG));
        //_afxWndFrameOrView是一个char类型的数组，值为"AfxFrameOrView140sud"，可以默认为lpszClass的默认名。
        cs.lpszClass = _afxWndFrameOrView;
    }
    return TRUE;
}
~~~

**AfxDeferRegisterClass(AFX_WNDFRAMEORVIEM_REG)执行过程**

~~~c++
BOOL AfxEndDeferRegisterClass(LONG fToRegister)
{
    //创捷窗口类的结构体
    WNDCLASS wndcls;
     //给窗口类赋值
    ...
    wndcls.lpfnWndProc = DefWindowProc; //将窗口的消息处理函数赋值为默认的窗口消息处理函数，那么我们如何进行窗口的消息处理呢？
    wndcls.hInstance = AfxGetInstanceHandle();
    ...
    wndcls.style = ...
    wndcls.hbrBackground = ...
       //_AfxRegisterWithIcon(&wndcls,_afxWndFrameOrView,AFX_IDI_STD_FRAME)该函数给wndcls的lpszClassName和hIcon赋值，并且对窗口进行注册。
        //_AfxRegisterWithIcon(&wndcls,_afxWndFrameOrView,AFX_IDI_STD_FRAME)的执行过程在下面
    if(_AfxRegisterWithIcon(&wncls,_afxWndFrameOrView,AFX_IDI_STD_FRAME))
    {
        ...
    }
}
~~~

**_AfxRegisterWithIcon(&wndcls,_afxWndFrameOrView,AFX_IDI_STD_FRAME)的执行过程**

~~~c++
_AfxRegisterWithIcon(&wndcls,_afxWndFrameOrView,AFX_IDI_STD_FRAME)
{
    //给窗口的名赋默认值
   wndcls.lpszClassName = _afxWndFrameOrView; //"AfxFrameOrView140sud"
    //加载图标
    //使用AfxRegisterClass(wndcls)对窗口进行注册，AfxRegisterClass(wndcls)的执行过程在下面
    return AfxRegisterClass(wndcls);
}
~~~

**AfxRegisterClass(wndcls)的执行过程**

~~~C++
BOOL AfxRegisterClass(wndcls)
{
    //使用RegisterClass进行注册窗口
   if(!RegisterClass(wndcls))
   {
       //注册失败
       return FAlSE;
   }
    return true;
}
~~~

**AfxHookWindowCreate(this)的执行过程**

~~~c++
void AFXAPI AfxHookWindowCreate(CWnd* pWnd)
{
   //获取第三个全局变量 _AFX_THREAD_STATE* pThreadState 当前线程状态信息
    _AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();
   
    if (pThreadState->m_hHookOldCbtFilter == NULL)
    {
        //设置窗口创建的钩子函数，其中_AfxCbtFilterHook是钩子的处理函数
        pThreadState->m_hHookOldCbtFilter = ::SetWindowsHookEx(WH_CBT,_AfxCbtFilterHook,NULL,::GetCurrentThreadId());
        //经Frame对象存储到全局变量 _AFX_THREAD_STATE* pThreadState中
        pThreadState->m_pWndInit = pWnd;
    }
}
~~~

**_AfxCbtFilterHook钩子处理函数执行过程如下**

~~~c++
_AfxCbtFilterHook(int code, WPARAM wParam(当前创建成功的窗口句柄),LPARAM lParam)
{
    //将Frame对象与窗口句柄进行绑定
    //获取全局变量_AFX_THREAD_STATE* pThreadState 当前线程状态信息
    _AFX_THREAD_STATE* pThreadState = _afxThreadState.GetData();
    //获取Frame对象
    CWnd* pWndInit = PThreadState->m_pWndInit;
    //获取窗口句柄
    HWND hWnd = (HWND)wParam;
    //将Frame对象与窗口句柄进行绑定 ,Attach的执行过程在下面
    pWndInit->Attach(hWnd);
    
    //更改窗口的消息处理函数
    WNDPROC afxWndProc = AfxGetAfxWndProc();
    //更改窗口消息处理函数为AfxWndProc
    oldWndProc = (WNDPROC)SetWindowLongPtr(hWnd, GWLP_WNDPROC,(DWORD_PTR)afxWndProc);
}
~~~

**Attach的执行过程**

~~~c++
BOOL CWnd::Attach(HWND hWndNew)
{
  //创建Map集合  afxMapHWND(true)的执行过程在下面
    CHandleMap* pMap = afxMapHWND(true);
  //将Frame中的窗口句柄与hWndNew绑定，双向绑定，即有Frame能找到句柄，有句柄能找到Frame,SetPermanent(this->m_hWnd = hWndNew,this)的执行过程在下面
    pMap->SetPermanent(this->m_hWnd = hWndNew,this);
}
~~~

**afxMapHWND(true)的执行过程**

~~~c++
CHandleMap* PASCAL afxMapHWND(true)
{
   //获取全局变量  AFX_MODULE_THREAD_STATE* pThreadState 当前程序模块线程状态信息 
    AFX_MODULE_THREAD_STATE8* pState = AfxGetModuleThreadState();
    //新建映射类对象，并保存到全局变量AFX_MODULE_THREAD_STATE* pThreadState的m_pmapHWND的成员中
    pState->m_pmapHWND = new CHandleMap(RUNTIME(CWnd),ConstructDestruct<CWnd>::Construct,ConstructDestruct<CWnd>::Destruct,offsetof(CWnd,m_hWnd));
    return pState->m_pmapHWND;
}
~~~

**SetPermanent(this->m_hWnd = hWndNew,this)的执行过程**

~~~c++
SetPermanent(this->m_hWnd = hWndNew,this)
{
  //实现由句柄得到Frame，将Frame存放到以句柄为索引的数组中，且该数组存放在全局变量AFX_MODULE_THREAD_STATE* pThreadState中
    m_permanentMap[句柄] = Frame;
}
~~~

总结：MFC创建窗口的步骤分为一下几步

1. 加载菜单

2. 调用CWnd::CreateEx函数创建窗口

   在CteateEx中调用PreCreateWindow函数设计和注册窗口类

   在PreCreateWindow函数中调用AfxDeferRegisterClass函数进行设计窗口类

   在AfxDeferRegisterClass内部

   ​    WNDCLASS wndcls；//设计窗口类

   ​    定义窗口的消息处理函数为DefWindowProc

   ​    调用_AfxRegisterWithIcon函数，处理图标，并调用AfxRegisterClass函数进行窗口注册，在AfxRegisterClass内部才真正到调用了win32的RegisterClass进行窗口的注册。

3. 调用AfxHookWindowCreate函数

    在AfxHookWindowCreate内部，调用SetWindowsHookEx创建WH_CBT类型的钩子，钩子的处理函数是_AfxCbtFilterHook。并且将框架类对象保存到全局变量当前啊程序线程信息中

4. 调用CreateWindowEx函数创建窗口，窗口创建成功后马上调用钩子的处理函数

5. 在钩子处理函数_AfxCbtFilterHook中将框架类对象与窗口句柄进行绑定和更改窗口的消息处理函数为AfxWndProc();

### MFC的消息处理

经过对MFC的窗口创建流程分析，我们可以知道，MFC将窗口的消息处理函数改为AfxWndProc(),那么MFC如果执行AfxWndProc()进行消息的处理呢？

为了能够更好的处理消息，我们需要在CMyFrameWnd中重写一下CFrameWnd中的WindowProc();具体代码如下：

~~~c++
LRESULT CMyFrameWnd::WindowProc (UINT msgID, WPARAM wParam, LPARAM lParam)
{
    switch (msgID)
    {
       case WM_ CREATE:
       {
		AfxMessageBox(_T("消息被处理了"));
       }break;
    }
    //调用父类的WindowProc
    return CFrameWnd::WindowProc(msgID, wParam, lParam);
}
~~~

使用调用堆栈找到AfxWndProc();

#### MFC消息处理的流程

以WM_CREATE消息为例子来解释消息处理的流程

~~~c++
LRESULT CALLBACK AfxWndProc(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)
{
    //通过窗口句柄获取Framed对象，通过Frame对象调用Frame的WindowProc函数进行窗口消息的处理
    //FromHandlePermanent(hWnd)的执行过程
    CWnd* pWnd = CWND::FromHandlePermanent(hWnd);
     //通过AfxCallWndProc将消息分发给不同的Frame的窗口消息处理函数，AfxCallWndProc的执行过程在下面
    return AfxCallWndProc(pWnd,hWnd,mMsg,wParam,lParam);
}
~~~

**FromHandlePermanent(hWnd)的执行过程**

~~~c++
CWnd* PASCAL CWnd::FromHandlePermanent(HWND hWnd)
{
       //获取保存窗口句柄与Frame对应的map集合 afxMapHWND()的执行过程在下面
       CHandleMap* pMap = afxMapHWND();
       //通过窗口句柄在map中获取Frame对象
       PWnd = (CWnd*)pMap->LookupPermanent(hWnd);
       //返回Frame对象
      return PWnd;
}
~~~

**afxMapHWND()的执行过程**

~~~c++
//再一次进入afxMapHWND()不会执行创建创建map的过程，直接通过全局变量 AFX_MODULE_THREAD_STATE 返回Frame
CHandleMap* PASCAL afxMapHWND(true)
{
   //获取全局变量  AFX_MODULE_THREAD_STATE* pThreadState 当前程序线程状态信息 
    AFX_MODULE_THREAD_STATE8* pState = AfxGetModuleThreadState()
    return pState->m_pmapHWND;
}
~~~

**AfxCallWndProc的执行过程**

~~~c++
LRESULT AFXAPI AfxCallWndProc(CWnd* pWnd, HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)
{
    //直接调用了Frame的窗口消息处理函数
    lResult = pWnd->WindowProc(nMsg,wParam,lParam);//回到自己的代码
}
~~~

总结：

1. 当收到消息时，进入AfxWinProc函数
2. AfxWndProc函数根据消息的窗口句柄吗，查询对应框架类Frame
3. 利用Frame调用Frame成员虚函数WindowProc,完成消息的处理

## MFC的消息映射机制

作用：在不重写WindowProc虚函数的大前提下，仍然可以处理消息

### 消息映射机制的使用

类必须具备的条件

​      类内必须添加声明宏   DECLARE_MESSAGE_MAP()

​      类外必须添加实现宏     BEGIN_MESSAGE_MAP(theClass,baseClass)

​                                           END_MESSAGE_MAP()

当一个类具备上述两个条件，这个类就可以按照消息映射机制来处理消息。

### 消息映射机制的具体实施

以WM_CREATE消息为例

1. 在  BEGIN_MESSAGE_MAP(theClass,baseClass)和END_MESSAGE_MAP()之间添加ON_MESSAGGE(WM_CREATE,OnCreate)的宏，其中OnCreate是自己定义的处理函数

2. 在CMyFrameWnd（继承CFrameWnd的类）类内添加OnCreate函数的声明和定义

   代码实现：

   ~~~C++
   class CMyFrameWnd :
   public CFrameWnd 
   {
      //通过消息机制进行消息处理
       DECLARE_MESSAGE_MAP()
       LRESULT OnCreate(WPARAM wParam, LPARAM lParam); 
   }
   
   
   BEGIN_MESSAGE_MAP(CMyFrameWnd, CFrameWnd)
   ON_MESSAGE(WM_CREATE, OnCreate)
   END_MESSAGE_MAP()
       
       
   LRESULT CMyFrameWnd::OnCreate(WPARAM wParam, LPARAM lParam)
   {
       AfxMessageBox(_T("消息被处理了"));
       return LRESULT();
   }
   ~~~

   为了能够更好的理解消息机制我们将宏代表的内容展开，代码变为

   ~~~c++
   class CMyFrameWnd:public CFrameWnd 
   {
      //通过消息机制进行消息处理
       
       //DECLARE_MESSAGE_MAP()展开的内容
   protected:
         static const AFX_MSGMAP* PASCAL GetThisMessageMap();
         virtual const AFX_MSGMAP* GetMessageMap() const;
   
         LRESULT OnCreate(WPARAM wParam, LPARAM lParam); 
   }
   
   
   //将BEGIN_MESSAGE_MAP(CMyFrameWnd,CFrameWnd) 展开的内容并且替换参数后的代码
   PTM_WARNING_DISABLE
   const AFX_MSGMAP* CMyFrameWnd::GetMessageMap() const
   {
      return GetThisMessageMap(); 
   }
   
   const AFX_MSGMAP* PASCAL CMyFrameWnd::GetThisMessageMap()
   {
      typedef CMyFrameWnd ThisClass;
      typedef CFrameWnd TheBaseClass;
      __pragma(warning(push))
      __pragma(warning(disab1e:4640)) /* message maps can only be called by single threaded message pump */
     static const AFX_MSGMAP_ENTRY _messageEntries[] =
      {
        //ON_MESSAGE(WM_CREATE, OnCreate)展开内容并且替换参数后的代码
       {WM_CREATE, 0, 0, 0, AfxSig_lwl, (AFX_PMSG) (AFX_PMSCW) (static_cast<LRESULT(AFX_MSG_CALL CWnd::*) (WPARAM, LPARAM)> (OnCreate))},
       //将END_MESSAGE_MAP() 展开的内容
       {0, 0, 0, 0, AfxSig_end, (AFX_PMSG)0 }
      };
     __pragma(warning(pop))
     static const AFX_MSGMAP messageMap = { &TheBaseC1ass::GetThisMessageMap, &_messageEntries[0]};
     return &messageMap;
   }
   PTM_WARNING_RESTORE
   ~~~

###   消息映射机制实现原理

   由上面的宏展开可以知道，类内的声明宏，声明了GetThisMessageMap()和GetMessageMap()。

   类为的实现宏，实现了GetThisMessageMap()和GetMessageMap()，且GetMessageMap()直接调用了GetThisMessageMap()。

为了更好的理解，先介绍一下AFX_MSGMAP_ENTRY和AFX_MSGMAP两个结构体

**AFX_MSGMAP_ENTRY**

~~~c++
struct AFX_MSGMAP_ENTRY
{
   UINT nMessage; //消息ID
   UINT nCode;  //control code or WM_ NOTIFY code
   UINT nID; // control ID (or 0 for windows messages) 控件ID
   UINT nLastID; //最后一个控件ID
   UINT_ PTR nSig;//处理消息的函数类型
   AFX_PMSG pfn; //处理消息的函数名
}
~~~

**AFX_MSGMAP**

~~~c++
struct AFX_MSGMAP
{
   const AFX_MSGMAP* (PASCAL* pfnGetBaseMap)(); //父类宏展开的静态变量地址
   const AFX_MSGMAP_ENTRY* lpEntries;           //本类宏展开的静态数组首地址
}
~~~

代码分析

在上面的代码展开的静态函数GetThisMessageMap()中，由两个静态的变量，分别是AFX_MSGMAP_ENTRY类型的数组和AFX_MSGMAP类型的变量。

其中AFX_MSGMAP_ENTRY类型的数组存放了和消息相关的内容。

AFX_MSGMAP类型的变量的第一个成员存放的是父类的AFX_MSGMAP的变量的地址，第二个成员存放的是AFX_MSGMAP_ENTRY类型的数组的首地址。

依次类推，CMyFrameWnd的父类也由这两个静态成员，其中AFX_MSGMAP的第一个成员存放了CMyFrameWnd的父类的父类的AFX_MSGMAP的变量的地址，第二个成员存放CMyFrameWnd的父类的AFX_MSGMAP_ENTRY类型的数组。

因此就形成了一个单向的链表

具体如下

![消息映射链](C:\Users\李福超\Desktop\消息映射链.png)

#### 宏展开各部分的作用

1. GetThisMessageMap() 静态函数

   作用：定义静态变量和静态数组，并返回本类静态变量的地址(获取链表头)

2. _messageEntries[] 静态数组

   作用：数组的每个元素，保存的为消息ID和处理消息的函数名(地址)

3. messageMap 静态变量

   作用：第一个成员，保存父类宏展开的静态变量地址（负责连接链表）

   ​           第二个成员，保存本类的静态数组首地址

4. GetMessageMap() 虚函数

    作用：返回本类静态变量地址(获取链表头)

#### 消息映射机制原理

通过程序运行的过程理解消息映射原理，通过程序调试找到OnCreate调用的源头AfxWindowProc。

程序运行过程,以WM_CREATE为例

~~~c++
LRESULT CALLBACK AfxWndProc(HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)
{
    //通过窗口句柄获取Framed对象，通过Frame对象调用Frame的WindowProc函数进行窗口消息的处理
    CWnd* pWnd = CWND::FromHandlePermanent(hWnd);
     //通过AfxCallWndProc将消息分发给不同的Frame的窗口消息处理函数，AfxCallWndProc的执行过程在下面
    return AfxCallWndProc(pWnd,hWnd,mMsg,wParam,lParam);
}
~~~

**AfxCallWndProc的执行过程**

~~~c++
LRESULT AFXAPI AfxCallWndProc(CWnd* pWnd, HWND hWnd, UINT nMsg, WPARAM wParam, LPARAM lParam)
{
    //直接调用了Frame的窗口消息处理函数
    lResult = pWnd->WindowProc(nMsg,wParam,lParam);//此时我们没有重写WindowProc(),所它会执行父类CWnd的WindowProc(),**CWnd::WindowProc**的执行过程在下面
}
~~~

**CWnd::WindowProc**的执行过程

~~~c++
LRESULT CWnd::WindowProc(UINT nMsg, WPARAM wParam, LPARAM lParam)
{
    //通过OnWndMsg对不同的消息进行分析,并且变量原型链表，OnWndMsg的执行过程在下面
    if(!OnWndMsg(nMsg,wParam,lParam,&lResult))
    {
       ......
    }
}
~~~

**OnWndMsg的执行过程**

~~~c++
BOOL CWnd::OnWndMsg(...)
{
    //获取链表头
    const AFX_MSGMAP* pMessageMap;
    pMessageMap = GetMessageMap();
    //定义AFX_MSGMAP_ENTRY* lpEntry的指针
    //循环遍历链表
    for( pMessageMap = GetMessageMap(); pMessageMap->pfnGetBaseMap != NULL; pMessageMap = (*pMessageMap->pfnGetBaseMap)())
    {
        if((lpEntry = AfxFindMessageEntry(pMessageMap->lpEntries,message,0,0)) != NULL)
        {
            //找到了与message匹配的AFX_MSGMAP_ENTRY的数组
            //跳出for循环，去执行AFX_MSGMAP_ENTRY的数组的OnCreate函数处理消息 LDisPatch的执行过程在下面
            goto LDisPatch;
        }
    }
    
}
~~~

**LDisPatch的执行**

~~~c++
LDisPatch：
         //获取OnCreate的地址
        mmf.pfn = lpEntry->pfn;
        //调用OnCreate函数
        lResult = (this->*mmf.pfn_l_W_1)(wParam,lParam);
~~~

总结：

1. 消息产生后进入窗口处理函数（AfxWndProc）
2. 根据已知窗口句柄，找到和它绑定在一起的框架类对象(Frame)
3. 利用框架类对象（Frame）调用框架类成员虚函数WindowProc,本类中未重写调用父类的WindowProc
4. 获取本类对应的静态变量，并找到对应数组中去匹配的消息Id
5. 如果未找到，获取父类对应的静态变量，并到对应的数字中去匹配查找
6. 如果找到了，利用找到的数组元素的最后一个成员（函数地址），并调用，完成消息处理。

## MFC消息分类

注意：在MFC中可以使用ON_MESSAGE()是一个通用的宏，可以向**AFX_MSGMAP_ENTRY**数组中放置任何种类的消息以及它的处理函数

但是在一般的开发中我们使用ON_MESSAGE的宏。

在MFC中不同种类的消息使用不同的消息宏和不同的消息处理函数。MFC的消息分类具体如下：

1. 标准Windows消息宏

     ON_WM_xxx            xxx代表不同的消息

   标准的Windows消息宏的不同消息具有固定且不同的消息处理函数，具体可以在msdn中查询

2. 自定消息宏

    ON_MESSAGE(消息ID，处理消息函数的名)

   处理消息函数的原型

   ~~~c++
   LRESULT OnCreate(WPARAM wParam, LPARAM lParam);
   ~~~

3. 命令消息宏

      ON_COMMAND(资源的ID,处理消息函数的名)

     处理消息函数的原型

       ~~~c++
       void OnButton();
       ~~~

   命令消息的处理顺序

## MFC的菜单

### 菜单相关

​     win32 -  HMENU       MFC -   CMENU

​      在MFC中菜单句柄要和菜单类形成一一对应的关系

### 菜单相关的类

#### CMenu类简介

 该类封装了菜单的各种操作的成员函数，另外还封装了一个非常重要的成员变量m_hMenu(菜单句柄)

### 菜单的使用使用

1. 添加菜单资源

2. 将菜单设置到窗口

    （1）利用pFrame调用Create函数时，传参

      （2） 在处理框架窗口的WM_CREATE消息时

   ​             CMenu menu；  

   ​            加载菜单  menu.LoadMenu(....);   //LoadMenu的功能时调用Win32的LoadMenu的函数 和 将菜单句柄与菜单类进行绑定

   ​           挂载菜单  Frame.SetMenu();          //该函数会调用Win32的SetMenu函数

   #### 菜单消息的处理（COMMAND）

## 工具栏

### 相关类

CToolBarCtrl  - 父类CWnd ,封装了关于工具栏控件的各种操作,该类代表工具栏中的控件

CToolBar  -父类CControlBar，封装了关于工具栏的操作，以及和框架窗口的关系。

### 工具栏的使用

1. 添加工具栏资源

2. 创建工具栏 CToolBar::CreateEx

3. 加载工具栏 CToolBar::LoadToolBar

4. 设置工具栏的停靠

      CToolBar::EnableDocking

      CFrameWnd::EnableDocking

   ​    CFrameWnd::DockCotrolBar

## MFC运行时类信息机制

### 运行时类信息机制的作用

在程序运行过程中可以获取对象的类的相关信息(例如：对象是否属于某个类)

### 运行时类信息机制的使用

1. 类必须派生自CObject

2. 类内必须添加声明 DECLARE_DYNAMIC(theclass)

3. 类外必须添加实现宏 IMPLEMENT_DYNAMIC(theclass,baseClass)

   当一个类具备上述三个要件后，CObject::IsKindOf函数就可以正确判断对象是否属于某个类

### 运行时类信息机制原理

MFC是如何通过IsKindOf就可以知道一个对象是否属于一个类呢？

为了进一步研究，我们将创建几个具有继承关系的类，并按照运行时类信息机制的使用声明这几个宏，最后将这几个宏进行展开

**宏展开的内容**

~~~c++
class CAnimal :pub1ic Cobject
{
//DECLARE_DYNAMIC(CAnima1)
//DECLARE_DYNAMIC(CAnima1)的展开
 public:
   static const CRuntimeClass classCAnimal;
   virtual CRunt imeClass* GetRuntimeClass() const;
} 
//类外实现宏
// IMPLEMENT_DYNAMIC(CAnimal, Cobject)的展开是IMPLEMENT_RUNTIMECLASS宏
 // IMPLEMENT_RUNTIMECLASS(CAnimal, Cobject, 0xFFF， NULL, NULL)
// IMPLEMENT_RUNTIMECLASS(c1ass_name, base_class_name,0xFFFF,NULL,NULL)展开
JAFX_ COMDAT const CRuntimeClass CAnimal: :classCAnimal = {
    "CAnimal",
    sizeof(class CAnima1),
    0xFFFF,
    NULL,
 // RUNTIME_CLASS(cobject)，
//此处将RUNTIMB_CLASS(CObject)展开为_RUNTIMECLASS(class_name)
//再将RUNTIME_CLASS(c1ass_name)宏展开为
   ((CRuntimeClass*)(&CObject::classC0bject)),
    NULL,
    NULL 
}
CRuntimeClass* CAnimal::GetRuntimeClass() const
{
  return RUNTIME_CLASS(CAnimal);
}

~~~

### CRuntimeClass数据结构简介

CRuntimeClass中不仅包含属性还包含一些函数，为了简化介绍，只介绍CRuntimeClass的属性

~~~c++
struct CRuntimeClass
{
   LPCSTR m_lpszClassName; //类名称
   int m_nObjectSize;      //类大小
   UINT m_wSchema;         //类版本(无用)
   CObject*(PASCAL* m_pfnCreateObject)(); //动态创建机制使用,这里为NULL
   CRuntimeClass* m_pBaseClass;   //父类宏展开静态变量地址
   CRuntimeclass* m_pNextClass; //不使用 为NULL
   const AFX_CLASSINIT* m_pClassInit; //不使用 为NULL
};
~~~

**一个非常重要的宏RUNTIME_CLASS(theClass)**

RUNTIME_CLASS(theClass) ----> &theClass::classtheclass; 获取thecClass这个类的静态变量classtheclass的地址

根据上面数据结构可知每一子类都会保存一个父类的静态变量，这样就会形成如下的根据继承关系形成的链表

![运算时类信息机制](C:\Users\李福超\Desktop\运算时类信息机制.png)

#### 运行时类信息机制的执行过程

~~~c++
dog.IsKinnOf(RUNTIME_CLASS(CAnimal))
{
    CRuntimeClass* pClassThis = GetRuntimeClass(); //调用虚函数，获取头节点
    //利用IsDeriveFrom进行链表的遍历，是否能够找到与pClass相同的类，IsDeriveFrom的具体执行在下面。
    return pClassThis->IsDeriveFrom(pClass);
}
~~~

**IsDeriveFrom的具体执行**

~~~C++
BOOL CRuntimeClass::IsDeriveFrom(pClass)
{
    const CRuntimeClass* pClassThis = this; //获取头节点
    //进行循环遍历
    while (pClass != NULL)
    {
        if (pClassThis == pClass)//判读与该父类对象是否相同
        {
            return true；
        }
        pClassThis = pClassThis->m_pBaseClass
    }
    return false;
}
~~~

总结：执行过程如下

1. 利用对象的地址调用宏展开的虚函数GetRuntimeClass(）获取本类静态变量的地址（链表头）
2. 利用本类静态变量地址（链表头）和目标进行比对
3. 如果相同，证明对象属于这个类
4. 如果不相同就会获取链表的下一个节点循环进行比对

## 动态创建机制

### 动态创建机制的作用

在不知道类名的情况下，将类的对象创建出来

### 动态创建机制的使用

类必须派生自CObject 

类内必须添加声明宏 DECLARE_DYNCREATE (theClass)

类外必须添加实现宏 IMPLEMENT_DYCREATE(theClass,baseClass)

但一个类具备上述三个要件后，CRuntimeClass::CreateObject(对象加工厂)函数就可以将类的对象创建出来。

### 动态创建机制的原理

为了能够及更好的理解动态创建机制的原理，将上面的全部展开为

~~~c++
class CDog :public CAnimal
{
    //动态创建类内声明宏
    DECLARE_DYNCREATE (CDog)
    //DECLARE_ DYNCREATE (CDog)展开为
         DECLARE_DYNAMIC(CDog)
         static Cobject* PASCAL CreateObject();
    //DECLARE DYNCREATE中包含了运行时类信息机制的类内声明宏，将DECLARE_ DYNAMIC (CDog) 展开为
    public:
     static const CRuntimeClass classCDog ;
     virtual CRuntimeClass* GetRuntimeC1ass() const;
}

//动态创建机制内外实现宏
IMPLEMENT_DYNCREATE(CDog,CAnima1)
// IMPLEMENT_DYNCREATE(CDog, CAnimal)展开为
CObject* PASCAL CDog::Create0bject()
{
  return new CDog;
}
IMPLEMENT_RUNTIMECLASS(CDog，CAnimal, 0xFFFF,CDog::Create0bject, NULL)
//与运行时类信息机制相同该宏内也包含了IMPLEMENT_RUNTIMECLASS
//将IMPLEMENT_RUNTIMECLASS展开为
AFX_COMDAT const CRuntimeC1ass CDog::classCDog = {
    "CDog",
    sizeof (class CDog),
    0xFFF,
    CDog::Create0bject,
    RUNTIMECLASS(CAnima1),
    NULL,
    NULL
};
CRuntimeClass* CDog::GetRuntimeC1ass() const
{
    return RUNTIMECLASS (CDog);
}
~~~

#### 动态创建机制与运行时类信息机制的区别

通过对动态创建机制和运行时类信息机制的宏展开可以得到动态创建机制与运行时类信息机制的区别：

1. 多了一个静态函数

   CObject* PASCAL CDog::Create0bject()
   {
     return new CDog;
   }

2. 静态变量的第四个成员不再为NULL，而为新增加的这个静态函数的地址CDog::Create0bject

#### 动态创建机制的执行过程

通过与运行时类信息机制对比，动态创建机制与运行时类信息机制相同都维护这一个与继承关系相对应的链表。由此可知动态创建机制的执行过程如下：

1. 利用本类(CDog)的静态变量CRuntimeClass调用其成员函数CreateObject（对象加工厂函数）
2. 获取静态变量的第四个成员(CreateObject();),并调用
3. 在CreateObject();的内部，完成对象的创建，并返回对象的地址。

## MFC视图窗口

### MFC视图窗口的作用

​     提供了一个用于显示数据的窗口

### 视图窗口的相关类

CView及其子类，父类为CWnd, 封装了关于视图窗口的各种操作，以及和文档类的数据交互

定义一个自己的视图类(CMyView)，派生自CView,并重写父类成员虚函数OnDraw(可以用于绘图)

其余框架类和应用程序类代码不变

在处理框架窗口的WM_CREATE消息时，定义cMyView类对象，并调用Create函数创建视图窗口  视图ID为 AFX_IDW_PANE_FIRST 

**视图窗口的创建过程与Frame的窗口的创建过程一致**

### OnDraw函数

使用OnDraw在视图窗口绘制图像，但是我们在Win32知道在绘制图像时需要处理绘图消息(WM_PAINT),那么OnDraw是如何绘制图像的呢？

首先我们现在CMyView中处理一下WM_PAINT消息(**一般在那个类的类的消息就在那个类中处理，即视图中的消息要在视图类中处理**)。

当我们在CMyView中处理了绘图消息后，OnDraw就不会执行了，这是为什么？

因为根据MFC的消息处理机制，MFC会维护一个关于消息的链表，在进行消息处理时，会首先获取链表头(现在为CMyView)，然后会循环遍历寻找WM_PAINT的消息处理函数，因为我们链表头放置了WM_PAINT的消息处理函数，就会执行此函数，然后退出循环。若我们没有处理WM_PAINT消息，肯定时父类帮助我们处理了该消息，并且在父类的消息处理函数中调用了OnDraw函数。

### theApp,pFrame,pView的关系图

theApp中保存着pFrame对象, pFrame对象中保存着pView对象

theApp ->m_pMainWnd = pFrame;

pFramem->pViewActive = pView   

## MFC的文档

### 文档类的作用

相关类： CDocument

  提供了一个用于管理数据的类，封装了关于数据的管理(数据提取、数据转换、数据存储等)，并和视图类进行数据交互。

### 文档类的使用

1. 定义一个自己的文档类(CMyDoc)，派生自CDocument
2. 其余具体见代码

​        新建CMyFrame、CMyView、CMyWinApp的类,其中CMyView具有动态创建机制

​       重写CMyWinApp的InitInstance()函数，其内容如下

```c++
  BOOL CMyWinApp: : InitIntance ()
  {
      CMyFrameWnd* pFrame = new CMyFrameWnd;
      // pFrame->Create (NULL,_T ("MFCDocument");
      //该函数内部调用了Create()
      CMyDoc* pDoc = new CMyDoc();
      CCreateContext cct;
      cct.m_pNewViewClass = RUNTIMECLASS(CMyView);
      cct.m_pCurrentDoc = pDoc;
      pFrame ->LoadFrame(IDR_MENU1,WS_OVERLAPPEDWINDOW,NULL,&cct);
      m_pMainWnd = pFrame;
      pFrame->ShowWindow(SW_SHOW);
      pFrame->UpdateWindow();
      return TRUE;
  }
```

### 新框架程序执行过程

#### 内容复习

WM_CREATE

产生时间：窗口创建后但未显示时。
附带消息：wParam:不使用    lParam：是CREATESTRUCT结构的指针，该指针包含了窗口的所有的信息，即::CreateWindowEx的全部12个参数。
一般用法：常用于初始化窗口的参数、资源等。

结合消息映射机制

ON_WM_CREATE() 处理WM_CREATE消息的函数为： int OnCreate(LPCREATESTRUCT pcs); 该函数为CREATESTRUCT的指针

#### 程序创建过程

1. 利用框架类对象pFrame调用LoadFrame函数，创建框架窗口。

2. pFrame的父类在处理框架窗口的WM_CREATE消息时,动态创建视图类对象，并创建视图窗口。

3. CMyView的父类在处理视图窗口的WM_CREATE消息时，将文档类对象和视图类对象建立关联关系。

   

   接下来通过伪代码的方式来验证上面的过程
   
   为了更好的调试代码，我们在程序的三个部分打断点：分别为  pFrame ->LoadFrame(IDR_ MENU1,WS_ OVERLAPPEDWINDOW,NULL,&cct);、框架类父类处理WM_CREATE的位置（return CFremaWnd::OnCreate）和视图类父类处理WM_CREATE的位置return CView::OnCreate
   
   首先进入  pFrame ->LoadFrame(IDR_ MENU1,WS_ OVERLAPPEDWINDOW,NULL,&cct);的断点处，查看窗口的执行过程：

~~~c++
CMyFrameWnd* pFrame = new CMyFrameWnd();
// pFrame->Create (NULL,_T("MFCDocument");
//该函数内部调用了Create()
CMyDoc* pDoc
new CMyDoc();
CCreateContext cct;
cct.m_pNewViewClass = RUNTIME_CLASS (CMyView);
cct.m_pCurrentDoc = pDoc;
pFrame->LoadFrame(IDR MENU1,WS_OVERLAPPEDWINDOW,NULL,&cct)
{
    //调用了Create函数进行了窗口的创建 Create内部执行过程（关注&cct的传递过程）的执行过程
    if (!Create(......,&cct)){
        return false;
    }
}

~~~

**Create内部执行过程（关注&cct的传递过程）的执行过程**

~~~C++
Create(......,&cct){
        //内部调用了CreateEx CreateEX内部执行过程
    CreateEx(.....,lParam:(LPVOID)cct){
        //给CREATESTRUCT结构体赋值
        .......
        cs.lpCreateParams = &cct;
        
        //调用win32API创建窗口，最后将cct传递给了CreateWindowEx的lParame的参数，我们在处理WM_CREATE消息时我们就可以获取并使用它
        //在初始化时我们将文档类对象地址和视图类的静态变量CRuntimeClass(通过它可以创建视图类的对象)
        ::CreateWindowEx(......,&cct)
    }
}
~~~



接下来我们继续进行到return CFremaWnd::OnCreate查看父类是如何创建视图类对象，并创建视图窗口

~~~c++
CFrameWnd::OnCreate(CREATESTRUCT lpcs)
{
   //由上面分析可知，lpcs中包含了CCreateContext cct;的内容
    //首先获取CCreateContext
    CCreateContext* pContext = (CCreateContext*)lpcs->lpreateParams;
    //对cct进行处理 OnCreateHelper的执行过程
    return OnCreateHelper(lpcs,pContext)
            {
                 //创建客户区 创建客户区的过程执行过程如下
                 OnCreateClient(lpcs,pContext)
                 {
                     //创建视图窗口 创建过程如下
                     CreateView(pContext,AFX_IDW_PANE_FIRST)
                     {
                         //创建视图对象
                         CWnd* pView = (CWnd*)pContext->m_pNewViewClass->CreateObject();       
                         //创建视图窗口 创建视图窗口的过程如下
                         pView->Create(NULL,NULL,AFX_WS_DEFAULT_VIEW,CRect(0,0,0,0),this,nID,pContext)
                         {
                             //此函数中又将pContext传递给pView的CreateEx的函数，在此函数的内部和pFrame中的CreateEx相同都调用了 ::CreateWindowEx并将cct传递给了lParam，即视图窗口的WM_CREATE的消息处理函数也能获取ctt,从而获取pDoc，并将pView与pDoc进行绑定，CrateEx的执行过程如下：
                            return CreateEx(......,pContext) 
                            {
                                 ::CreateWindowEx(......,pContext)
                            }
                         }
                     }
                 }
            }
}
~~~

接下来我们继续进行到returnreturn CView::OnCreate查看父类是如何创建视图类对象与文档类对象进行绑定

~~~c++
CView::OnCreate(CREATESTRUCT lpcs)
{
    //首先获取CCreateContext
    CCreateContext* pContext = (CCreateContext*)lpcs->lpreateParams;
    //获取文档类对象，并将视图类对象与文档类对象绑定 AddView的执行过程如下：
    pContext->m_pCurrentDoc->AddView(this)
    {
        m_viewList.AddTail(pView);//将pView存放到文档类的list的集合中，这里可以知道一个文档类可以对应多个视图类
        pView->pDocument = this(文档类); //在视图类的成员变量存放一个文档类对象，这里也可以知道一个视图类只能包含一个文档类
    }
    return 0;
}
~~~

### 四种对象的关系图

应用程序对象theApp->pMainWnd = pFrame(该成员变量保存着框架类对象)

框架类对象pFrame->m_pViewActive = pView(该成员变量保存着视图类对象)

视图类对象pView->m_pDocument = pDoc(该成员变量保存着文档类对象)

文档类对象pDoc->m_viewList(该成员变量保存着所有的视图类对象)

经过分析可知：一个文档类对象可以对应多个视图对象，一个视图对象只能对应一个文档类对象

思考：每个框架中可以包含多个视图，但是pFrame->m_ViewActive的成员只能保存一个视图对象，那么它保存是那个窗口对象？框架是如何和多个视图进行对应？

其实pFrame->m_ViewActive保存的活动的窗口，即鼠标点击的那个窗口。当系统处理鼠标点击消息时，系统会将该鼠标点击的视图对象保存到pFrame->m_ViewActive中，因此我们在程序初始化时可以人为的指定活动窗口即 pFrame->m_ViewActive = pView

### 命令消息处理顺序

经过实验可知。命令消息的处理过程顺序为 视图类 —》文档类——》框架类 ——》应用程序类 (默认顺序)

 为什么会有这样的顺序？接下来我们通过伪代码的方式进行解释一下，为了能够顺利的调试程序，我们只能在应用程序类中处理命令消息

**命令消息执行过程**

在CMyWinApp的WM_CREATE的消息处理函数OnCreate添加断点，通过调用堆栈找到最终调用为AfxWndProc,我们曾经根据WM_CREATE消息的处理过程来演示消息的处理过程，在WM_CREATE消息处理到OnMessage之前步骤是一致的，但是在OnMessage中WM_CREATE的处理过程与命令消息有了不同，因此我们将从OnMessage函数来研究命令消息的执行过程

**OnWndMsg中命令消息的处理**

~~~c++
BOOL CWnd::OnWndMsg(...)//该函数的内部的this是pFrame
{
    if(message == WM_COMMAND)
    {
        //利用OnCommand处理命令消息，OnCommand的执行过程在下面
        if(OnCommand(wParam,lParam))
        {
            lResult = 1;
            goto LReturnTrue;
        }
    }
    
}
~~~

**OnCommand的执行过程**

~~~~C++
OnCommand(wParam,lParam)//该函数的内部的this是pFrame
{
   //在该函数中调用了父类的CWnd的OnCommand,父类的OnCommand的执行过程如下
    return CWnd::OnCommand(wParam,lParam)
            {
  			  //该函数调用了OnCmdMsg(nID,nCode,NULL,NULL); OnCmdMsg的执行过程如下：
                OnCmdMsg(nID,nCode,NULL,NULL) //this是pFrame
                {
                   //获取活动窗口对象
                    CView* pView = GetActiveView();
                    //如果pView不为空就会调用pView->OnCmdMsg 如果pView没有找到命令消息的处理函数，pView就会执行pDoc的OnCmdMsg函数pView->OnCmdMsg的执行过程在下面。
                    if(PView != NULL && pView->OnCmdMsg)
                    {
                        return true；
                    }
                    //否则就会执行pFrame父类的OnCmdMsg函数,函数会遍历链表，来寻找命令消息的处理函数,因为该函数时pFrame，所以是遍历Frame支脉的链表
                    if(CWnd::OnCmdMsg()) return true;
                    //如果视图类中没用处理命令消息就会执行TheApp的OnCmdMsg函数
                    CWinApp* pApp = AFXGetApp();
                    if(pApp != NULL && pApp->OnCmdMsg) return true;
               return false;
                }
            }
}
~~~~

**pView->OnCmdMsg的执行过程**

~~~c++
pView->OnCmdMsg(...)
{
   //调用父类的OnCmdMsg ,函数会遍历链表，来寻找命令消息的处理函数,因为该函数时pView，所以是遍历View支脉的链表
    if(CWnd::OnCmdMsg(...))return true;
    //pView中没有命令消息处理函数 就会获取文档类对象
    if(this->m_pDocument != NULL)
    {
        return m_pDocument->OnCmdMsg();//里面调用了CWnd::OnCmdMsg(...)
    }
    
}
~~~

总结：

1. 消息产生后进入窗口处理函数（AfxWndProc）

2. 根据已知窗口句柄，找到和它绑定在一起的框架类对象(Frame)

3. 利用框架类对象（Frame）调用框架类成员虚函数WindowProc,本类中未重写调用父类的WindowProc

4. 在父类的WindowProc中调用了OnWndMsg,在OnWndMsg中的OnCommand处理了命令消息。

5. 进入pFrame->OnCommand()中调用了pFrame->OnWndMsg().在pFrame->OnWndMsg()中首先获取了pView,并执行了pView的OnWndMsg()。

6. 在pView->OnWndMsg中先执行CWnd::OnWndMsg,即在CView的支脉找查找命令消息的处理函数，如果未找到就会执行pDoc的OnWndMsg，然后其内部执行了CWnd::OnWndMsg.若未找到就退出函数

7. 接下来执行pFrame中的CWnd::OnWndMsg,若失败就会执行theApp的OnWndMsg

   因此总上所示，命令消息的执行顺序是通过函数调用顺序实现的

## 文档类与视图类的关系

 视图类成员函数

​         获取和视图类对象关联的文档类对象，调用GetDocument()

文档类成员函数

​        当文档类数据发生变化时，调用UpDataAllViews刷新和文档类对象相关联的视图类对象

​        GetFristViewPosition(),获取存放视图对象的链表的迭代器 通过迭代器GetNextView获取视图对象

## 窗口切分

窗口切分就是将两个或两个以上的窗口的对应一个文档类

### 相关类

CSplitterWnd -- 不规则框架窗口类，封装了关于不规则框架窗口的操作。

​                             规则框架 -- 一个框架只有一个客户区

​                             不规则框架 -- 一个框架有多个客户区

### 窗口切分的使用

重写CFrameWnd类的成员虚函数OnCreateClient

​       在虚函数中调用CSplitterWnd::CreateStatic创建不规则框架窗口。

​       在虚函数中调用CSplitterWnd::CreateView创建视图窗口

​      **在窗口切分中可以使用CSplitterWnd::GetPane来获取窗口中的视图对象**

## 单文档视图架构

特点：只能管理一个文档（只有一个文档类对象）

### 单文档视图架构的使用

 1.参与架构的类

​      CFrameWnd / CWinApp / CView / CDocument

2. 需要用到的类

   CDocTemplate(文档模板类) 的子类 CSingleDocument(单文档模板类)            

    CDocManager（文档管理类）

3. 参与架构的四个类除了应用程序类，其余的三个类均支持动态创建机制

4. 具体代码

   ~~~c++
   //重新重写了InitInstance,要想程序正确执行还添加一个菜单资源作为单文档模板类构造函数的第一个参数,添加字符串资源 ID为AFX_IDS_UNTITLED 值为 61443 标题随意
   B0OL CMyWinApp::InitInstance ()
   {
       //创建单文档模板类
       CSingleDocTemplate* pTemp1te = new CSingleDocTemplate(IDR_MENU1,RUNTIME_CLASS(CMyDoc),
                                          RUNTIME_CLASS(CMyFrameWnd),RUNTIME_CLASS(CMyView));
       AddDocTemplate(pTemp1te);
       OnFileNew();
       m_pMainWnd->ShowWindow(SW_SHOW);
       pMainWnd->UpdateWindow();
       return TRUE;
   }
   
   ~~~

   ### 单文档的执行过程

   我们将通过跟踪新重写的InitInstance来了解单文档的执行过程

​      1.跟踪 CSingleDocTemplate的构造

~~~C++
CSingleDocTemplate::CSing1eDocTemp1ate (UINT nIDResource,CRuntimeC1ass* pDocClass, CRuntimeC1ass* pFrameClass,CRuntimeClass* pViewClass)
: CDocTemplate (nIDResource, pDocClass, pFrameClass, pViewClass)
{
  m_ pOnlyDoc = NULL; //唯一文档类对象
    //该构造函数的构造主要是在父类CDocTemplate中
}
~~~

~~~C++
//父类CDocTemplate的构造
CDocTemplate::CDocTemplate (UINT nIDResource,CRuntimeClass* pDocClass, CRuntimeClass* pFrameClass,CRuntimeClass* pViewClass)
{
    //将pDoc、pFrameClass、pViewClass分别赋值给文档模板类的CDocTemplate的三个成员
    m_pDocClass = pDocClass;
    m_pFrameClass = pFrameClass;
    m_pViewClass = pViewClass;
}
~~~

总结：单文档模板类分别有三个成员m_ pOnlyDoc（保存唯一的文档类对象）、 m_pDocClass(保存文档类)、m_pFrameClass(保存框架类)、 m_pViewClass（保存视图类）

2. 跟踪AddDocTemplate

   ~~~c++
   void CWinApp::AddDocTemplate(CDocTemplate* pTemplate)//该函数的this指针为theApp
   {	
       if(m_pDocManager == NULL)
       m_pDocManager = new CDocManager;//创建CDocManager(文档管理类)并theApp的成员属性m_pDocManager中
       //执行文档管理类的AddDocTemplate(pTemplate),该函数的执行过程如下：
       m_pDocManager->AddDocTemplate(pTemplate);
                       {  //该函数内部是this是CDocManager
                           //将文档模板类放入CDocManager的模板链表中
   					 m_templateList.AddTail(pTemp1ate);
                       }
   }
   ~~~

总结：经过第二个步骤将theApp和模板类建立了关系，具体结构如下

​          theApp ——》m_pDocManager(文档类管理对象)  ——》m_templateList(保存了单文档类对象)  ——》CSingleDocTemplate（单文档类）

​          ——》（m_ pOnlyDoc（唯一的文档类）、m_pDocClass = pDocClass、 m_pFrameClass = pFrameClass、m_pViewClass = pViewClass;）

3. 重要的执行过程OnFileNew();

   ~~~c++
   void CWinApp::OnFileNew() //this为theApp
   {
     if(m_pDocManager != NULL)
         //调用CDocManager的OnFileNew,具体内容如下：
        m_pDocManager->OnFileNew();
                      {//该函数的this是CDocManager
                          //获取第一个单文档模板类
                          CDocTemplate* pTemplate = (CDocTemplate*) m_templateList.GetHead();
                          //执行单文档的OpenDocumentFile()，具体内容如下：
                          pTemplate->OpenDocumentFile(NULL);
                                     { //该函数的this为CSingleDocTemplate
                                         return OpenDocumentFile (lpszPathName,TRUE,bMakeVisible);
                                                { //该函数的this为CSingleDocTemplate
                                                    //创建新文档，内容如下：
                                                   pDocument = CreateNewDocument();
                                                               {
                                                                    //创建文档类对象
                                                                   CDocument* pDocument = (CDocument*)m_pDocC1ass- >Create0bject();
                                                                   //向单文档模板类中添加文档,其具体内容如下：
                                                                   AddDocument(pDocument);
                                                                   {//该函数的this为CSingleDocTemplate
                                                                       m_pOnlyDoc = pDocument;
                                                                   }
                                                                   return pDocument;
                                                               }
                                                         //创建框架类对象，参数为文档类 其内容如下
                                                    pFrame = CreateNewFrame(pDocument,NULL)
                                                             {//该函数的this为CSingleDocTemplate
                                                                //定义了CCreateContext结构体
                                                                CCreateContext context;
                                                                //context的参数赋值
                                                                ....
   														context.m_pCurrentDoc = pDocument;
                                                                 context.m_pNewViewClass = m_pViewClass; //( RUNTIME_CLASS(CMyView))
                                                                 ....
                                                                  //创建pFrame对象
                                                                 CFrameWnd* pFrame =(CFrameWnd*) m_pFrameClass->Create0bject();
   														 //执行LoadFrame，回到我们以前分析的代码 动态创建视图窗口，视图窗口创建成功后就                                                              //会执行CFrame::OnCreat()。在OnCreate中创建视图对象并将视图对象与文档对象绑定 
                                                                  pFrame->LoadFrame(m_nIDResource,WS_OVERLAPPEDWINDOW |                                                                                                     FWS_ADDTOTITLE,NULL,&context);
                                                                   return pFrame;
                                                             }
                                                    //返回文档类对象 
                                                   return pDocument;
                                                }
                                     }
   
                      }
   }
   ~~~

   ## 多文档视图架构

    多文档视图架构的特点： 可以管理多个文档（可以有多个文档类对象）

   ### 多文档视图架构的使用

   1. 参与架构的类

   ​     CMDIFrameWnd（主框架窗口类） / CMDIChildWnd（子框架窗口类） / CWinApp / CView / CDocument 

   2. 需要用到的类

      CDocTemplate (文档模板类)的子类  CMultiDocTemplate（多文档模板类）

      CDocManager (文档管理类)

   3. 参与架构的四个类除了应用程序类和主框架窗口类（ CMDIFrameWnd），其余的三个类均支持动态创建机制

   4. 具体代码

      ~~~c++
      //重新重写了InitInstance,要想程序正确执行还添加一个菜单资源作为单文档模板类构造函数的第一个参数,添加字符串资源 ID为AFX_IDS_UNTITLED 值为 61443 标题随意
      BOOL CMyWinApp::InitInstance()
      {
          CMyFrameWnd* pFrame = new CMyFrameWnd();
          pFrame->LoadFrame(IDR_ MENU1);
          m_pMainWnd = pFrame;
          pFrame->ShowWindow(SW_SHOW);
          pFrame->UpdateWindow();
          //创建子窗口
          CMultiDocTemplate* pTemp1ate = new CMu1tiDocTemplate(IDR_MENU3, RUNTIMECLASS(CMyDoc), RUNTIME_CLASS(CMyChi1d),
                                                               RUNTIMB CLASS (CMyView));
          AddDocTemplate(pTemp1ate);
          OnFileNew();
          OnFileNew();
          OnFileNew();
          return TRUE;
      }
      ~~~

      ### 多文档的执行过程

      1. 跟踪pFrame->LoadFrame(IDR_ MENU1)，创建主窗口

         ~~~c++
         pFrame->LoadFrame(IDR_ MENU1)
         {
             //调用了CFrameWnd::LoadFrame,CFrameWnd::LoadFrame的执行如下
           CFrameWnd::LoadFrame(nIDResource, dwDefaultStyle,pParentWnd, pContext)
           {   
               //调用了Create我想后面的步骤大都知道了
               Create(...)
               {
                   CreateEx(...)
                   {
                       //创建主窗口
                   }
               }
           }
         }
         ~~~

         2. 跟踪CMultiDocTemplate的构造过程

            ~~~c++
            CMultiDocTemplate::CMultiDocTemplate(UINT nIDResource, CRuntimeClass* pDocClass, CRuntimeClass* pFrameClass, CRuntimeClass* pViewClass):CDocTemplate (nIDResource, pDocClass,pFrameClass, pViewClass)
            {//该函数的this为CMultiDocTemplate
                
                //判断文档链表是否为空
                ASSERT(m_docList.IsEmpty());  //m_docList是CMultiDocTemplate的成员变量，保存着多个文档类对象
            }
            ~~~

            ~~~c++
            //父类CDocTemplate的构造
            CDocTemplate::CDocTemplate (UINT nIDResource,CRuntimeClass* pDocClass, CRuntimeClass* pFrameClass,CRuntimeClass* pViewClass)
            {
                //将pDoc、pFrameClass、pViewClass分别赋值给文档模板类的CDocTemplate的三个成员
                m_pDocClass = pDocClass;
                m_pFrameClass = pFrameClass;
                m_pViewClass = pViewClass;
            }
            ~~~

         3. 跟踪AddDocTemplate（与单文档相同）

            ~~~c++
            void CWinApp::AddDocTemplate(CDocTemplate* pTemplate)//该函数的this指针为theApp
            {	
                if(m_pDocManager == NULL)
                m_pDocManager = new CDocManager;//创建CDocManager(文档管理类)并theApp的成员属性m_pDocManager中
                //执行文档管理类的AddDocTemplate(pTemplate),该函数的执行过程如下：
                m_pDocManager->AddDocTemplate(pTemplate);
                                {  //该函数内部是this是CDocManager
                                    //将文档模板类放入CDocManager的模板链表中
            					 m_templateList.AddTail(pTemp1ate);
                                }
            }
            ~~~

         4. 跟踪OnFileNew();

            ~~~c++
            void CWinApp::OnFileNew() //this为theApp
            {
              if(m_pDocManager != NULL)
               //调用CDocManager的OnFileNew,具体内容如下：
                 m_pDocManager->OnFileNew();
                             {//该函数的this是CDocManager
                                //获取第一个多文档模板类
                                CDocTemplate* pTemplate = (CDocTemplate*) m_templateList.GetHead();
                                //执行多文档的OpenDocumentFile()，具体内容如下：
                                   pTemplate->OpenDocumentFile(NULL);
                                           { //该函数的this为CMultiDocTemplate
                                                  return OpenDocumentFile (lpszPathName,TRUE,bMakeVisible);
                                                         { //该函数的this为CMultiDocTemplate
                                                             //创建新文档，内容如下：
                                                            pDocument = CreateNewDocument();
                                                             {
                                                                //创建文档类对象
                                                                 CDocument* pDocument = (CDocument*)m_pDocC1ass->Create0bject();
                                                                  //向多文档模板类中添加文档,其具体内容如下：
                                                                   AddDocument(pDocument);
                                                                     {//该函数的this为CSingleDocTemplate
                                                                        m_docList.AddTail(pDocument);
                                                                      }
                                                                     return pDocument;
                                                                }
                                                            //创建框架类对象，参数为文档类 其内容如下
                                                             pFrame = CreateNewFrame(pDocument,NULL)
                                                                {//该函数的this为CSingleDocTemplate
                                                                   //定义了CCreateContext结构体
                                                                    CCreateContext context;
                                                                    //context的参数赋值
                                                                     ....
            												   context.m_pCurrentDoc = pDocument;
                                                                    context.m_pNewViewClass = m_pViewClass;//(RUNTIME_CLASS(CMyView))
                                                                      ....
                                                                      //创建pFrame对象
                                                                   CFrameWnd* pFrame =(CFrameWnd*) m_pFrameClass->Create0bject();
            									           //执行LoadFrame，回到我们以前分析的代码 动态创建视图窗口，视图窗口创建成功后就                                                    //会执行CFrame::OnCreat()。在OnCreate中创建视图对象并将视图对象与文档对象绑定 
                                                                   pFrame->LoadFrame(m_nIDResource,
                                                                                 WS_OVERLAPPEDWINDOW | FWS_ADDTOTITLE,NULL,&context);
                                                                      return pFrame;
                                                                  }
                                                             //返回文档类对象 
                                                            return pDocument;
                                                         }
                                              }
            
                               }
            }
            ~~~

            ## MFC序列化机制

             作用： 以二进制流形式读写硬盘文件。但效率高。
            
            ### 文件操作相关类
            
            1. CFile - 文件操作类，封装了关于文件读写等操作
            
                  CFile::Open
            
                  CFile::Write/Read
            
                  CFile::Close
            
                  CFile::SeekToBegin / SeekToEnd / Seek
            
            ### 序列化相关的类
            
            ​     CFile - 文件操作类， 完成硬盘文件的读写操作
            
            ​     CArchive - 归档类，完成内存数据的读写操作， 该类一般会维护一个缓冲区，内存中的数据首先写入缓存区，再写入磁盘
            
            ### 序列化的使用（序列化）
            
               1. 创建或打开文件 CFile::Open
            
               2. 定义归档类对象  CArchive ar；
            
               3. 数据序列化（存储/写） ar << 数据
            
               4. 关闭归档类对象  ar.Close()
            
               5. 关闭文件  CFile::Close();
            
                  代码实现：
            
                  ~~~c++
                  void Store()//序列化
                  {
                      CFile file;
                      LPCTSTR str = _T("E:/text/serial. txt");
                      file.Open (str,CFile::modeCreate | CFile::modeWrite);
                      CArchive ar(&file, CArchive::store, 4096);
                      long age = 18;
                      ar << age;
                      float score = 88.5;
                      ar << score;
                      CString str1 = _T( "Hello world");
                      ar << str1;
                      ar.Close();
                      file.Close();
                  }
                  
                  ~~~
            
            ### 反序列化机制的使用
            
            1. 打开文件 CFile::Open 
            2. 定义归档类对象  CArchive ar；
            3. 数据反序列化（加载/读）  ar >> 变量
            4. 关闭归档类对象 ar.Close()
            5. 关闭文件  CFile::Close()
         
         ​             代码实现
         
         ```C++
         
         void Load()//反序列化
         {
             CFile file;
             LPCTSTR str = _T("E:/text/serial.txt");
             file.Open(str,CFile::modeRead);
             CArchive ar(&file, CArchive::load, 4096) ;
             1ong age;
             ar >> age;
             float score;
             ar >> score;
             CString str1;
             ar >> str1;
             ar.Close();	
             file.Close();
             cout<<age<<" "<<score <<" "<< str1 <<endl;
         }
         ```
         
         ### 序列化的执行机制
         
           CArchive的结构
         
         ~~~c++
         class CArchive
         {
             enum Mode{store = 0, load = 1,....};
             bool m_nMode; //访问方式
             int m_nBufSize; //buff的大小
             CFile* m_pFile; //操作文件的对象
             BYTE* m_lpBufCur; //当前指向
             BYTE* m_lpBufMax; //终止指向
             BYTE* m_lpBufStart; //开始指向
             ...
         }
         ~~~
         
         #### 序列化执行过程
         
         ~~~c++
         
          CFile file;
          LPCTSTR str = _T("E:/text/serial.txt");
          file.Open(str,CFile::modeRead);
           //CArchive的构造
           CArchive ar(&file, CArchive::store, 4096);
                     {
                          m_nMode = nMode; //0
                          m_pFile = pFile; //E:/text/serial.txt
                          m_nBufSize = 4096 // 4K
                          m_lpBufStart = new BYTE[m_nBufSize];
                          m_lpBufMax = m_lpBufStart +  m_nBufSize;
                          m_lpBufCur = (IsLoading()) ? m_lpBufMax : m_lpBufSart
                            //IsLoaid的内部指执行为： return (m_nMode & CArchive::loade != 0)  
                     } 
          long age = 8;
         // ar << age;的执行过程 将18保存到当前指向的位置，并向后移动当前指向相应字节数
          ar << age;
              {
                  //重载函数
                  CArchive& CArchive::operator<<(LONG l)
                  {
                      // 超过缓冲区长度就进行刷新
                      if (m_lpBufCur + sizeof(LONG l) > m_lpBufMax)
                      {
                          Fluse();
                              {
                                  //写入文件
                                  m_pFile -> Write()
                                  m_lpBufCur = m_lpBufStart;//重置当前指向
                              }
                      }
                      //将变量保存到当前指向
                      *m_lpBufCur = l;
                      //移动当前指向
                      m_lpBufCur += sizeof(LONG);
                  }
              }
             float score = 88.5;
             ar << score; //与上面的long相同
             CString str1 = _T( "Hello world");
             //ar << str1;执行过程
             ar << str1;
                {
                     CArchive& CArchive::operator<<(str)
                     {  
                         //写字符串长度 ,执行过程
                         AfxWriteStringLength(*this,str.GetLength,sizeof(BaseType) == sizeof(wchar_t));
                             {
                                 ar << nLength; //写入字符串长度 （str.GetLength）
                             }
                         //写字符串 ，执行过程
                         Write(str,str.GetLength*sizeof(BaseType));
                               {       
                                     //拷贝数据
                                   Checked::memcpy_s(m_lpBufCur,(size_t)(m_lpBufMax-m_lpBufCur),str,str.GetLength);
                                   //移动当前指向
                                   m_lpBufCur += str.GetLength
                               }
                     }
                }
          // ar.Close();的内部执行
          ar.Close();
             {
                 //Flush的内部执行
                 Flush();
                 {
                    //写入文件
                    m_pFile -> Write(m_lpBufStart,ULONG(m_lpBufCur - m_lpBufStart)); 
                    m_lpBufCur = m_lpBufStart;//重置当前指向
                 }
             }
         
         ~~~
         
         总结： 
         
         1. ar对象维护一个缓冲区
         2. 将各个数据一次序列化（存储）到ar对象维护的缓冲区中，并将m_lpBufCur的指针指向移动相应字节
         3. 如果ar维护的缓冲区不足，则将ar维护的缓冲区的数据写入硬盘文件，并重置m_lpBufCur为开始指向
         4. 当关闭ar对象时，将ar对象维护的缓冲区写入硬盘文件，并释放ar对象维护的缓冲区
         
         #### 反序列化执行过程
         
         反序列化与序列化过程相反。执行过程为：
         
         1. ar对象维护一个缓冲区
         2. 当反序列化第一个数据时，将文件数据全部读取到ar维护的缓冲区中，并将第一个数据反序列化到第一个变量，并将m_lpBufCur移动相应的字节数
         3. 依次反序列化每个数据到变量中。
         4. 当关闭ar对象时，释放ar维护的缓冲区
      
      ### 序列化类对象
      
        序列化对象要求对象满足一下四个条件
      
      1. 类必须派生自CObject
      
      2. 类内必须添加声明宏 DECLARE_SERIAL(theClass)
      
      3. 类外必须添加实现宏 IMPLEMENT_SERIAL(theClass,baseClass,1)
      
      4. 类必须重写虚函数 Serialize
      
         如序列化CMyDoc类 如下
      
         ~~~c++
         //类声明
         class CMyDoc:public CDocument
         {
                   DECLARE_SERIAL(CMyDoc)
                   //将DECLARE_SERIAl宏展开为
                     DECLARE_DYNCREATE (class_name)//动态机制创建宏
                       // >> 重载函数
                     AFX_API friend CArchive& AFXAPI operator>> CArchive& ar,CMyDoc*& pob);
         
             public: //重写
                   virtual void Serialize (CArchive& ar) ;
             public://构造
                  CMyDoc(){}
                  CMyDoc(CString str, int age, int score):m_str(str), m_age(age), m_score(score)
             public:
             CString m_str;
             int m_age;
             float m_score;
         };
         //类实现
         #include "MyDoc.h"
         IMPLEMENT_SERIAL(CMyDoc, CDocument, 1)
          //IMPLEMENT_SERIAL(CMyDoc, CDocument, 1)展开为
           Cobject* PASCAL CMyDoc::Create0bject()	
         	{ 
                return new CMyDoc;
         	}
         extern AFX_CLASSINIT init_CMyDoc;
         _IMPLEMENT_RUNTIMECLASS(CMyDoc,CDocument, wSchema, CMyDoc::Create0bject, &_init_CMyDoc) 
         AFX_CLASSINIT init_CMyDoc (RUNTIME_CLASS(CMyDoc)); 
         //以上为动态创建机制实现宏的展开
         // >> 重载函数实现
         CArchive& AFXAPI operator>> (CArchive& ar,CMyDoc* &p0b)
         { 
              pob = (CMyDoce*)ar.ReadObject(RUNTIME_CLASS(CMyDoc));
              return ar;
         }
         //以上为IMPLEMENT_SERIAL展开内容
         
         
         void CMyDoc::Serialize(CArchive& ar)
         {
             if (ar.IsStoring())
             {
               ar<<mstr<<m_age<<mscore;  
             }
             else
             {
               ar>>mstr>>m_age>>mscore;
             }
         }
         
         //序列化MyDoc
         void Store0bject()//序列化对象
         {
             CFile file;
             LPCTSTR str = _T("E:/text/serialObject.txt");
             file.Open(str,CFile::modeCreate | CFile::modeWrite);
             CArchive ar (&file, CArchive::store, 4096);
             CMyDoc doc( _T("he1lo world"), 18, 82.5);
             ar << &doc;
             ar.Close() ;
             file.Close() ;
         }
         
         //反序列化MyDoc
         void LoadObject()
         {
              CFile file;
             LPCTSTR str = _T("E:/text/serial.txt");
             file.Open(str,CFile::modeRead);
             CArchive ar(&file, CArchive::load, 4096);
             CMyDoc* doc = NULL;
             ar >> doc;
             ar.Close() ;
             file.Close();
             cout<<doc->m_age<<" "<<doc->m_score <<" "<<doc->m_str <<endl;
         }
         
         
         ~~~
      
         对象序列化执行过程如下
      
         ~~~C++
         //序列化对象
         CFile file;
         file.Open("E:/text/serialObject.txt", CFile::modeCreate|CFile::modeWrite);
         CArchive ar(&file, CArchive::store, 4096);//归档类对象,维护缓冲区。
         CMyDoc doc(_T("he1lo world"), 18, 82.5);
         //ar << 内部执行过程
         ar << &doc; 
             {	
               operator<< (ar, const &doc)
               {
                   ar.WriteObject(&doc)//函数内部this为&ar
                   {
                     CRuntimeClass* pClassRef = &data->GetRuntimeClass();//文档类静态变量
                     WriteClass(pClassRef);//将类的相关信息（类名/类大小/类版本）存入ar维护的buff中
                     (&data)->Serialize(ar)//函数内部this为&data
                         {
                           ar << this->m_age << this->m_score << this->m_name; //序列化基本类型变量
                         }
                   }
               }
             }
         
         
         //反序列化对象
         CFile file;
         file.Open( "E:/text/serialObject.txt", CFile::modeRead );
         CArchive ar( &file, CArchive::load, 4096 );//维护一个buff，大小4096字节
         CMyDoc* doc = NULL;
         ar >> doc
         {
              operator>>(ar, doc) 
              {
                   doc = ar.ReadObject(RUNTIME_CLASS(CMyDoc))//函数内部this为&ar
                   {
                     CRuntimeClass* pClassRef = ReadClass(RUNTIME_CLASS(CMyDoc),...);
                            //从文件读取 类的相关信息，和 RUNTIME_CLASS(CMyDoc)中信息进行比对，
                            //如果相同返回RUNTIME_CLASS(CMyDoc),如果不同返回NULL
                     CObject*pOb = RUNTIME_CLASS(CMyDoc)->CreateObject();
                            //动态创建CMyDoc类的对象，并返回对象地址
                     pOb->Serialize(ar)//函数内部this为刚刚创建的CMyDoc类对象（pOb）
                     {
                       ar >> m_age >> m_score >> m_str;//反序列化基本类型变量
                     }
                     return pOb;
                   }
              }
         }
         ~~~
      
         ## MFC的对话框架构
         
         对话框分类
         
         ​          模式对话框（使用无模式对话框模拟的）/  无模式对话框
         
         ### 无模式对话架构使用
         
           参与架构的类
         
         ​     CDialog/ CWinApp
         
          使用：
         
            1. 添加对话框资源
         
            2. 定义一个自己的对话框类(CMyDlg),管理对话框资源，派生自CDialog或CDialogEx均可
         
               代码书写
         
               ~~~c++
               #include <afxwin.h>
               #include"resource.h"
               class CMyDlg : public CDialog
               {
                   enum{IDD=IDD_DIALOG1};
                   CMyDlg:CMyDlg()
                   {
                       
                   }
               }
               class CMyWinApp : pub1ic CWinApp
               {
               public:
                 virtual B0OL InitInstance();
               }
               BOOL CMyWinApp::InitInstance()
               {
                   CMyDlg* dlg = new CMyDlg();
                   dlg->Create(IDD_DIALOG1);
                   m_pMainWnd = dlg;
                   dlg->ShowWindow(SW_SHOW);
                   return TRUE;
                }
               CMyWinApp theApp;
               ~~~
         
               ### 执行过程
         
               1. 当程序启动时，构造theApp对象，调用父类CWInApp的构造函数。 主要功能是将theApp对象保存到两个MFC的全局变量中。
               2.  首先获取应用程序类对象theApp，
               3. 利用theApp调用InitApplication，初始化当前应用程序的数据
               4. 利用theApp调用InitInstance初始化程序，在函数中我们创建窗口并显示
               5. 利用theApp调用CWinApp的Run函数进行消息循环
               6.  如果没用消息，利用theApp调用OnIdle虚函数进行空闲处理
               7. 当对话框销毁（必须利用DestroyWindow),消息循环才能退出
               8. 程序退出利用theApp地址调用ExitInstance虚函数实现退出前的善后处理工作
         
               ### 模式对话框
         
               参与架构的类
         
               ​     CDialog/ CWinApp
         
                使用：
         
                       1. 添加对话框资源
                       2. 定义一个自己的对话框类(CMyDlg),管理对话框资源，派生自CDialog或CDialogEx均可
         
               ​     代码书写
         
               ```c++
               #include <afxwin.h>
               #include"resource.h"
               class CMyDlg : public CDialog
               {
               class CMyWinApp : pub1ic CWinApp
               {
               public:
                 virtual B0OL InitInstance();
               }
               BOOL CMyWinApp::InitInstance()
               {
                   CMyDlg dlg; //栈空间
                   m_pMainWnd = &dlg;
                   dlg.DoModal();
                   return FALSE;
                }
               CMyWinApp theApp;
               
               ```
               
               #### 执行过程
               
               1. 首先获取应用程序类对象theApp，
               
               2. 利用theApp调用InitApplication，初始化当前应用程序的数据
               
               3. 利用theApp调用InitInstance初始化程序，在函数中调用DoModal函数
               
               4. DoModal内部执行过程 
               
                         1. 将父窗口设置为不可用
                         2. 创建无模式对话框
                         3. 进入消息寻循环（自带的）
                         4. 退出消息循环（父类的OnOk/OnCancel导致循环退出）
                         5. 将父窗口设置为可用状态
                         6. 销毁无模式对话框
               
               5. 不再执行CWinApp的Run函数的消息循环。
               
               6. 程序结束、
               
         
         ## 对象和控件的绑定
         
            作用：
         
          将控件窗口和类对象绑定具有两大作用
         
            1. 如果和数据类对象(管理数据的类 ，例如：CString )绑定，对象和控件可以进行数据交换(可以将控件中的数据保存到对象中，也可以将对象信息保存到控件中)。
         
            2. 如果和控件类对象(例如 CButton )绑定，对象就可以代表整个控件
         
               
         
               ### 与数据类型对象进行绑定的使用
         
                数据类型对象和控件可以是实现数据交互
         
                  重写父类成员虚函数DoDataExchange
         
               ​            在函数内部通过一系列的DDX_xxx函数，实现控件和数据类型对象的数据交互。(类向导可以自动的添加对应的DDX_xxx函数)
         
               如果需要实现数据交互，调用 UpdateData函数
         
               ​    UpdateData(TRUE): 控件 ->变量
         
               ​    UpdateData(FALSE): 变量 ->控件
         
         ### 与控件类型对象绑定的使用
         
            控件类型对象和控件可实现对像代表整个控件
         
         ​    重写父类成员虚函数DoDataExchange
         
         ​          在函数内部通过一系列的DDX_xxx函数，实现控件句柄和控件类型对象的绑定。(类向导可以自动的添加对应的DDX_xxx函数)
         
         ​    控件类型对象，就代表这个控件
         
         ## 控件消息的处理
         
         ​     控件消息的处理
         
         ​            WM_COMMAND消息
         
         ​                   LOWORD(wParam) -- 菜单项ID，控件ID
         
         ​                   HIWORD(wParam) --菜单项为0
         
         ​                                                      控件，通知码(控件发生的事情)
         
         ​                   lParam -- 均无用
         
         ​      控件处理消息需要放置消息的专职宏 例如 ON_BN_CLICKED(IDC_BUTTON1,OnButton1) ，不要使用ON_COMMAND();宏
         
         ## 基本控件
         
         ​    学习一个控件需要掌握的几个方面
         
                     1. 控件如何 与 数据类型对象绑定
                     2. 控件如何 与 控件对象绑定
                     3. 控件的消息如何处理。
         
         ### 下压式按钮
         
         
         
         
      
      
      
      
      
      
      
      
      
         

