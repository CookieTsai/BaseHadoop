## Grizzly 2.0: HttpServer API. Asynchronous HTTP Server - Part I

<pre>
注意事項：
	1. 本文轉載於 <a href="https://blogs.oracle.com/oleksiys/entry/grizzly_2_0_httpserver_api1">Mytec Blog for oleksiys</a>
	2. 建議先瞭解一下 <a href="https://grizzly.java.net/iostrategies.html">Grizzly IOStrategies</a> 的第一種 Worker-thread 模式
	3. 僅供參考查閱使用
</pre>

In my previous blog entry I described basic Grizzly HttpServer API abstractions and offered a couple of samples to show how one can implement light-weight Servlet-like web application.

Here I'll try to show how we can process HTTP requests asynchronously within HttpHandler, in other words implement asynchronous HTTP application.

**What do we mean by "asynchronous"?**

Normally HttpServer has a service thread-pool, whose threads are used for HTTP requests processing, which includes following steps:

1. parse HTTP request;
2. execute processing logic by calling HttpHandler.handle(Request, Response);
3. flush HTTP response;
4. return service thread to a thread-pool.

Normally the steps above are executed sequentially in a service thread. Using "asynchronous" feature it's possible to delegate execution of steps 2 and 3 to a custom thread, which will let us release the service thread faster.

**Why would we want to do that?**

As it was said above, the service thread-pool instance is shared between all the HttpHandlers registered on HttpServer. Assume we have application (HttpHandler) "A", which  executes a long lasting task (say a SQL query on pretty busy DB server), and application "B", which serves static resources. It's easy to imagine the situation, when couple of application "A" clients block all the service threads by waiting for response from DB server. The main problem is that clients of application "B", which is pretty light-weight, can not be served at the same time because there are no available service threads. So it might be a good idea to isolate these applications by executing application "A" logic in the dedicated thread pool, so service threads won't be blocked.

Ok, let's do some coding and make sure the issue we've just described is real.

<pre>
HttpServer httpServer = new HttpServer();

NetworkListener networkListener = new NetworkListener("sample-listener", "127.0.0.1", 18888);
<font color="red">
// Configure NetworkListener thread pool to have just one thread,
// so it would be easier to reproduce the problem
ThreadPoolConfig threadPoolConfig = ThreadPoolConfig
        .defaultConfig()
        .setCorePoolSize(1)
        .setMaxPoolSize(1);
</font>
networkListener.getTransport().setWorkerThreadPoolConfig(threadPoolConfig);

httpServer.addListener(networkListener);

httpServer.getServerConfiguration().addHttpHandler(new HttpHandler() {
    @Override
    public void service(Request request, Response response) throws Exception {
        response.setContentType("text/plain");
        response.getWriter().write("Simple task is done!");
    }
}, "/simple");

httpServer.getServerConfiguration().addHttpHandler(new HttpHandler() {
    @Override
    public void service(Request request, Response response) throws Exception {
        response.setContentType("text/plain");
        <font color="red">// Simulate long lasting task
        Thread.sleep(10000);</font>
        response.getWriter().write("Complex task is done!");
    }
}, "/complex");

try {
    server.start();
    System.out.println("Press any key to stop the server...");
    System.in.read();
} catch (Exception e) {
    System.err.println(e);
}
</pre>
	
In the sample above we create, initialize and run HTTP server, which has 2 applications (HttpHandlers) registered: "simple" and "complex". To simulate long-lasting task in the "complex" application we're just causing the current thread to sleep for 10 seconds.

Now if you try to call "simple" application from you Web browser using URL: http://localhost:18888/simple - you see the response immediately. However, if you try to call "complex" application http://localhost:18888/complex - you'll see response in 10 seconds. That's fine. But try to call "complex" application first and then quickly, in different tab, call the "simple" application, do you see the response immediately? Probably not. You'll see the response right after "complex" application execution completed. The sad thing here is that service thread, which is executing "complex" operation is idle (the same situation is when you wait for SQL query result), so CPU is doing nothing, but still we're not able to process another HTTP request.

How we can rework the "complex" application to execute its task in custom thread pool? Normally application (HttpHandler) logic is encapsulated within HttpHandler.service(Request, Response) method, **once we exit this method, Grizzly finishes and flushes HTTP response**. So coming back to the service thread processing steps:

1. <font color="#AAA">parse HTTP request;</font>
2. execute processing logic by calling HttpHandler.handle(Request, Response);
3. flush HTTP response;
4. <font color="#AAA">return service thread to a thread-pool.</font>

we see that it wouldn't be enough just to delegate HTTP request processing to a custom thread on step 2, because on step 3 Grizzly will automatically flush HTTP response back to client at the state it currently is. We need a way to instruct Grizzly to not do 3 automatically on the service thread, instead we want to be able to perform this step ourselves once asynchronous processing is complete.

Using Grizzly HttpServer API it could be achieved following way:

* HttpResponse.suspend(...) to instruct Grizzly to not flush HTTP response in the service thread;
* HttpResponse.resume() to finish HTTP request processing and flush response back to client.

So asynchronous version of the "complex" application (HttpHandler) will look like:

<pre>
httpServer.getServerConfiguration().addHttpHandler(new HttpHandler() {
    final ExecutorService complexAppExecutorService =
        GrizzlyExecutorService.createInstance(
            ThreadPoolConfig.defaultConfig()
            .copy()
            .setCorePoolSize(5)
            .setMaxPoolSize(5));
            
    @Override
    public void service(final Request request, final Response response) throws Exception {
                
        <font color="red">response.suspend(); // Instruct Grizzly to not flush response, once we exit the service(...) method </font>
                
        <font color="red">complexAppExecutorService</font>.execute(new Runnable() {   // Execute long-lasting task in the custom thread
            public void run() {
                try {
                    response.setContentType("text/plain");
                    // Simulate long lasting task
                    Thread.sleep(10000);
                    response.getWriter().write("Complex task is done!");
                } catch (Exception e) {
                    response.setStatus(HttpStatus.INTERNAL_SERVER_ERROR_500);
                } finally {
                    <font color="red">response.resume();  // Finishing HTTP request processing and flushing the response to the client</font>
                }
            }
        });
    }
}, "/complex");
</pre>

* As you might have noticed, "complex" application uses Grizzly ExecutorService implementation. This is the preferred approach, however you can still use own ExecutorService.

The three most important steps in the code above are marked red:

1. Suspend HTTP response processing: **response.suspend()**
2. Delegating task to the custom thread pool: **complexAppExecutorService.execute(...)**
3. Resuming HTTP response processing: **response.resume()** 

Now, using your browser, you can make sure "simple" and "complex" applications are not affecting each other, and the "simple" application works just fine when the "complex" application is busy.

### 筆記

實作中筆者刻意將 thread pool 設定為僅有一個 Worker 的情況下運行。

在此案例中，如果我們先行呼叫 complex 而後在呼叫 simple 會發現，simple 需要等 complex 做完才可以運行。

在這樣的狀況下會發生一個問題，也就是**當一些複雜的 API 被呼叫時可能造成一些簡單的 Request 在排隊等待**。

緊接著筆者透過另起一群 Thread pool 專門處理 complex 這樣即使只有一個 Worker 也不至會造成排隊的問題。

個人對此文章的理解為：**建議耗時長的API應與耗時短的API，分別處理這樣可以使伺服器運作得更為順利**