### 1. const int*, const int * const, int const
> ref: https://stackoverflow.com/questions/1143262/what-is-the-difference-between-const-int-const-int-const-and-int-const

### 2. regular expression
> ref: https://blog.csdn.net/zengxiantao1994/article/details/77816972

### 3. stack vs heap
* code segment
* data segment(数据段): 存放程序中已初始化且不为0的全局/静态变量的一块内存区域
* bss segment(BSS段): 存放程序中未初始化的全局/静态变量的一块内存区域
* stack: local variable, method parameter
* heap：new, malloc, calloc, realloc
> ref: https://blog.csdn.net/billcyj/article/details/78783741

### 4. process vs thread
* 一个程序至少有一个进程,一个进程至少有一个线程
* 一个进程崩溃后, 在保护模式下不会对其它进程产生影响; 一个线程死掉就等于整个进程死掉
* 地址空间为每个进程所私有的; 线程有自己的堆栈和局部变量, 但线程共享进程的地址空间(线程之间没有单独的地址空间)
    * 速度：线程产生的速度快, 线程间的通讯快、切换快等, 因为他们在同一个地址空间内
    * 资源利用率：线程的资源利用率比较好也是因为他们在同一个地址空间内
    * 同步问题：线程使用公共变量/内存时需要使用同步机制还是因为他们在同一个地址空间内
* 线程执行开销小, 但不利于资源管理和保护；而进程正相反, 进程切换时, 耗费资源较大
* 总线程数 <= CPU数量: 并行运行; 总线程数 > CPU数量: 并发运行 
> ref: https://blog.csdn.net/mxsgoden/article/details/8821936

### 5. 加密算法
* 对称密钥: DES、RC4,RC5, AES
* 非对称密钥: RSA
* 哈希算法: MD5, SHA-1，SHA-2(SHA-256，SHA-512)
> ref: https://www.jianshu.com/p/bf1d7eee28d0

### 6. 调用约定
* stdcall: 1) 参数从右向左压入堆栈; 2) 函数自身修改堆栈; 3) 函数名自动加前导的下划线, 后面紧跟一个@符号, 其后紧跟着参数的尺寸
* cdecl: 1) 参数从右向左压入堆栈; 2) 调用者负责清理堆栈
* fastcall: 1) 函数的第一个和第二个DWORD参数（或者尺寸更小的）通过ecx和edx传递, 其他参数通过从右向左的顺序压栈; 2) 被调用函数清理堆栈; 3) 函数名修改规则同stdcall
* thiscall: 1) 参数从右向左入栈; 2) 如果参数个数确定, this指针通过ecx传递给被调用者; 如果参数个数不确定, this指针在所有参数压栈后被压入堆栈; 3) 对参数个数不定的, 调用者清理堆栈, 否则函数自己清理堆栈
> ref: https://blog.csdn.net/u010852680/article/details/78316633

### 7. 锁
* mutex（互斥锁): 
    * 同一时间, 锁只有一把, 如果线程A加锁正在访问资源, 这时B尝试加锁, 就会阻塞;
    * 不加锁也可以访问数据 eg. 线程A加锁了正在访问资源, 这时B不加锁也可以直接访问数据

```
// pthread_mutex_t变量类型是一个结构体, 可以简单理解成整数, 只有1或者0两个取值, 初始值为0
int pthread_mutex_init(pthread_mutex_t* mutex, const pthread_mutexattr_t* attr); 
int pthread_mutex_destroy(pthread_mutex_t* mutex);
int pthread_mutex_lock(pthread_mutex_t* mutex); // mutex--
int pthread_mutex_trylock(pthread_mutex_t* mutex); 
int pthread_mutex_unlock(pthread_mutex_t* mutex); // mutex++
// 当解锁成功时, 该函数会把阻塞在该锁上的所有线程全部唤醒, 之后哪个线程抢到锁继续执行, 就是调度优先级的问题了, 默认是先阻塞的先唤醒
```

* 读写锁: 写独占, 读共享; 写锁优先级高; 读写锁特别适用于读的次数远大于写的情况
    * 当读写锁是读模式加锁时, 其它线程以读模式加锁都会成功, 但是线程以写模式加锁会阻塞;
    * 当读写锁是写模式加锁时, 直到解锁前，其它线程加锁都会被阻塞;
    * 当读写锁是读模式加锁时, 其它线程既有试图以写模式加锁的线程，也有试图以读模式加锁的线程, 这时读写锁会阻塞在写模式加锁请求之后的读模式加锁请求, 优先满足写模式
    
