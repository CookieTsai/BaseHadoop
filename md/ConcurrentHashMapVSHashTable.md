<pre>
注意事項：
	1. 實驗一，自己編寫測試
	2. 實驗二，參考至 <a href="http://crunchify.com/hashmap-vs-concurrenthashmap-vs-synchronizedmap-how-a-hashmap-can-be-synchronized-in-java/">HashMap Vs. ConcurrentHashMap Vs. SynchronizedMap</a>
	3. 僅供參考使用
</pre>

## 目錄

[TOC]

## 前言

在工作上經常會接觸到多執行緒的問題，其中一件事情就是當很多執行緒需要共同存取一個 Map 的時後該怎麼處理。

Map 的功能就像是一個字典檔，而很多的執行緒就等於有很多的人要編寫或讀取這個字典檔，如果這時使用 HashMap 會容易發生問題。例如：ConcurrentModificationException、順利找到對應的 key 取出 value 但值卻是 null、同時存取 Map 導致 Lock... 等。

在 JDK 1.5 版後常見的解法大概有二種，如下：

1. 使用 Hashtable 類別
2. 使用 ConcurrentHashMap 類別
3. 使用 Collections.synchronizedMap() 方法

為了更加認識這幾個 Map 的使用時機與狀況，在本篇文章中小編整理了兩種實驗方式及結果。

## 實驗一

### Writer

```
import java.util.Map;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * Created by Cookie on 16/3/1.
 */
public class Writer extends Thread{

    private Map<String, String> map;
    private boolean run = true;

    public Writer(Map<String, String> map) {
        this.map = map;
    }

    public void close() {
        run = false;
    }

    public void run() {
        while (run) {
            try {
                String keyAndValue = UUID.randomUUID().toString();
                map.put(keyAndValue, keyAndValue);
                TimeUnit.NANOSECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### SampleMain

```
import java.util.*;
import java.util.concurrent.*;

import static java.lang.System.out;

/**
 * Created by Cookie on 16/3/1.
 */
public class SampleMain {

    private static final int WRITERS_NUM = 10;
    private static final int READ_TIMES = 500;

    private static void execute(Map<String, String> map) {
        List<Writer> writers = new ArrayList<Writer>();
        try {
            long started = System.currentTimeMillis();
            for (int i = 0; i < WRITERS_NUM; i++) writers.add(new Writer(i, map));
            for (Writer writer: writers) writer.start();
            for (int i = 0; i < READ_TIMES; i++) {
                for (Map.Entry<String, String> entry : map.entrySet()) {
                    if (entry.getKey() == null || entry.getValue() == null) {
                        out.println(entry.getKey());
                        out.println(entry.getValue());
                    }
                }
                TimeUnit.MILLISECONDS.sleep(1);
            }
            out.println(String.format("Size:%d ,Spend Time:%d", map.size(),
                                                                (System.currentTimeMillis() - started)));
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            for (Writer writer: writers) {
                writer.close();
            }
        }
    }

    private static void run(Map<String, String> map) {
        out.println(map.getClass().getSimpleName() + " Start");
        try {
            execute(map);
        } catch (Exception e) {
            e.printStackTrace();
        }
        out.println(map.getClass().getSimpleName() + " Stop");
    }

    public static void main(String[] args) {
        run(new ConcurrentHashMap<String, String>());
        run(new Hashtable<String, String>());
        run(Collections.synchronizedMap(new HashMap()));
    }
}
```

### Console Output

```
ConcurrentHashMap Start
Size:5470 ,Spend Time:707
ConcurrentHashMap Stop
Hashtable Start
java.util.ConcurrentModificationException
	at java.util.Hashtable$Enumerator.next(Hashtable.java:1167)
	at sample.concurrent.SampleMain.execute(SampleMain.java:23)
	at sample.concurrent.SampleMain.run(SampleMain.java:45)
	at sample.concurrent.SampleMain.main(SampleMain.java:54)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
Hashtable Stop
SynchronizedMap Start
java.util.ConcurrentModificationException
	at java.util.HashMap$HashIterator.nextEntry(HashMap.java:922)
	at java.util.HashMap$EntryIterator.next(HashMap.java:962)
	at java.util.HashMap$EntryIterator.next(HashMap.java:960)
	at sample.concurrent.SampleMain.execute(SampleMain.java:23)
	at sample.concurrent.SampleMain.run(SampleMain.java:45)
	at sample.concurrent.SampleMain.main(SampleMain.java:55)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
