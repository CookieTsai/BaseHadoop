# HBase Split Table And Base64 Pre-Split Shell


## Split Table 的三種方法

<h3>第一種：Pre-Splitting</h3>

當 Table 被建立時，預設會配置一個 Region 給 Table，這時所有讀寫請求都會訪問到同一個 RegionServer 的同一個 Region 中，此時是沒有負載平衡的效果的，集群中可能存在空閒的 RegionServer。為了解決這個問題可以利用 Pre-Splitting，在建立 Table 的時候配置多個 Region 到同一個 Table 以期許達到分散的效果。

在 Table 初始化時，HBase 是不知道如何去 Split Region 的，因為 HBase 不知道RowKey 會以怎樣的方式被配置及產生。HBase 自帶了數種 Pre-Split 的算法，如 HexStringSplit 和 UniformSplit，我們可以根據 RowKey 的前綴來決定使用哪種 Split 方法。

**範例**

```
$ hbase org.apache.hadoop.hbase.util.RegionSplitter pre_split_table HexStringSplit -c 10 -f f1
```

P.S. `-c` 10 為 Region 數目；`-f`  f1 為 Table's Column Family.

<h4>Base64 Pre-Split Table Shell</h4>

由於小編用的是 Base64 字元當前綴，所以另外寫了一個小工具可以幫助切割 Table，產出的結果可在 HBase Shell 中執行。

**preSplit.sh**

``` preSplit.sh
#!/bin/bash

CHARS="1234567890qwertyuiopasdfghjklzxcvbnmQWERTYUIOPASDFGHJKLZXCVBNM+/"
TABLE=$1
PARTS=$2

CHARS_LEN=$(echo $CHARS | perl -pe 's/(\S)/\1\n/g' | xargs -I {} | wc -l)
RANGE=$(($CHARS_LEN/($PARTS)))
SPLIT=$(($PARTS-1))

function split_table() {
    echo $CHARS | perl -pe 's/(\S)/\1\n/g' | LC_ALL=C sort | xargs -n $2 \
    | cut -s -d ' ' -f $2 | head -n $SPLIT | xargs | sed "s/ /','/g" \
    | sed "s/^/create '$1', 'cf', SPLITS => ['/g" | sed "s/$/']/g"
}

split_table $TABLE $RANGE
```

**Demo**

```
# 執行方式
$ ./preSplit.sh {Table Name} {Region Count}

# Sample 1
$ ./preSplit.sh UserTable 3
create 'UserTable', 'cf', SPLITS => ['I','d']

# Sample 2
$ seq 2 10 | xargs -I {} ./preSplit.sh UserTable {}
create 'UserTable', 'cf', SPLITS => ['T']
create 'UserTable', 'cf', SPLITS => ['I','d']
create 'UserTable', 'cf', SPLITS => ['D','T','j']
create 'UserTable', 'cf', SPLITS => ['9','L','X','j']
create 'UserTable', 'cf', SPLITS => ['7','H','R','b','l']
create 'UserTable', 'cf', SPLITS => ['6','F','O','X','g','p']
create 'UserTable', 'cf', SPLITS => ['5','D','L','T','b','j','r']
create 'UserTable', 'cf', SPLITS => ['4','B','I','P','W','d','k','r']
create 'UserTable', 'cf', SPLITS => ['3','9','F','L','R','X','d','j','p']
```

<h3>第二種：Auto Splitting</h3>

<h4>觸發時機</h4>

1. After Memstore Flush
2. After Compact

<h4> Split 策略</h4>

* **ConstantSizeRegionSplitPolicy**：0.94 版本前默認的切割策略。

**程式註解**

```
A RegionSplitPolicy implementation which splits a region as soon as any of its store 
files exceeds a maximum configurable size. 

This is the default split policy. From 0.94.0 on the default split policy has changed 
to IncreasingToUpperBoundRegionSplitPolicy
```

**關鍵程式碼**

```
@Override
protected boolean shouldSplit() {
  boolean force = region.shouldForceSplit();
  boolean foundABigStore = false;

  for (HStore store : region.getStores()) {
    if ((!store.canSplit())) {
       return false;
    }
    if (store.getSize() > desiredMaxFileSize) {
      foundABigStore = true;
    }
  }
  return foundABigStore || force;
}
```

**切分規則**

使用 ConstantSizeRegionSplitPolicy 策略，即 storefile 固定大小策略，當任一個 Storefile 的大小大於 `hbase.hregion.max.filesize(Default: 10G)` 的時候 Region 就會自動切割。

---

* **IncreasingToUpperBoundRegionSplitPolicy**：0.94 ~ 2.0版本間默認的切割策略。

**程式註解**

```
Split size is the number of regions that are on this server that all are of the same table, 
cubed, times 2x the region flush size OR the maximum region split size, whichever is smaller.

For example, if the flush size is 128MB, then after two flushes (256MB) we will split which 
will make two regions that will split when their size is 2^3 * 128MB*2 = 2048MB.

If one of these regions splits, then there are three regions and now the split size is 
3^3 * 128MB*2 = 6912MB, and so on until we reach the configured maximum file size and then 
from there on out, we'll use that.
```