```
int pthread_rwlock_init(pthread_rwlock_t* rwlock, const pthread_rwlockattr_t* attr);
// attr：表示读写锁属性，通常使用默认属性，传NULL就行了
int pthread_rwlock_destroy(pthread_rwlock_t* rwlock);
int pthread_rwlock_rdlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_wrlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_unlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_tryrdlock(pthread_rwlock_t* rwlock);
int pthread_rwlock_trywrlock(pthread_rwlock_t* rwlock);
```

* 死锁
* 条件变量

```
int pthread_cond_init(pthread_cond_t *restrict cond, const pthread_condattr_t *restrict attr); 
int pthread_cond_destroy(pthread_cond_t *restrict cond);
int pthread_cond_wait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex);
int pthread_cond_timewait(pthread_cond_t *restrict cond, pthread_mutex_t *restrict mutex, const struct timespec *restirct abstime);
int pthread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthread_cond_t *cond);
```

* 信号量: 
    * 互斥锁在同一时刻只能有一个线程访问加锁的数据, 但是信号量可以允许多个线程来访问
    * 互斥量的加锁和解锁必须由同一线程分别对应使用, 信号量可以由一个线程释放, 另一个线程得到
    * 基于上面的特点, 互斥锁一般用于控制一段临界代码，当然信号量也可以做到; 但是如果释放和获取不在一个函数中甚至不在一个线程中时就必须使用信号量了,

```
int sem_init(sem_t *sem, int pshared, unsigned int value)

int sem_init(sem_t *sem, int pshared, unsigned int value) // pshared:取0代表用于线程间，取非0代表用于进程间； value:该参数指定信号量初值
int sem_destroy(sem_t *sem)
int sem_wait(sem_t *sem) // == sem--
int sem_post(sem_t *sem) // == sem++
int sem_trywait(sem_t *sem)
int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout)
```

* 信号量vs互斥锁
    * 信号量：对一个进程加锁, 可以不断加锁, 设置一个标记a == 0, a++; 解锁的时候a--, 当a == 0时可以继续进行; 应用场景：生产者-消费者模型; 使用场景: 操作系统分配多个打印任务时
    * 互斥锁：当一个进程把持资源时, 其他进程不能访问此资源, 此特性代表了此资源一次只能被一个进程利用, 使用场景: 文本的写入
    
### 8. TCP UDP
> ref: https://www.quora.com/What-is-the-difference-between-HTTP-protocol-and-TCP-protocol

* TCP网络编程中connect()、listen()和accept()
> ref: https://blog.csdn.net/lianghe_work/article/details/46443691

* TCP建立连接／断开连接状态
Client: CLOSED->SYN_SENT->ESTABLISHED->FIN_WAIT_1->FIN_WAIT_2->TIME_WAIT->CLOSED
Server: CLOSED->LISTEN->SYN_RCVD->ESTABLISHED->CLOSE_WAIT->LAST_ACK->CLOSED
> ref: https://blog.csdn.net/Mary19920410/article/details/63711522

### 9. TCP的流量控制 拥塞控制
* 流量控制: 滑动窗口

* 拥塞控制: 慢开始、拥塞避免、快重传、快恢复
    * 增加缓存空间到一定程度时, 只会加重拥塞, 而不是减轻拥塞, 因为当数据包经过长期时间排队完成转发时, 他们可能早已经超时, 从而引起源端超时重发; 增加链路宽带及提高处理能力也不能解决拥塞问题
    * 缓存空间不足导致的丢包更多的是拥塞"症状"而非原因
    * 拥塞只是一个动态问题, 我们没有办法用一个静态方案去解决, 从这个意义上说, 拥塞时不可避免的; 拥塞是一个全局控制的过程, 他不像点对点的流控机制, 是一个局部的控制

* 慢开始: cwnd *= 2
    * 发送方维持着一个拥塞窗口cwnd(congestion window)
    * cwnd初始化为1个最大报文段(MSS)大小
    * cwnd的值就随着网络往返时间(Round trip time，RTT)呈指数级增长(cwnd *= 2)
    * 如果宽带为W，那么经过RTT*log2W时间就可以占满带宽
