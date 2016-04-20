<pre>
注意事項：
	1. 文章內容皆為自己臨時撰寫並且已經過實測，僅提供了部分程式碼並不能直接運行 
	2. 文中有使用另一篇文章內容可參考 <a href="http://tsai-cookie.blogspot.tw/2015/10/java-sample-trigger-and-masker.html">Trigger and Masker</a>
	3. 僅供參考
</pre>

## 目錄

[TOC]

## 前言

> 今天恰好在某社群上看到一篇關於 『RESTful API如何一次完成多個相依請求』的內容，想想小編也是做這方面的工作。索性將幾個曾經遇到的問題，寫成自己用過的解決方案提供給大家作為參考使用。

## 問題

本篇文章中有 Resource1, Resource2, Resource3 三支 Api，其中目的，如下：

1. Resource 1：提供查詢功能可回應 Data1 及 Data2 兩種內容。
2. Resource 2：提供創建（例如：創建使用者）
3. Resource 3：提供創建（例如：創建團體）

P.S. 以下都用 R1, R2, R3 作為簡稱。

情境描述，如下：

1. 通過 R1 希望可以取得多種回應結果，例如：使用者的基本資料或所擁有的權限，選擇部分或全部取得。
2. 執行 R2 的某些情況下可能有需要同時執行 R3，例如：在沒有團體時創建使用者，將同時創建一個團體。

P.S. 以上皆為隨意假設需以真實情況異動，請勿直接取用。

## 解決方法

1. 在情境中選擇使用一個 Tigger Value（一種二進制整數）做為開關，當請求發送上來後可以決定需要的資料。
2. 在執行 R2 時多帶一個參數決定是否要額外執行 R3。（有將 R2 的結果帶入 R3）

P.S. Trigger Value 的製作與應用請參考 <a href="http://tsai-cookie.blogspot.tw/2015/10/java-sample-trigger-and-masker.html">Trigger and Masker</a>


## Smaple Code

### Resource Api

```
/**
 * Created by Cookie on 16/4/15.
 */
@Singleton
@Produces(MediaType.APPLICATION_JSON)
@Path("sample")
public class SampleApi {

    private static final SingleService singleService = new SampleService();

    @GET
    @Path("resource1")
    public SingleResponse getResource1(@QueryParam("tv") Integer triggerValue) {
        SingleResponse response = new SingleResponse();

        Trigger trigger = SampleMasker.newTrigger(triggerValue);
        if (trigger.isOn(SampleMasker.OPTION_01)) {
            // 為證明有執行在回應中放入點資料
            response.put("data1", "value1");
        }
        if (trigger.isOn(SampleMasker.OPTION_02)) {
            // 為證明有執行在回應中放入點資料
            response.put("data2", "value2");
        }
        // 回傳 Response
        return response.ok();
    }

    @POST
    @Path("resource2")
    public SingleResponse getResource2(@QueryParam("make5") Boolean make5, 
                                       SingleRequest request) {

        // 執行 Resource 2
        SingleResponse response = AppUtils.callback(singleService);
        // 為證明有執行在回應中放入點資料
        response.put("R2", "ok");

        // Resource 2 執行成功且請求有需要完成 Resource 3 則執行 Resource 3
        if (response.isSuccess() && BooleanUtils.isTrue(make5)) {

            // 取出 Resource 2 執行後獲得的資料
            Map<String, Object> map = response.getDataObj();

            // 將 Resource 2 執行後的內容傳送給 Resource 3
            SingleResponse response3 = getResource3(new SingleRequest().putAll(map));

            // TODO 可自行決定是否將 Resource 3 與 Resource 2 的 Response 合併
            // 為證明有執行在回應中放入點資料
            response.put("R3", "ok");
        }

        // 回傳 Response
        return response;
    }

    @POST
    @Path("resource3")
    public SingleResponse getResource3(SingleRequest request) {
        // 執行 Resource 3 並回傳 Response
        return AppUtils.callback(singleService);
    }
}
```


### Resource 1 測試與輸出

```
// 僅取回 第一種資料
$ curl -s http://127.0.0.1:9090/api/sample/resource1?tv=1 | jq .
{
  "data": {
    "data1": "value1"
  },
  "code": 0,
  "success": true
}
// 僅取回 第二種資料
$ curl -s http://127.0.0.1:9090/api/sample/resource1?tv=2 | jq .
{
  "data": {
    "data2": "value2"
  },
  "code": 0,
  "success": true
}
// 同時取回 第一及第二種資料
$ curl -s http://127.0.0.1:9090/api/sample/resource1?tv=3 | jq .
{
  "data": {
    "data1": "value1",
    "data2": "value2"
  },
  "code": 0,
  "success": true
}
```

### Resource 2 測試與輸出

```
// 執行 Resource 2 並要求執行 Resource 3
$ curl -s -X POST http://127.0.0.1:9090/api/sample/resource2?make5=true -d "{}" | jq .
{
  "data": {
    "R2": "ok",
    "R3": "ok"
  },
  "code": 0,
  "success": true
}
```

