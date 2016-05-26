# TimingLogger

标签（空格分隔）： `android` `log`

---

##What's TimingLogger
　　TimingLogger是android在API-1上就添加了的工具类，它通过方法调用就可以**打印程序执行的时间**。
##Typical Usage
```java
TimingLogger timings = new TimingLogger(TAG, "methodA");
// ... do some work A ...
timings.addSplit("work A");
// ... do some work B ...
timings.addSplit("work B");
// ... do some work C ...
timings.addSplit("work C");
timings.dumpToLog();
```

timings.dumpToLog()会将下面的日志加到Log中
```java
     D/TAG     ( 3459): methodA: begin
     D/TAG     ( 3459): methodA:      9 ms, work A
     D/TAG     ( 3459): methodA:      1 ms, work B
     D/TAG     ( 3459): methodA:      6 ms, work C
     D/TAG     ( 3459): methodA: end, 16 ms
```

##Sample
我用kotlin举例，和java语法差不多，对kotlin不熟的话可以勉强看一下
```kotlin
    /**
     * save 1000 cats to realm on the main thread
     */
    fun initCatsSync(realm: Realm) {
        val timeLogger = TimingLogger(TAG, "initCatsAsync") //start timing

        var cats = LinkedList<Cat>()

        timeLogger.addSplit("init cats list")

        for (i in 0..1000) {
            val cat = Cat(i, "cat$i")
            cats.add(cat)
        }

        timeLogger.addSplit("create and save 1000 cats to LinkdedList")
        timeLogger.dumpToLog()
    }
```
日志：
```java
D/CatUtil: initCatsAsync: begin
D/CatUtil: initCatsAsync:      0 ms, init cats list
D/CatUtil: initCatsAsync:      10 ms, create and save 1000 cats to LinkdedList
D/CatUtil: initCatsAsync: end, 10 ms
```
可以看出`var cats = LinkedList<Cat>()`执行的时间是0ms, for循环执行的时间是10ms,整个方法执行的时间是10ms

-----

##ISSUE
> + 正常操作后，我们会发现日志并没有被打出来
解决方法：
> 1. adb shell
> 2. setprop log.tag.`<tag>` `<level>`

eg:

$adb shell

$setprop log.tag.**CatUtil** **VERBOSE**

解析：
我们看一下源码：
```java
        mDisabled = !Log.isLoggable(mTag, Log.VERBOSE);
        if (mDisabled) return;
        if (mSplits == null) {
            mSplits = new ArrayList<Long>();
            mSplitLabels = new ArrayList<String>();
        } else {
            mSplits.clear();
            mSplitLabels.clear();
        }
        addSplit(null);
```
从第一行可以看到，要打日志的话，需要`Log.isLoggable(mTag, Log.VERBOSE)`返回true。我在自己的代码中打印一下发现确实返回的是false：

```java
Log.d(TAG, "Is Loggable? " + Log.isLoggable(TAG, Log.VERBOSE));
```

我们知道日志是有以下几个级别：**VERBOSE, DEBUG, INFO, WARN, ERROR, ASSERT, or SUPPRESS**。，级别由低到高。，verbose几乎会打印所有日志，suppress是关闭日志。譬如有如下几个日志：
> 
  Log.v(tag, "info")
  Log.d(tag, "info")
  Log.i(tag, "info")
  Log.e(tag, "info")
  
如果级别为VERBOSEDE，则会将上述所有日志都打印出来，级别为DEBUG则会将除Log.v以外的所有日志打印出来。
