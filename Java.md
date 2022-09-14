## JVM Safepoint的妙用-可能提前进行gc --JDK8

出自Rocketmq源码

```java
public void warmMappedFile(FlushDiskType type, int pages) {
        this.mappedByteBufferAccessCountSinceLastSwap++;

        long beginTime = System.currentTimeMillis();
        ByteBuffer byteBuffer = this.mappedByteBuffer.slice();
        int flush = 0;
        long time = System.currentTimeMillis();
        for (int i = 0, j = 0; i < this.fileSize; i += DefaultMappedFile.OS_PAGE_SIZE, j++) {
            byteBuffer.put(i, (byte) 0);
            // force flush when flush disk type is sync
            if (type == FlushDiskType.SYNC_FLUSH) {
                if ((i / OS_PAGE_SIZE) - (flush / OS_PAGE_SIZE) >= pages) {
                    flush = i;
                    mappedByteBuffer.force();
                }
            }

            // prevent gc
            if (j % 1000 == 0) {
                log.info("j={}, costTime={}", j, System.currentTimeMillis() - time);
                time = System.currentTimeMillis();
                try {
                    Thread.sleep(0);
                } catch (InterruptedException e) {
                    log.error("Interrupted", e);
                }
            }
        }
```

for循环中出现下列代码:

```java
// prevent gc
if (j % 1000 == 0) {
	log.info("j={}, costTime={}", j, System.currentTimeMillis() - time);
	time = System.currentTimeMillis();
	try {
		Thread.sleep(0);
	} catch (InterruptedException e) {
		log.error("Interrupted", e);
	}
}	
```

注释中说明是 prevent gc,实际上提前触发gc,防止出现长时间的gc.

### 安全点

只有所有线程都到达安全点时,才能进行gc

### 提前进入安全点的原理（可能提前触发gc）

> HotSpot 虚拟机为了避免安全点过多带来过重的负担，对循环还有一项优化措施，认为循环次数较少的话，执行时间应该也不会太长，所以使用 int  类型或范围更小的数据类型作为索引值的循环默认是不会被放置安全点的。这种循环被称为可数循环（Counted Loop），相对应地，使用 long  或者范围更大的数据类型作为索引值的循环就被称为不可数循环（Uncounted Loop），将会被放置安全点。
>
> ​																				----深入JVM虚拟机

**（自JDK10以后，HotSpot对Loop Strip Mining进行了优化，该特点已失效)**

简单的说,即使HotSpot虚拟机认为,循环在执行int类型或更小的数据类型作为索引时（可数循环），默认不放置安全点，使用long或者更大的数据类型时作为索引时（不可数循环），将会被放置安全点

### Thread.sleep(0)

使用Thread.sleep(0)可以进入安全点

#### 原理

下图为hotspot源码注释

> ```
> // Begin the process of bringing the system to a safepoint.
> // Java threads can be in several different states and are
> // stopped by different mechanisms:
> //
> //  1. Running interpreted
> //     The interpeter dispatch table is changed to force it to
> //     check for a safepoint condition between bytecodes.
> //  2. Running in native code
> //     When returning from the native code, a Java thread must check
> //     the safepoint _state to see if we must block.  If the
> //     VM thread sees a Java thread in native, it does
> //     not wait for this thread to block.  The order of the memory
> //     writes and reads of both the safepoint state and the Java
> //     threads state is critical.  In order to guarantee that the
> //     memory writes are serialized with respect to each other,
> //     the VM thread issues a memory barrier instruction
> //     (on MP systems).  In order to avoid the overhead of issuing
> //     a memory barrier for each Java thread making native calls, each Java
> //     thread performs a write to a single memory page after changing
> //     the thread state.  The VM thread performs a sequence of
> //     mprotect OS calls which forces all previous writes from all
> //     Java threads to be serialized.  This is done in the
> //     os::serialize_thread_states() call.  This has proven to be
> //     much more efficient than executing a membar instruction
> //     on every call to native code.
> //  3. Running compiled Code
> //     Compiled code reads a global (Safepoint Polling) page that
> //     is set to fault if we are trying to get to a safepoint.
> //  4. Blocked
> //     A thread which is blocked will not be allowed to return from the
> //     block condition until the safepoint operation is complete.
> //  5. In VM or Transitioning between states
> //     If a Java thread is currently running in the VM or transitioning
> //     between states, the safepointing code will wait for the thread to
> //     block itself when it attempts transitions to a new state.
> //
> ```

第二条中提到，执行naive方法返回后，必须进行安全点的检测，而sleep()就是一个naive方法，故从该方法返回后，会进入安全点（但是不一定进行gc）。

### 总结

通过调用Thread.sleep(0)（naive）方法，可以在方法结束后进入安全点，也可以把正在执行的native函数的线程看作已经进入了safepoint，或者看作在safe-region里。

而只有到了safepoint或者说到了safe-region后，JVM才**可能**进行gc