* 拥塞避免: cwnd += 1
    * 慢启动门限（ssthresh）
    * 当cwnd < ssthresh时, 使用慢开始算法。
    * 当cwnd > ssthresh时, 改用拥塞避免算法。(cwnd += 1)
    * 当cwnd = ssthresh时, 慢开始与拥塞避免算法任意。
* 快重传
    * 发送方只要一连收到三个重复确认就应当立即重传对方尚未收到的报文段, 而不必继续等待设置重传计时器时间到期。
* 快恢复
    * 当发送方连续收到三个重复确认时, 把ssthresh门限减半(ssthresh = cwnd / 2), 并直接进入拥塞避免算法(cwnd = ssthresh)
* 发生超时重传: 网络拥塞的主要依据是它重传了一个报文段
    * 把ssthresh降低为cwnd值的一半(ssthresh = cwnd / 2);
    * 把cwnd重新设置为1(cwnd = 1);
    * 重新进入慢启动过程;
> ref: http://www.voidcn.com/article/p-vrdkquop-ms.html

### 10. 一维 二维
```
int a[] = {1, 2, 3, 4, 5, 6, 7, 8};
int b[8];
int* c = new int[8];

cout << sizeof(a)/sizeof(a[0]) << endl; // 8
cout << sizeof(b)/sizeof(b[0]) << endl; // 8
cout << sizeof(c)/sizeof(c[0]) << endl; // 2
cout << sizeof(*c)/sizeof(c[0]) << endl; // 1
```

```
int a[2][2]={1, 2, 3}; // true
int a[2][2]={{1}, {2}};
int a[2][2]={{1}, 2, 3};
int a[3][2]={{1}, 2, 3, {4}};
int a[3][2]={{1}, 2, {4}}; // false

int a[][2]={1, 2, 3}; // true
int a[][2]={{1}, {2}}; 
int a[][2]={{1}, 2, 3};
int a[][2]={{1}, 2, 3, {4}};
int a[][2]={{1}, 2, {4}}; // false

int a[2][]={{1,2},{3,4}} // false
// *(a[2] + 1) == *(*(a + 2) + 1) == a[2][1]
```

```
int a[][2]={{1}, 2, 3, {4}};
// 相当于int ** a

cout << &a[0][0] << endl;
cout << *a << endl; // address of a[0][0], 步长1
cout << a << endl; // address of a[0][0], 步长2
cout << &a << endl; // address of a[0][0], 步长6

cout << *a + 1 << endl; // address of a[0][1]
cout << &a[0][1] << endl;

cout << a + 1 << endl; // address of a[0][2]
cout << &a[0][2] << endl;

cout << &a + 1 << endl; // address of after a[2][1]
cout << &a[2][1] << endl; 

int* p = (int*)(&a + 1);
cout << *(p - 2) << endl; // 4
```

```
int* p[3]; // 指针数组

cout << p << endl;
cout << p + 1 << endl; // 步长: 1 size: 8

cout << &p << endl;
cout << &p + 1 << endl; // 步长: 3 size: 8

cout << *p << endl;
cout << *p + 1 << endl; // 步长: 1 size: 4

cout << **p << endl;
cout << **p + 1 << endl; // value

int (*p)[3]; // 数组指针 行指针
cout << p << endl;
cout << p + 1 << endl; // 步长: 3 size: 4

cout << *p << endl;
cout << *p + 1 << endl; // 步长: 1 size: 4

cout << **p << endl;
cout << **p + 1 << endl; // value
```
* 数组名作为形参传入函数时，退化为指针

### 11. KMP
> ref: http://www.ruanyifeng.com/blog/2013/05/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm.html

### 12. 函数参数入栈顺序
* 参数入栈时顺序从右向左入栈
* 入栈前先把参数列表里的表达式算一遍得到表达式的结果, 最后再把这些运算结果统一入栈
* ++运算符直接从变量a所在的内存地址中取值

```
void f(int a, int b, int c, int d) {
    cout << a << endl; 
    cout << b << endl; 
    cout << c << endl; 
    cout << d << endl; 
}

int main(){
    int i = 0;
    f(i++, ++i, i, i++); // 2 3 3 0
}
```
> ref: https://blog.csdn.net/xidiancoder/article/details/49160317

