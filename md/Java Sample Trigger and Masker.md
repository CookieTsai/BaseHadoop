	注意事項：
		1. 文章內容為使用二進制實現開關效果
		2. 利用一個整數來存取大量開關
		3. 文章及程式碼皆為 CookieTsai 撰寫，可供參考及分享

## 目錄

[TOC]

## 前言

有些時候，在 Coding 時會面臨需要使用大量開關來處理的問題。

在這時候程式碼會使用大量 if else 造成不易閱讀及不易維護。除此之外，在儲存這些設定時可能會需要用到許多欄位。過多的判斷式及欄位將造成維護上的困擾。

因此，這幾天嘗試實作了一個叫 Trigger 還有 Masker 的兩種物件。Trigger and Masker 是利用一個整數來便於管理及儲存這大量的開關的設計。

文章的 Start Demo 是實作 SampleMasker 及 SampleMain 來管理五個開關。

## Defined Trigger and Masker

### Trigger Java Class

	package sample.trigger;

	/**
	 * Created by Cookie on 15/10/29.
	 */
	public class Trigger {
	    public static final int ALL_ON  = -1;
	    public static final int ALL_OFF = 0;

	    private int value;

	    public Trigger(Masker... maskers) {
	        for (Masker masker: maskers) {
	            on(masker);
	        }
	    }

	    public void on(Masker... maskers){
	        for (Masker masker: maskers) {
	            value = (value | masker.getMask());
	        }
	    }

	    public void off(Masker... maskers){
	        for (Masker masker: maskers) {
	            if (!isOn(masker)) {
	                continue;
	            }
	            value -= masker.getMask();
	        }
	    }

	    public boolean isOn(Masker masker){
	        return (value & masker.getMask()) > 0;
	    }

	    public int getValue() {
	        return value;
	    }

	    @Override
	    public String toString() {
	        return String.valueOf(getValue());
	    }
	}

### Masker Interface

	package sample.trigger;

	/**
	 * Created by Cookie on 15/10/29.
	 */
	public interface Masker {
	    boolean isOn(int value);
	    int getIndex();
	    int getMask();
	}
	
## Start Demo

### Sample Masker Java Class

	package sample.trigger;

	/**
	 * Created by Cookie on 15/10/29.
	 */
	public enum  SampleMasker implements Masker {

	    OPTION_01(1),
	    OPTION_02(2),
	    OPTION_03(3),
	    OPTION_04(4),
	    OPTION_05(5);

	    private final int index;
	    private final int mask;

	    SampleMasker(int index) {
	        this.index = index;
	        this.mask = (1 << (index -1));
	    }

	    public int getIndex() {
	        return index;
	    }

	    public int getMask() {
	        return mask;
	    }

	    public boolean isOn(int value){
	        return (value & getMask()) > 0;
	    }

	    public static Trigger newTrigger() {
	        return new Trigger();
	    }

	    public static Trigger newTrigger(Integer value) {
	        if (value == null) {
	            return new Trigger(values());
	        }
	        Trigger result = new Trigger();
	        for (Masker masker: values()) {
	            if (masker.isOn(value)) {
	                result.on(masker);
	            }
	        }
	        return result;
	    }

	    @Override
	    public String toString() {
	        return this.name();
	    }
	}

### Smaple Main Java Class

	package sample.trigger;

	/**
	 * Created by Cookie on 15/10/29.
	 */
	public class SampleMain {

	    public static void main(String[] args) {

	        Trigger trigger;

	        System.out.println("========== Sample 1 output ==========");

	        // Create a new Trigger
	        trigger = SampleMasker.newTrigger();

	        // Open the option 02 and 05
	        trigger.on(SampleMasker.OPTION_02, SampleMasker.OPTION_05);

	        // show all option
	        for (Masker masker: SampleMasker.values()) {
	            System.out.println(masker.toString() + " => " + trigger.isOn(masker) );
	        }

	        // show the trigger value
	        System.out.println("Get Trigger Value => " + trigger.getValue());
	        System.out.println();

	        System.out.println("========== Sample 2 output ==========");

	        // Create a new Trigger,and all option be on
	        trigger = SampleMasker.newTrigger(Trigger.ALL_ON);

	        // Close the option 02 and 05
	        trigger.off(SampleMasker.OPTION_04);

	        for (Masker masker: SampleMasker.values()) {
	            System.out.println(masker.toString() + " => " + trigger.isOn(masker) );
	        }

	        // show the trigger value
	        System.out.println("Get Trigger Value => " + trigger.getValue());
	        System.out.println();

	        int value = trigger.getValue();

	        System.out.println("========== Sample 3 output ==========");

	        // Using a trigger value to create a new Trigger
	        trigger = SampleMasker.newTrigger(value);

	        // show all option
	        for (Masker masker: SampleMasker.values()) {
	            System.out.println(masker.toString() + " => " + trigger.isOn(masker) );
	        }

	        // show the trigger value
	        System.out.println("turn to Int Value => " + trigger.getValue());
	        System.out.println();

	    }
	}

### Console Output
	
	========== Sample 1 output ==========
	OPTION_01 => false
	OPTION_02 => true
	OPTION_03 => false
	OPTION_04 => false
	OPTION_05 => true
	Get Trigger Value => 18

	========== Sample 2 output ==========
	OPTION_01 => true
	OPTION_02 => true
	OPTION_03 => true
	OPTION_04 => false
	OPTION_05 => true
	Get Trigger Value => 23

	========== Sample 3 output ==========
	OPTION_01 => true
	OPTION_02 => true
	OPTION_03 => true
	OPTION_04 => false
	OPTION_05 => true
	turn to Int Value => 23