## Java 之神奇的 1+1+1+1+1 = 1

	注意事項：
		1. 文章內容為閱讀相關資訊後所做的測試結果及心得
		2. 可供參考、分享及測試


下面是某程式的一部分，表達的只是 5 個 1 相加，結果卻會是 1。

	Integer a = 1;
	Integer b = new Integer(1);
	Integer c = Integer.valueOf(1);
	Integer d = Integer.parseInt("1");
	Integer e = Long.valueOf(1).intValue();
	
	System.out.println("a : " + a);
	System.out.println("b : " + b);
	System.out.println("c : " + c);
	System.out.println("d : " + d);
	System.out.println("e : " + e);
	
	System.out.println("a+b+c+d+e : " + (a+b+c+d+e)); // output 'a+b+c+d+e : 1'

在看解答前不仿先嘗試猜猜看原因為何。

.

.

.

.

.

.

.

> 完整程式碼 如下：

	package sample.puzzler;

	import java.lang.reflect.Field;

	/**
	 * Created by Cookie on 15/10/29.
	 */
	public class JavaPuzzler {
	    
	    public static void main(String[] args) throws IllegalAccessException, NoSuchFieldException {

	        Integer one = 1;

	        Field field = Integer.class.getDeclaredField("value");
	        field.setAccessible(true);
	        field.setInt(one, 0);

	        Integer a = 1;
	        Integer b = new Integer(1);
	        Integer c = Integer.valueOf(1);
	        Integer d = Integer.parseInt("1");
	        Integer e = Long.valueOf(1).intValue();

	        System.out.println("a : " + a);
	        System.out.println("b : " + b);
	        System.out.println("c : " + c);
	        System.out.println("d : " + d);
	        System.out.println("e : " + e);

	        System.out.println("a+b+c+d+e : " + (a+b+c+d+e));

	    }
	}

> Console Output

	a : 0
	b : 1
	c : 0
	d : 0
	e : 0
	a+b+c+d+e : 1
	
在程式後半段宣告 a ~ e 並執行 a+b+c+d+e 應該為 5，可執行結果卻為 1，究竟是什麼原因會造成這樣的結果呢？

## 程式解讀

	Field field = Integer.class.getDeclaredField("value");

這是利用 Java 反射機制來取得 Integer 中名叫 value 的欄位。

	field.setAccessible(true);
	field.setInt(one, 0);

再來的兩行，分別是將存取權限改成 true，之後將整數 0 放入 one 這個物件的 value 欄位之中。

	// 宣告 a ~ e
	Integer a = 1;
	Integer b = new Integer(1);
	Integer c = Integer.valueOf(1);
	Integer d = Integer.parseInt("1");
	Integer e = Long.valueOf(1).intValue();
	
	// 輸出每一個變數的數值
	System.out.println("a : " + a);
	System.out.println("b : " + b);
	System.out.println("c : " + c);
	System.out.println("d : " + d);
	System.out.println("e : " + e);
	
	// 計算 a+b+c+d+e 然後輸出到畫面
	System.out.println("a+b+c+d+e : " + (a+b+c+d+e));

後面幾行如一開始所說的是宣告 a ~ e 的變數，之後做輸出及加總後輸出。

## 認識 IntegerCache

IntegerCache 是 Integer 物件中的一個 Inner Class。如其名是用來存放 Integer 的 Cache 以達到加速的效果，IntegerCache 程式碼 內容如下：

	private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);
        }

        private IntegerCache() {}
    }

## 觀察 Integer.valueOf()

	public static Integer valueOf(int i) {
        assert IntegerCache.high >= 127;
        if (i >= IntegerCache.low && i <= IntegerCache.high)
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
    
## 結論

在實作 Integer 的工程師，為了提高 Integer 物件的效能，定義了一段 Cache 的規則，大致上內容為如果所宣告的數值 -128 ~ 127 之間，將直接從 integerCache 中直接取得相對應的物件。

Example：

	Integer one = 1;
	Integer a = 1;
	Integer c = Integer.valueOf(1);
	
上列三個參考皆會指向到相同的 Integer 物件並不會不同，除非是自行宣告為 new Integer(1)，否則是不會產生新物件的。所以當使用`field.setInt(one, 0);`將 one 所指向的物件進行修改。由於 a 和 c 也指在相同物件上自然也就會被修改到。