### 13. 父子进程
* 子进程从父进程继承了: 
    * 当前工作目录, 根目录
    * 环境, 堆栈, 共享内存
    * 打开文件的描述符
    * 信号掩码
    * nice值 (由nice函数设定，数值越小，优先级越高)>   
* 子进程与父进程不同的: 
    * 地址空间
    * 进程号PID, 各自的父进程号
    * 子进程不继承父进程的进程正文, 数据和其它锁定内存
    * 不继承异步输入和输出
> ref: https://blog.csdn.net/carl_wu_/article/details/77976020 https://blog.csdn.net/koudaidai/article/details/8014782

### 14. sizeof struct/sizeof class
* 每个结构体成员的起始地址为该成员大小的整数倍, 即int型成员的起始地址只能为0、4、8等
> ref: https://blog.csdn.net/RadianceBlau/article/details/60867307

* 遵循结构体的对齐原则
* 类的大小与普通数据成员有关, 与成员函数和静态成员无关: 即普通成员函数, 静态成员函数, 静态数据成员, 静态常量数据成员均对类的大小无影响
* 虚函数对类的大小有影响, 是因为虚函数表指针带来的影响
    * 虚函数的类或实例最后的sizeof是实际的数据成员的sizeof加8(需要字节对齐)
    * 多继承: 每个基类都有自己的虚表; 子类的成员函数被放到了第一个基类的表中
* 虚继承对类的大小有影响, 是因为虚基表指针带来的影响
    * 子类有指向虚基类的指针
* 空类的大小为1
    * 当子类继承空类后, 子类如果有自己的数据成员, 则空基类的一个字节并不会加到子类中去
    * 一个类包含一个空类对象数据成员, 空类的1字节是会被计算进去(需要字节对齐)
> ref: https://blog.csdn.net/fengxinlinux/article/details/72836199

### 15. 扇区 簇 块 页
* 扇区：扇区是磁盘最小的物理存储单元, 硬盘不是一次读写一个字节而是一次读写一个扇区（512个字节）
* 簇：系统读读写文件的基本单位, 一般为2的n次方个扇区(由文件系统决定), 每个簇或者块可以包括2、4、8、16、32、64…2的n次方个扇区; 操作系统规定一个簇中只能放置一个文件的内容
* 原因: 读取方便, 由于扇区的数量比较小, 数目众多在寻址时比较困难, 所以操作系统就将相邻的扇区组合在一起, 形成一个块, 再对块进行整体的操作
* 块/页: 与内存操作，是虚拟一个页的概念来作为最小单位。与硬盘打交道，就是以块为最小单位。
> ref: https://blog.csdn.net/qq_34228570/article/details/80209748

### 16. 进程状态
> ref: https://blog.csdn.net/qicheng777/article/details/77427157

### 17. 初始化列表
```
class A
{
public:
    A() {cout<<"A"<<endl;}
    ~A() {cout<<"~A"<<endl;}
    
    A(const A& a) // 拷贝构造函数
    {
        cout << "Copy constructor for A" << endl ;
    }

    A& operator = (const A& a) // 赋值运算符
    {
        cout << "assignment for A" << endl ;
        return *this;
    }
};

class C
{
public:
    C() {cout<<"C"<<endl;}
    ~C() {cout<<"~C"<<endl;}
};

class D
{
public:
    D() {cout<<"D"<<endl;}
    ~D() {cout<<"~D"<<endl;}
    
    D(const D& d) // 拷贝构造函数
    {
        cout << "Copy constructor for D" << endl ;
    }

    D& operator = (const D& d) // 赋值运算符
    {
        cout << "assignment for D" << endl ;
        return *this;
    }
};

class B : public A
{
public:
    B(D& d, A& a, C& c) : _c(c),  _a(a)
    { 
        _d = d;
        cout<<"B"<<endl; 
    }
    ~B() { cout<<"~B"<<endl; }
private:
    D _d;
    A _a;
    C _c;
};

int main()
{
    A a;
    C c;
    D d;
    B b(d, a, c);
    return 0;
}
```
> ref: https://www.cnblogs.com/graphics/archive/2010/07/04/1770900.html

