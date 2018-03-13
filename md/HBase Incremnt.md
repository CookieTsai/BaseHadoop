## HBase Incremnt

	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.hbase.HBaseConfiguration;
	import org.apache.hadoop.hbase.client.*;
	import org.apache.hadoop.hbase.util.Bytes;
	import java.io.IOException;

	public class Conn {

	    private static HTable table;

	    static {
	        try {
	            Configuration conf = HBaseConfiguration.create();
	            table = new HTable(conf, "t1");
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	    }

	    public static void main(String[] args) throws IOException {

	        Runnable runnable = new Runnable() {
	            public void run() {
	                try {
	                    Increment incr = new Increment(Bytes.toBytes("row1"));  
	                    // Create Increment and Set RowKey
	                    incr.addColumn(Bytes.toBytes("cf"), Bytes.toBytes("lct"),1l); 
	                    // Amount Column Family
	                    Result res = table.increment(incr);
	                    System.out.println(Bytes.toLong(res.value()) + ":" + 
	                                       System.currentTimeMillis());
	                } catch (IOException e) {
	                    e.printStackTrace();
	                }
	            }
	        };
	        for (int i = 0; i<1000;i++) {
	            new Thread(runnable).start();
	        }
	    }
	}
