# kilim疑问解答




## 1. Kilim中的Task，即用户线程如何调度和切换？ 

相比传统的Thread多线程间抢占式调度，Kilim中的Task采用的是协作式调度，即由Task本身负责释放和恢复占用CPU
在多任务的调度上操作系统采取抢占式和协作式两种方式。

## 2.Kilim如何识别线程堆栈中哪些方法是Pauseable，即可暂停的？

Kilim通过代码编译期识别抛出的Pauseable异常注解，来判断识别方法可暂停

## 3.Kilim是如何实现线程执行过程中当前方法的暂停和恢复？

Kilim通过编译期字节码编织，对每一个可暂停的方法进行字节码处理，在方法执行前和执行后加上相关的执行上下文的处理，暂停时会保存整个线程堆栈，包括完整栈帧，程序计数器等等，然后通过特定的字节码跳转指令jsr跳转到另外一个Task的执行方法中，恢复时将复原整个线程堆栈，包括完整栈帧，程序计数器等等，回到上次暂停时的指令地址处继续执行。

## 4.Kilim中的Weaver**工具是如何针对编译的代码实现织入的**？

字节码技术，具体来说通过ASM字节码框架实现对class文件的重写

## 5.如何将一个传统的线程执行方法改造成Kilim的Task模型？

Kilim通过编译期字节码编织，对每一个可暂停的方法进行字节码处理，在方法执行前和执行后加上相关的执行上下文的处理：

- 首先需要实现一个类继承Task，实现Task的execute方法，业务逻辑不再放在线程的run方法体体，而是放在Task的execute方法体中。
- execute方法中调用的方法如果是可能暂停的，则必须声明抛出Pauseable异常，否则可以不需要抛出。
- Task之间的通信通过Mailbox邮箱来传递消息，put和get时存在三种版本，包括阻塞线程，阻塞Task但不阻塞线程，非阻塞。
- 针对这些Task和Pauseable方法编译时，需要使用Kilim提供的Weaver工具进行编织处理，如果不进行编织处理，运行时将会异常。
- 对于Kilim中的方法需要注意，一个Pauseable方法只能被另一个Pauseable方法调用。

## 6.  Kilim中哪些操作可以使得Task暂停或者恢复运行？

Kilim中常用有以下方法暂停、恢复Task的运行：

- Task.sleep()能使当前Task暂停运行一段时间
- Task.yield()能使当前Task暂停放弃运行
- Task.pause()能使当前Task暂停执行
- Task.resume()能使当前Task恢复执行
- Mailbox.get()能使当前Task暂停执行直到Mailbox队列非空。

## 7.  Kilim中Task占用的内存有多大？

一个Task所占用的内存包括以下部分：

- Task的具体实现类实例本身占用的内存
- Task之间通信依赖的Maibox的内容占用的内存
- 如果Task暂停，那么Task函数调用链上的函数栈帧数组需要保存到Fiber中，不过Kilim的Weaver工具在代码编译期间将分析代码控制流程、有用的变量、常量等，保证只保存在后续Task恢复执行时需要用到的数据。

## 8.  Kilim中Fiber的作用

Fiber主要作用用来管理和保存Task执行过程中调用层次中的函数栈帧的状态，这里的函数栈帧与JVM运行时中的函数栈帧是相同含义，但是Fiber不会将函数栈帧中的全部信息原封不动的镜像拷贝一份，比如局部变量表中的所有变量，而是经过代码分析之后有选择的暂存有必要保留的变量，一般只需要保存后续执行流程中需要用到的变量，例如静态常量等就无需保存到Fiber中，因为静态变量可以直接通过iconst之类的字节码直接加载到操作数栈。

## 9.  Kilim中Fiber中的pc的真正含义？