**關鍵程式碼**

```
protected long initialSize;

@Override
protected void configureForRegion(HRegion region) {
  super.configureForRegion(region);
  Configuration conf = getConf();
  initialSize = conf.getLong("hbase.increasing.policy.initial.size", -1);
  if (initialSize > 0) {
    return;
  }
  TableDescriptor desc = region.getTableDescriptor();
  if (desc != null) {
    initialSize = 2 * desc.getMemStoreFlushSize();
  }
  if (initialSize <= 0) {
    initialSize = 2 * conf.getLong(HConstants.HREGION_MEMSTORE_FLUSH_SIZE,
                                   TableDescriptorBuilder.DEFAULT_MEMSTORE_FLUSH_SIZE);
  }
}

@Override
protected boolean shouldSplit() {
  boolean force = region.shouldForceSplit();
  boolean foundABigStore = false;
  int tableRegionsCount = getCountOfCommonTableRegions();
  long sizeToCheck = getSizeToCheck(tableRegionsCount);
  
  for (HStore store : region.getStores()) {
    if (!store.canSplit()) {
      return false;
    }
    long size = store.getSize();
    if (size > sizeToCheck) {
      foundABigStore = true;
    }
  }
  return foundABigStore || force;
}

protected long getSizeToCheck(final int tableRegionsCount) {
  // safety check for 100 to avoid numerical overflow in extreme cases
  return tableRegionsCount == 0 || tableRegionsCount > 100
             ? getDesiredMaxFileSize()
             : Math.min(getDesiredMaxFileSize(),
                        initialSize * tableRegionsCount * tableRegionsCount * tableRegionsCount);
}
```

**切分規則**

使用 IncreasingToUpperBoundRegionSplitPolicy 策略，0.94 (包含)之後默認使用此策略，當 Storefile 大於 getSizeToCheck() 所回傳的值時會進行切割。此策略與 RegionServer 上的 Region 個數有關(假定其個數為 RS)

getSizeToCheck() 的規則如下：

1. 當 RS 等於 0 或大於 100 時，與 ConstantSizeRegionSplitPolicy 策略相同
2. 當 RS 介於 0 ~ 100 之間時，公式如下：
	* Min(`hbase.hregion.max.filesize`, `hbase.increasing.policy.initial.size` * RS ^ 3)

**註解**

1. `hbase.increasing.policy.initial.size` 在未設定的情況下為 `hbase.hregion.memstore.flush.size` * 2
2. `hbase.hregion.memstore.flush.size` 預設值為 128M

---

* **SteppingSplitPolicy**：從 2.0版本開始默認的切割策略。

**關鍵程式碼**

```
public class SteppingSplitPolicy extends IncreasingToUpperBoundRegionSplitPolicy {
  @Override
  protected long getSizeToCheck(final int tableRegionsCount) {
    return tableRegionsCount == 1 ? this.initialSize : getDesiredMaxFileSize();
  }
}
```

**切分規則**

繼承 IncreasingToUpperBoundRegionSplitPolicy 策略，當 Storefile 大於 getSizeToCheck() 所回傳的值時會進行切割。

getSizeToCheck() 的規則如下：

1. 當 Regions 數量為 1 時，回傳 `hbase.increasing.policy.initial.size`
2. 當 Regions 數量不為 1 時，與 ConstantSizeRegionSplitPolicy 策略相同


<h3>第三種：forced splits</h3>

使用 HBase Shell 可以強制切分 Table，方法如下：

**HBase Shell**

```
hbase(main):001:0> split

ERROR: wrong number of arguments (0 for 1)

Here is some help for this command:
Split entire table or pass a region to split individual region.  With the
second parameter, you can specify an explicit split key for the region.
Examples:
    split 'tableName'
    split 'namespace:tableName'
    split 'regionName' # format: 'tableName,startKey,id'
    split 'tableName', 'splitKey'
    split 'regionName', 'splitKey'
```

<h2>相關的配置參數</h2>

1. `hbase.regionserver.regionSplitLimit` 預設值為 Integer.MAX（Cloudera）

控制 RegionServer 的最大 Region 數量，超過則不會進行 Auto Splitting。如果不禁止 Auto Splitting 在每次 flush 或著 compact 之後，RegionServer 都會檢查是否需要 Split，Split 過程老 Region 會先下線在上線 Split 後的 Region，在此期間 Client 訪問會失敗，Retry 過程中會成功但是如果是提供實時服務的系統則想印時長會增加，同時 Split 後的 Compact 是一個比較消耗資源的行為。

2. `hbase.hregion.max.filesize` 預設值為 10G

設定 Region's Store Size 的 Max 值，此參數是主要觸發  Split 的條件之一，建議勿將此值設定過低，否則可能會有過多的零碎的 Region，一般的建議是 5G ~ 10G。



