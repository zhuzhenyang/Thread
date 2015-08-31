# Thread
Player  and   Game
1. 线程的挂起和唤醒
      挂起实际上是让线程进入“非可执行”状态下，在这个状态下CPU不会分给线程时间片，进入这个状态可以用来暂停一个线程的运行；在线程挂起后，可以通过重新唤醒线程来使之恢复运行。

挂起的原因可能是如下几种情况：
     （1）通过调用sleep()方法使线程进入休眠状态，线程在指定时间内不会运行。
     （2）通过调用join()方法使线程挂起，使自己等待另一个线程的结果，直到另一个线程执行完毕为止。
     （3）通过调用wait()方法使线程挂起，直到线程得到了notify()和notifyAll()消息，线程才会进入“可执行”状态。
     （4）使用suspend挂起线程后，可以通过resume方法唤醒线程。
      虽然suspend和resume可以很方便地使线程挂起和唤醒，但由于使用这两个方法可能会造成死锁，因此，这两个方法被标识为 deprecated（抗议）标记，这表明在以后的jdk版本中这两个方法可能被删除，所以尽量不要使用这两个方法来操作线程。

      调用sleep()、yield()、suspend()的时候并没有被释放锁
      调用wait()的时候释放当前对象的锁

      wait()方法表示，放弃当前对资源的占有权，一直等到有线程通知，才会运行后面的代码。
      notify()方法表示，当前的线程已经放弃对资源的占有，通知等待的线程来获得对资源的占有权，但是只有一个线程能够从wait状态中恢复，然后继续运行wait()后面的语句。
      notifyAll()方法表示，当前的线程已经放弃对资源的占有，通知所有的等待线程从wait()方法后的语句开始运行。 

2.等待和锁实现资源竞争
      等待机制与锁机制是密切关联的，对于需要竞争的资源，首先用synchronized确保这段代码只能一个线程执行，可以再设置一个标志位condition判断该资源是否准备好，如果没有，则该线程释放锁，自己进入等待状态，直到接收到notify，程序从wait处继续向下执行。
synchronized(obj) {
　　while(!condition) {
　　 obj.wait();
　　}
　　obj.doSomething();
}
以上程序表示只有一个线程A获得了obj锁后，发现条件condition不满足，无法继续下一处理，于是线程A释放该锁，进入wait()。

      在另一线程B中，如果B更改了某些条件，使得线程A的condition条件满足了，就可以唤醒线程A：
synchronized(obj) {
　condition = true;
　obj.notify();
}
需要注意的是：
　　# 调用obj的wait(), notify()方法前，必须获得obj锁，也就是必须写在synchronized(obj) {...} 代码段内。
　　# 调用obj.wait()后，线程A就释放了obj的锁，否则线程B无法获得obj锁，也就无法在synchronized(obj) {...} 代码段内唤醒A。
　　# 当obj.wait()方法返回后，线程A需要再次获得obj锁，才能继续执行。
　　# 如果A1,A2,A3都在obj.wait()，则B调用obj.notify()只能唤醒A1,A2,A3中的一个(具体哪一个由JVM决定)。
　　# obj.notifyAll()则能全部唤醒A1,A2,A3，但是要继续执行obj.wait()的下一条语句，必须获得obj锁，因此，A1,A2,A3只有一个有机会获得锁继续执行，例如A1，其余的需要等待A1释放obj锁之后才能继续执行。
　　# 当B调用obj.notify/notifyAll的时候，B正持有obj锁，因此，A1,A2,A3虽被唤醒，但是仍无法获得obj锁。直到B退出synchronized块，释放obj锁后，A1,A2,A3中的一个才有机会获得锁继续执行。
