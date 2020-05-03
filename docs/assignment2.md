# Assignment 2 - Map Reduce

In this assignment we are going to take a look at the "Hello World" of map reduce: Word count. We do this using HDFS. First we have to format our filesystem and create a user directory. We do this using the `dfs` command, this command lets me use filesystem commands on the hadoop supported file system. Once this is done we can do some map reduce! 

## Counting Romeo & Juliet

Using map reduce I can easily count the occurences of all words in the complete works of Shakespeare. We do this using the following mapper:

```java

public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }
  
  ```
  
It looks at al tokens (words) of an input text, and combines it with the value 1 into a key-value pair (word, 1). After the text has been processed it is sent to the next reducer:

```java
public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
  ```
  
This reducer sums up all the values corresponding to the same key, and thus giving us the word counts of the input. Using this we could determine whether Romeo or Juliet occurs more in the work of Shakespeare. When taking every version of Romeo and Juliet into account (like 'Romeo, Romeo etc.) we find that Romeo occurs a whopping 313 time while Julliet stays behind with only 211 occurences.

After this I decided to find out which letter occurs the most in the text. To do so the mapper needed a makeover:

```java

public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      String line = value.toString();
      char[] carr = line.toCharArray();
      for (charc : carr) {  
      context.write(new Text(String.valueOf(c)), one);        
      }
    }
  }

```

Using this mapper we transform the text in an array of all of its characters and loop over this array, the output of the mapper is structured exactly the same as before, therefore we can use the same reducer. Using this mapper I found that the letter "e" is the most used letter in shakespeare's work, no surprises there!

[Home](https://rubigdata.github.io/big-data-blog-2020-sweersr/)



