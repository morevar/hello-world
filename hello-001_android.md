- 消息机制
```
关键字：Handler、MessageQueue、Looper；Looper中的ThreadLocal
理解：
(1)从开发的角度来看，Handler是消息机制的上层接口，开发工作中一般也只需和Handler交互。通过Handler，我们可以将一个任务切换到Handler所在线程中执行。使用场景举例：在主线程中创建一个Handler，在子线程中执行完耗时操作后使用Handler发送消息到主线程，在主线程的消息回调中更新UI。
(2)为什么需要？直接与主线程交互(访问/修改其中的变量)可能会带来线程安全问题。通过消息机制，我们可以在子线程中发送消息，让主线程进行处理并执行我们的逻辑代码。
(3)MessageQueue，提供消息的存储，以队列(实际是单链表)的形式对外提供消息的插入和删除。主要方法有enqueueMessage()和next()。
(4)Looper，不停地从MessageQueue中查看是否有新消息，有的话立刻处理，没有就会一直阻塞
(5)Handler，扮演着消息发送和消息处理的角色。
(6)工作流程：Handler发送消息(向消息队列中插入消息)，MessageQueue的next()返回消息给Looper，Looper接收到消息后开始处理(最终的消息是交给Handler处理，即Handler的disaptchMessage()被调用。
(7)ThreadLocal，可以在不同线程中互不干扰的存储和读取数据，通过ThreadLocal就可以轻松获取到每个线程的Looper。

[20171206]腾讯云·深入理解 Android 消息机制原理 https://zhuanlan.zhihu.com/p/31760169
[20170616]重温Android中的消息机制 https://www.cnblogs.com/lilinjie/p/7026063.html
```

- 事件分发机制
```
关键字：Touch、dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()
理解：
(1)事件分发，将点击事件(MotionEvent)传递到某个具体的View和处理的整个过程。
(2)事件的传递：先传递到ViewGroup，再由ViewGroup传递到View。
(3)事件的拦截：在ViewGroup中可以通过onInterceptTouchEvent方法对事件传递进行拦截，onInterceptTouchEvent方法返回true代表不允许事件继续向子View传递，返回false代表不对事件进行拦截，默认返回false。

(4)例子：点击了某个控件，首先会去调用该控件所在布局的dispatchTouchEvent方法，然后在布局的dispatchTouchEvent方法中找到被点击的相应控件，再去调用该控件的dispatchTouchEvent方法。
 >>> 点击View->ViewGroup.dispatchTouchEvent()->在ViewGroup中查找View->View.dispatchTouchEvent()
 >>> 点击View->ViewGroup.dispatchTouchEvent()[如果此处onInterceptTouchEvent()返回true，直接调用ViewGroup的onTouch/onTouchEvent/performClick/onClick等方法消费掉此事件，不再向View传递]->在ViewGroup中查找View->View.dispatchTouchEvent()

(5)View.dispatchTouchEvent()：最先执行的就是onTouch方法(优先于onClick执行)
 >>> 如果在onTouch方法里返回了true，就会让dispatchTouchEvent方法直接返回true，不会再继续往下执行(onClick就不会再执行)。
 >>> 如果在onTouch方法里返回了false，就一定会进入到onTouchEvent方法中。

(6)onTouch和onTouchEvent：这两个方法都是在View的dispatchTouchEvent中调用的，onTouch优先于onTouchEvent执行。如果在onTouch方法中通过返回true将事件消费掉，onTouchEvent将不会再执行。onTouch能够得到执行需要两个前提条件，第一mOnTouchListener的值不能为空，第二当前点击的控件必须是enable的。

(6)每次点击事件，都会触发一系列的ACTION_DOWN，ACTION_MOVE，ACTION_UP等事件。
 >>> 这里需要注意，当View不是可点击的时候，如果你在执行ACTION_DOWN的时候返回了false，后面一系列其它的action就不会再得到执行了。简单的说，就是当dispatchTouchEvent在进行事件分发的时候，只有前一个action返回true，才会触发后一个action。
 >>> 当View是可以点击的时候，onTouch返回false时会继续执行onTouchEvent(执行onClick)，onTouch返回true的时候结束事件分发(不会执行onClick)。
 >>> 即，View不可点击时，onTouch返回false的话只触发一次onTouch；View可以点击的时候，onTouch返回false会执行onClick。

[20130620]Android事件分发机制完全解析，带你从源码的角度彻底理解(上) http://blog.csdn.net/guolin_blog/article/details/9097463/
[20130705]Android事件分发机制完全解析，带你从源码的角度彻底理解(下) http://blog.csdn.net/guolin_blog/article/details/9153747
[20170106]Android事件分发机制 详解攻略 http://blog.csdn.net/carson_ho/article/details/54136311
[20170502]Android View事件处理 https://www.jianshu.com/p/6ed3b8616421
```
