### Binder机制

Binder：Android系统进程间通信（IPC）方式之一

#### Linux的传统IPC

管道|system V IPC（消息队列/共享内存/信号量）|socket

1. 只有socket支持C-S通信方式，但传输效率低、开销大，需要两次拷贝，主要用于跨网络IPC和本机低速IPC；
2. 消息队列|管道采用存储-转发方式，至少有两次拷贝过程；
3. 共享内存无需拷贝，但控制复杂，难以使用。

4. 传统IPC存在安全性隐患
   1. 传统IPC没有任何安全措施，完全依赖上层协议。
   2. 接收方无法获得对方进程可靠UID|PID，无法鉴别身份。Android为每个App分配UID，故进程UID是鉴别身份的标志，但传统IPC只能由用户手动填入UID|PID，易被恶意程序篡改。可靠的身份标记只有通过IPC本身在内核中添加。
   3. 传统IPC访问接入点是开放的，无法建立私有通道。如管道名称、socket的IP或文件名都是开放的，无法阻拦恶意程序。

Binder基于C-S通信模式，传输过程只需一次拷贝，为发送方添加UID|PID身份，支持实名|匿名Binder，安全性高。

#### Binder IPC

1. Binder IPC使用C-S通信方式：一个进程作为Server提供诸如音频/视频解码、地址查询、网络连接等服务；多个进程作为Client向Server请求服务。Binder可看作Server提供的实现某具体服务的访问接入点，Client通过该地址向Server发送请求；Binder可看作Client通往Server的管道入口，通过该入口与Server建立通信。
2. 面向对象：Binder是一个实体位于Server中的对象，提供一套方法实现对服务的请求；Client通过Binder的引用访问Server。面向对象将IPC转化为通过Binder对象引用调用方法，Binder对象是一个可跨进程引用的对象，实体位于一个进程中，但引用遍布系统各进程。

#### Binder模型

Binder框架包含四个角色：Server、Client、ServiceManager(SMgr)、Binder驱动。Server|Client|SMgr位于用户空间，驱动运行于内核空间。SMgr类似于DNS，Binder驱动类似于路由器。

1. Binder驱动

   Binder驱动与硬件设备无关，工作于内核态，提供标准文件操作，负责进程之间Binder通信的建立，Binder在进程间的传递，Binder引用计数管理，数据包在进程间的传递和交互等一系列底层支持。

2. ServiceManager

   SMgr将字符形式的Binder名转化为Client中的引用。注册了名字的Binder叫实名Binder，Server创建Binder及名字并通过BInder驱动传递给SMgr注册实名Binder，驱动为该实名Binder创建位于内核中的实体节点，将名字及实体引用传递给SMgr中填入查找表。
