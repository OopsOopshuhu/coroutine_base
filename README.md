# MYFiber

#### 介绍
C++实现轻量级的协程

#### 软件架构
epoll 调度器


#### 安装教程

1.  xxxx
2.  xxxx
3.  xxxx

#### 使用说明
协程实现两种方式
* stack_for --- 每个协程有独立栈
* stack_nees --- 共享栈空间
1.  ucontext_t                           --- 上下文结构体定义
    ```cpp
    // linux 5.4 内核中，ucontext_t 定义
    typedef struct ucontext_t
    {
        unsigned long int __ctx(uc_flags);
        struct ucontext_t *uc_link; // 下一个context，如果置NULL，表示执行完后进程退出
        stack_t uc_stack; // 当前上下文使用栈，包括起始地址和大小
        mcontext_t uc_mcontext; // 实际保存上下文的变量，不用管
        sigset_t uc_sigmask; // 阻塞信号集合
        struct _libc_fpstate __fpregs_mem;
    } ucontext_t
    ```
2.  getcontext/setcontext                --- 实现for循环
    ```cpp
    int getcontext(ucontext_t *ucp) 
    // 获取当前上下文初始化ucp，初始化内容包括CPU寄存器，信号mask和当前栈空间。成功返回0,失败返-1
    
    int setcontext(const ucontext_t *ucp) 
    // 恢复上下文到CPU执行
    // 注意：set成功之后调用set不会有返回值，因为函数已经放弃CPU，CPU去执行别的函数了
    // ucp 上下文由 getcontext 和 makecontext 设置而来
    ```
3.  makecontext/(setcontext/swapcontext) --- 实现函数调用
    ```cpp
    void makecontext(ucontext_t *ucp, void (*func)(), int argc, ...);
    // 修改上下文信息，设置上下文入口函数，ucp由 getcontext 初始化。
    // ucp 由谁恢复，传递给哪个函数
    // 执行后需要为新上下文分配一个栈空间，如果不创建，那么新函数func执行时会使用旧上下文的栈，而这个栈可能已经不存在了。argc 必须和 func 中整型参数的个数相等。
    ```
4.  swapcontext
    ```cpp
    int swapcontext(ucontext_t *oucp, ucontext_t *ucp);
    // 切换上下文---保存上下文到 oucp, 重置 ucp 上下文由 getcontext 和 makecontext 设置而来
    // 成功返回 0，失败返回 -1 并置 errno。如果 ucp 所指向的上下文没有足够的栈空间以执行余下的过程，将返回 -1
    ```
5.  ucontext簇函数                        --- 实现函数循环调用

#### 难点

CPU切换实现---需要在用户程序中实现调度切换
1. 嵌入汇编实现
2. setjmp/longjmp
3. ucontext簇函数
