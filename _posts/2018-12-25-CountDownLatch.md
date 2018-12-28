[阅读原文](http://ifeve.com/countdownlatch%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/)



相比`ReentranceLock`，`CountDownLatch`的流程还是相对比较简单的，`CountDownLatch`也是基于`AQS`，它是`AQS`的共享功能的一个实现。下面从源代码的实现上详解`CountDownLatch`。

## 1. CountDownLatch 构造

```java
public CountDownLatch(int count) {
	if (count < 0) 
        throw new IllegalArgumentException("count < 0");
	// Sync 就是继承了一个AQS
	this.sync = new Sync(count);
}
```

下面为`CountDownLatch`中`Sync`的部分代码片段，`CountDownLatch`构造器中的`count`最终还是传递了`ASQ`中的`state`，所以`CountDownLatch`中的`countDown`也是对于`state`状态的改变。

```java
private static final class Sync extends AbstractQueuedSynchronizer {
	private static final long serialVersionUID = 4982264981922014374L;
	Sync(int count) {
		setState(count);
	}
}
```



## 2. CountDownLatch.countDown()实现

先看countDown所涉及的代码

```java
public void countDown() {
	sync.releaseShared(1);
}
```

```java
public final boolean releaseShared(int arg) {
	//如果计数器的值为0，那么执行下面操作
	if (tryReleaseShared(arg)) { 
		//这一步的操作主要是唤醒主线程，因为如果state不等于0的话，主线程一直是阻塞的
		doReleaseShared(); 
		return true;
	}
	return false;
}
```

```java
//CountDownLatch重写的方法
@Override
protected boolean tryReleaseShared(int releases) {
	// Decrement count; signal when transition to zero
	for (;;) {
		int c = getState();
        //countDown的计数器以及到0，体现在我们程序中的，就是并行的几个任务已经执行完
		if (c == 0) 
			return false;
        //每执行一次countDown，计数器减1
		int nextc = c-1; 
        // 利用cas来更新state的状态，这里可能有并发，所以这也是用死循环更新的原因
		if (compareAndSetState(c, nextc)) 
            //更新成功就返回
			return nextc == 0; 
	}
}
```

```java
private void doReleaseShared() {
	for (;;) {
		Node h = head;
		if (h != null && h != tail) { //至少有两个节点
			int ws = h.waitStatus;
			if (ws == Node.SIGNAL) { // 说明后继节点需要唤醒
				if (!compareAndSetWaitStatus(h, Node.SIGNAL, 0))
					continue; // loop to recheck cases
				unparkSuccessor(h); //唤醒后继节点
			} 
            else if (ws == 0 && !compareAndSetWaitStatus(h, 0, Node.PROPAGATE))
				continue; // loop on failed CAS
		}
	
        if (h == head) // loop if head changed
			break;
	}
}
```

```java
private void unparkSuccessor(Node node) {
	int ws = node.waitStatus;
    if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);
    
    Node s = node.next;
	if (s == null || s.waitStatus > 0) {
		s = null;
		for (Node t = tail; t != null && t != node; t = t.prev)
			if (t.waitStatus <= 0)
				s = t;
	}
    if (s != null)
		//唤醒主线程
		LockSupport.unpark(s.thread);
}
```



## 3. CountDownLatch.await()实现

```java
public void await() throws InterruptedException {
	sync.acquireSharedInterruptibly(1);
}
```

```java
public final void acquireSharedInterruptibly(int arg)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        if (tryAcquireShared(arg) < 0)
            doAcquireSharedInterruptibly(arg);
}
```

```java
private void doAcquireSharedInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.SHARED);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    // state的状态是0，说明，countDown的所有任务已经完成
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);//主线程所在的节点设置为头节点
                        p.next = null; // help GC
                        failed = false;
                        //主线程结束等待
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }
```

```java
private void setHeadAndPropagate(Node node, int propagate) {
        Node h = head; 
        setHead(node);
       
        if (propagate > 0 || h == null || h.waitStatus < 0 ||
            (h = head) == null || h.waitStatus < 0) {
            Node s = node.next;
            if (s == null || s.isShared())
                doReleaseShared();
        }
    }
```

```java
private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this);
        return Thread.interrupted();
    }
```





```java
//取消当前节点获取锁
private void cancelAcquire(Node node) {
        // Ignore if node doesn't exist
        if (node == null)
            return;
        node.thread = null;
        Node pred = node.prev;
        while (pred.waitStatus > 0)
            node.prev = pred = pred.prev;
		//如果节点都没有被取消的话，那么这个节点和node是同一个节点
        Node predNext = pred.next;
		//node的后继节点取消
        node.waitStatus = Node.CANCELLED;
        // CountDownLatch 逻辑就到这里
        if (node == tail && compareAndSetTail(node, pred)) {
            compareAndSetNext(pred, predNext, null);
        } else {
            int ws;
            if (pred != head &&
                ((ws = pred.waitStatus) == Node.SIGNAL ||
                 (ws <= 0 && compareAndSetWaitStatus(pred, ws, Node.SIGNAL))) &&
                pred.thread != null) {
                Node next = node.next;
                if (next != null && next.waitStatus <= 0)
                    compareAndSetNext(pred, predNext, next);
            } else {
                unparkSuccessor(node);
            }
            node.next = node; // help GC
        }
    }
```