Fiber中的pc，字面意义是指程序计数器，实际含义是：如果pc值为0，则表示第一次开始执行，程序执行流程和字节码增强前的流程是一样的；如果pc值为N，则表示直接跳转至本函数中第N个Pauseable方法处开始执行，说明之前执行到第N个Pauseable方法时暂停了，此时Task恢复执行，字节码层面通过tableswitch指令将直接跳转该Pauseable方法处执行，也即再次进入该函数执行体。以此类推，整个函数调用链均按照这种逻辑流转。

```java
public void function() throw Pauseable

{

    XXX;  // 临时变量等初始化

    A();  // function A is pauseable，如果执行到函数A 暂停了，则pc=1

    B();  // function B is pauseable，如果执行到函数B暂停了，则pc=2，下次恢复时从function()函数入口直接跳转到这里，执行函数B

    C();  // function C is pauseable，如果执行到函数B暂停了，则pc=3，下次恢复时从function()函数入口直接跳转到这里，执行函数C  

}
```

## 10. Kilim中Fiber中State的作用？

Fiber中的State作用主要体现在curState和stateStack两个变量，它们用来维护函数调用链执行过程中的函数栈帧。

当Task将要执行某个Pauseable方法时，将首先调用Fiber的down方法，来记录当前执行到整个函数调用链中的下一层次，并记录curState和pc。

当Task在执行某个Pauseable方法过程中暂停时，内部会调用Task的pause方法，而pause直接调用togglePause方法，这个方法会根据curState是否为null，来设置Fiber的isPausing的值，而isPausing表示Task是暂停还是恢复，相应源码如下：

```java
if (curState == null) {

    setState(PAUSE_STATE);

} else {

    stateStack[iStack] = null;

    isPausing = false;

}
```

当Task执行完某个Pauseable方法时，将会调用Fiber的up方法，标识调用某个Pauseable方法返回，且up方法的返回值表示该Pauseable方法是正常返回还是暂停返回，因为up方法内部会根据Fiber中的isPausing变量值和本函数栈帧stateStack[iStack]是否为null来判断是否暂停，以及函数栈帧是否已经保存。如果是PAUSING__NO_STATE，说明被调函数暂停，本函数还未保存栈帧，则需要将本函数栈帧，一般后续执行需要使用到的变量（包括函数实参、函数局部变量）保存到State中，也即 stateStack[iStack]，

```java
   /*
     * Status indicators returned by down()
     *
     * normal return, nothing to restore
     */
    public static final int   NOT_PAUSING__NO_STATE  = 0;
    
    /*
     * Normal return, have saved state to restore before resuming
     */
    public static final int   NOT_PAUSING__HAS_STATE = 1;
    
    /*
     * Pausing, and need to save state before returning
     */
    public static final int   PAUSING__NO_STATE      = 2;
    
    /*
     * Pausing, and have saved state from an earlier invocation,
     * so nothing left to do.
     */
    public static final int   PAUSING__HAS_STATE     = 3;
```

## 11.什么场景下适合做Kilim协程？

**IO密集型的应用比较适合使用协程**，比如应用中存在较多的与后端的网络交互，存在较多的时间在等待后端响应,可以保证线程不会阻塞在等待网络响应，充分利用多核多线程的能力。而对于CPU密集型应用，由于大部分情况CPU都比较繁忙，Kilim反而不会产生很好的作用。

## 12.Kilim中Task的工作机制？

Actor采取的这种类似消息机制的方式，实际在守护线程和外部线程之间有一个队列，俗称信箱，外部线程只要把请求放入，守护线程就读取进行处理。这种异步高效方式是Actor基本原理。Task 是轻量型的线程，它们通过 Kilim 的 Mailbox 类型与其他 Task 通信。

## 13.Kilim框架做了什么？

1. 利用字节码增强(基于ASM字节码框架)，将普通代码转化为支持协程的代码；
2. 调用pausable的时候，如果pause了就保存当前方法栈的State，停止执行，将控制权交给调度器；
3. 调度器负责协调就绪的协程；
4. 协程resume的时候，自动恢复State，回复到上次执行的位置继续执行