### 18. copy constructor
```
class A 
{
public:
    int i;
};

class B 
{
A *p;
public:

    /*
    B(const B& b){
        p = b.p;
    }*/ 
    // 编译器在生成default copy construction的时候使用的bitwise copy语义，也就是只是简单的浅拷贝
    
    B(const B& b) // 拷贝构造函数
    {
        p = new A;
        p->i = b.p->i; 
        // *p = *(b.p);
        // memcpy(p, b.p, sizeof(A));
        cout << "Copy constructor for B" << endl ;
    }
    B(){
        cout << "B" << endl ;
        p = new A;
    }
    ~B(){
        cout << "~B" << endl ;
        delete p;
    }
};

void f(B b){
}

int main(){
   B b;
   f(b);
}
```

### 19. Copy constructor, Move constructor, Move assignment
* 移动构造函数和移动赋值运算符重载函数不会隐式声明, 必须自己定义
* 如果用户自定义了拷贝构造函数或者移动构造函数, 那么默认的构造函数将不会隐式定义, 如果用户需要, 也需要显式的定义
* 移动构造函数不会覆盖隐式的拷贝构造函数
* 移动赋值运算符重载函数不会覆盖隐式的赋值运算符重载函数
* A move constructor is called:
    * when an object initializer is std::move(something)
    * when an object initializer is a temporary 当对象初始化程序是临时的
    * when returning a function-local class object by value 当按值返回函数局部类对象时
```
class A
{ 
public:
    A(){
        cout << "A" << endl ;
    }
    
    virtual ~A(){
        cout << "~A" << endl ;
    }
    
    A& operator=(const A& a) // 赋值运算符
    {
        cout << "assignment for A" << endl ;
        return *this;
    }
    
    A(const A& a) // 拷贝构造函数
    {
        cout << "Copy constructor for A" << endl ;
    }
    
    A(A&& a)
    {
        cout << "Move constructor for A" << endl;
    }
    
    A& operator=(A&& a) {
        cout << "Move assignment for A" << endl;
        return *this;
    }
};

/* A f(A a) // copy //
{  
    A a2(a); // copy // copy
    A a3 = a2; // copy // copy 
    return a3; // //
    // ~A for a, a2 // ~A for a, a2
}

int main(){
    A a; // A
    A a2 = f(f(a)); // 
    // ~A for a, a2
} */

A RetByValue() {
    A a;
    return a; // Might call move ctor, or no ctor.
}

void TakeByValue(A a){};

int main() {
    A a1;
    A a2 = a1; // copy ctor
    A a3 = std::move(a1); // move ctor
    TakeByValue(std::move(a2)); // Might call move ctor, or no ctor.
    A a4 = RetByValue(); // Might call move ctor, or no ctor.
    a1 = RetByValue(); // Calls move assignment, a::operator=(a&&)
}
```
> ref: https://stackoverflow.com/questions/13125632/when-does-move-constructor-get-called

### 20. #define typedef
```
#define INT_PTR int*
typedef int* int_ptr;
INT_PTR a1, a2;
int_ptr b1 ,b2;
int *c1, c2;

cout << sizeof(a1) << endl;
cout << sizeof(a2) << endl;
cout << sizeof(b1) << endl;
cout << sizeof(b2) << endl;
cout << sizeof(c1) << endl;
cout << sizeof(c2) << endl;
```
> ref: https://stackoverflow.com/questions/3263252/is-typedef-just-a-string-replacement-in-code-or-somethings-else

### 21. InterruptedException异常
当一个方法后面声明可能会抛出InterruptedException 异常时，说明该方法是可能会花一点时间，但是可以取消的方法: wait sleep join

> ref: https://blog.csdn.net/derekjiang/article/details/4845757

### 22. 
```
#include <iostream>

using namespace std;

class Animal
{
public:
    virtual void f()
    {
        cout<<"Animal"<<endl;
    }
};
  
class Cat : public Animal
{
public:
    int mNum = 1;
    void f() override
    {
        cout<<"Cat"<<endl;
    }
};
  
int main()
{
    Animal* a = new Animal(); 
    cout << sizeof(*a) << endl; // 8
    a->f(); // Animal
    
    Cat* c = (Cat*)(a);
    cout << sizeof(*c) << endl; // 16
    cout << c->mNum << endl; // not 1
    c->f(); // Animal
    delete a;
  
    c = new Cat();
    cout << sizeof(*c) << endl; // 16
    a = c;
    cout << sizeof(*a) << endl; // 8
    a->f(); // Cat
    // cout << a->mNum << endl; // error
    cout << c->mNum << endl; // 1
    delete c;
    
    return 0;
}

```