SynchronizedMap Stop
```

## 實驗二

### Do you have any of below questions?

* What’s the difference between ConcurrentHashMap and Collections.synchronizedMap(Map)?
* What’s the difference between ConcurrentHashMap and Collections.synchronizedMap(Map) in term of performance?
* ConcurrentHashMap vs Collections.synchronizedMap()
* Popular HashMap and ConcurrentHashMap interview questions

In this tutorial we will go over all above queries and reason why and how we could Synchronize HashMap?

### Why?

The Map object is an associative containers that store elements, formed by a combination of a uniquely identify key and a mapped value. If you have very highly concurrent application in which you may want to modify or read key value in different threads then it’s ideal to use Concurrent Hashmap. Best example is Producer Consumer which handles concurrent read/write.

So what does the thread-safe Map means? If multiple threads access a hash map concurrently, and at least one of the threads modifies the map structurally, it must be synchronized externally to avoid an inconsistent view of the contents.

### How?

There are two ways we could synchronized HashMap

* Java Collections synchronizedMap() method
* Use ConcurrentHashMap

### ConcurrentHashMap

* You should use ConcurrentHashMap when you need very high concurrency in your project.
* It is thread safe without synchronizing the whole map.
* Reads can happen very fast while write is done with a lock.
* There is no locking at the object level.
* The locking is at a much finer granularity at a hashmap bucket level.
* ConcurrentHashMap doesn’t throw a ConcurrentModificationException if one thread tries to modify it while another is iterating over it.
* ConcurrentHashMap uses multitude of locks.

### SynchronizedHashMap

* Synchronization at Object level.
* Every read/write operation needs to acquire lock.
* Locking the entire collection is a performance overhead.
* This essentially gives access to only one thread to the entire map & blocks all the other threads.
It may cause contention.
* SynchronizedHashMap returns Iterator, which fails-fast on concurrent modification.

### Now let’s take a look at code

1. Create class CrunchifyConcurrentHashMapVsSynchronizedHashMap.java
2. Create object for each HashTable, SynchronizedMap and CrunchifyConcurrentHashMap
3. Add and retrieve 500k entries from Map
4. Measure start and end time and display time in milliseconds
5. We will use ExecutorService to run 5 threads in parallel

### SampleMain2

```
import java.util.*;
import java.util.concurrent.*;

import static java.lang.System.out;

/**
 * Created by Cookie on 16/3/1.
 */
public class SampleMain2 {
    public final static int POOL_SIZE = 5;

    public static Map<String, Integer> hashtableObj = null;
    public static Map<String, Integer> syncMapObj = null;
    public static Map<String, Integer> concurrentHashMapObj = null;

    public static void main(String[] args) throws Exception {

        // Test with Hashtable Object
        hashtableObj = new Hashtable<String, Integer>();
        performTest(hashtableObj);

        // Test with synchronizedMap Object
        syncMapObj = Collections.synchronizedMap(new HashMap<String, Integer>());
        performTest(syncMapObj);

        // Test with ConcurrentHashMap Object
        concurrentHashMapObj = new ConcurrentHashMap<String, Integer>();
        performTest(concurrentHashMapObj);

    }

    public static void performTest(final Map<String, Integer> t) throws Exception {

        out.println("Test started for: " + t.getClass());
        long avgTime = 0;
        for (int i = 0; i < 5; i++) {

            long startTime = System.nanoTime();
            ExecutorService exService = Executors.newFixedThreadPool(POOL_SIZE);

            for (int j = 0; j < POOL_SIZE; j++) {
                exService.execute(new Runnable() {
                    @SuppressWarnings("unused")
                    public void run() {
                        for (int i = 0; i < 500000; i++) {
                            Integer randomNum = (int) Math.ceil(Math.random() * 550000);
                            // Retrieve value. We are not using it anywhere
                            Integer value = t.get(String.valueOf(randomNum));
                            // Put value
                            t.put(String.valueOf(randomNum), randomNum);
                        }
                    }
                });
            }

            // Make sure executor stops
            exService.shutdown();
            // Blocks until all tasks have completed execution after a shutdown request
            exService.awaitTermination(Long.MAX_VALUE, TimeUnit.DAYS);

            long entTime = System.nanoTime();
            long totalTime = (entTime - startTime) / 1000000L;
            avgTime += totalTime;
            out.println("500K entried added/retrieved in " + totalTime + " ms");
        }
        out.println("For "+ t.getClass()+ " the average time is "+ avgTime / 5+ " ms\n");
    }
}
```

### Console Output

```
Test started for: class java.util.Hashtable
500K entried added/retrieved in 1530 ms
500K entried added/retrieved in 1336 ms
500K entried added/retrieved in 1283 ms
500K entried added/retrieved in 1243 ms
500K entried added/retrieved in 1289 ms
For class java.util.Hashtable the average time is 1336 ms

Test started for: class java.util.Collections$SynchronizedMap
500K entried added/retrieved in 1409 ms
500K entried added/retrieved in 1360 ms
500K entried added/retrieved in 1222 ms
500K entried added/retrieved in 1260 ms
500K entried added/retrieved in 1299 ms
For class java.util.Collections$SynchronizedMap the average time is 1310 ms

Test started for: class java.util.concurrent.ConcurrentHashMap
500K entried added/retrieved in 559 ms
500K entried added/retrieved in 547 ms
500K entried added/retrieved in 654 ms
500K entried added/retrieved in 468 ms
500K entried added/retrieved in 454 ms
For class java.util.concurrent.ConcurrentHashMap the average time is 536 ms
```

## 筆記

在實驗一，發現在大量 Thread 存取 Map 時如有發生 `java.util.ConcurrentModificationException` 的話，選擇使用 ConcurrentHashMap。在實驗二，大量的 get 及 put 過程中 ConcurrentHashMap 可能提供更佳的效率。因此，在無特殊的前提下使用 ConcurrentHashMap 確實可能得到最好的結果。



