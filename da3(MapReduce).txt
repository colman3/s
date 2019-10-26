import java.io.*;
import java.util.*;

import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapred.FileInputFormat;
import org.apache.hadoop.mapred.JobClient;
import org.apache.hadoop.mapred.JobConf;
import org.apache.hadoop.mapred.MapReduceBase;
import org.apache.hadoop.mapred.Mapper;
import org.apache.hadoop.mapred.OutputCollector;
import org.apache.hadoop.mapred.Reducer;
import org.apache.hadoop.mapred.Reporter;
import org.apache.hadoop.mapred.TextInputFormat;
import org.apache.hadoop.mapred.FileOutputFormat;
import org.apache.hadoop.mapred.TextOutputFormat;

public class WordCount
{
	public static class MyMap extends MapReduceBase implements Mapper<LongWritable,Text,Text,IntWritable>
	{
		Text mykey = new Text();
		
		public void map(LongWritable arg0, Text arg1, OutputCollector<Text, IntWritable> arg2, Reporter arg3) throws IOException 
		{
			String line = arg1.toString();
			StringTokenizer tokenizer = new StringTokenizer(line);
			
			while(tokenizer.hasMoreTokens())
			{
				mykey.set(tokenizer.nextToken());
				arg2.collect(mykey, new IntWritable(1));
			}
		}	
	}
	
	public static class MyReduce extends MapReduceBase implements Reducer<Text, IntWritable, Text, IntWritable>
	{
		public void reduce(Text arg0, Iterator<IntWritable> arg1, OutputCollector<Text, IntWritable> arg2, Reporter arg3) throws IOException 
		{
			int sum = 0;
			
			while(arg1.hasNext())
				sum += arg1.next().get();
			
			arg2.collect(arg0, new IntWritable(sum));
		}
	}
	
	public static void main(String[] args) throws Exception
	{
		JobConf conf = new JobConf(WordCount.class);
		conf.setJobName("MyFirstProgram");
		
		conf.setMapperClass(MyMap.class);
		conf.setReducerClass(MyReduce.class);
		
		conf.setOutputKeyClass(Text.class);
		conf.setOutputValueClass(IntWritable.class);
		
		conf.setInputFormat(TextInputFormat.class);
		conf.setOutputFormat(TextOutputFormat.class);
		
		FileInputFormat.setInputPaths(conf, new Path(args[0]));
		FileOutputFormat.setOutputPath(conf, new Path(args[1]));
		
		JobClient.runJob(conf);
	}
}
