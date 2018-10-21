---
layout: post
title:  "Hadoop In-Mapper Combiner"
image: ''
date:   2018-10-21 00:12:00
tags:
- Hadoop
- Java
description: 'Optimizing combining process in Hadoop'
categories:
- Hadoop
---

## Abstract

Hadoop has a Traditional Combiner which **writes** it's Iterables to memory. In-Mapper is basically moving parts of this **writes** to local aggregation, which optimizes the running time since it has <span style="color:orange">less write</span>. However, due to states being perserved within the mapper (local), it causes large memory overhead. To put it simply you may think that In-Mapper Combiner takes up <span style="color:blue">more space</span>, but in return takes <span style="color:red">less time</span>.

Most likely programmers will have to tweak parts of other parts of the MapReduce process, but in this example it is a simple Word Length Count, so it's not required here.

---

## Traditional Combiner

Let...

**Input** be a simple text (i.e. "The brown fox jumps over the lazy dog")

**Output** be a tuple of length of the word and the number of counts (i.e. (3,4), (4,2), (5,2))

{% highlight java %}
public static class Map extends Mapper<Object, Text, Text, IntWritable> {

    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        StringTokenizer itr = new StringTokenizer(value.toString());
        while (itr.hasMoreTokens()) {
            String token = itr.nextToken();
            int length = token.length();
            word.set(length+"");
            context.write(word, one);
        }
    }
}
{% endhighlight %}

The code is quite straight forward except the fact I'm passing Text type instead of IntWritable. It's possible to utilize only IntWritables instead of Text and IntWritable, but I was modifying the traditional WordCount java program, so I just kept the Text type. (you can see at "word.set(length+"")" that I'm overriding an int type to a string so that it can be set as a Text type.). If you are thinking of making the most optimal code, I suggest you pass IntWritables only instead.

---

## Modifying Traditional Combiner to In-Mapper Combiner (Abstract)

Pseudocode might help here.

{% highlight text %}
class Map
    initialize count ← new AssociativeArray (using HashMap here)

    method map
        if first time seeing current length
            count = 1
        if not the first time seeing current length
            count += 1

    method cleanup
        for all tuples inside the Associative Array
            emit word length, word length count
{% endhighlight %}

You need to have the cleanup, else you will run into troubles because 1. the partial map that has been used before, it might aggregate in a wrong manner on the next. 2. you want to flush out accumulation of results as the map finishes. I feel like both 1 and 2 are pretty much the same, but trust me, if you don't do a cleanup method it's likely to have bugs.

---

## Modifying Traditional Combiner to In-Mapper Combiner (Code)

{% highlight java %}
public static class Map extends Mapper<Object, Text, Text, IntWritable> {
    Map count = new HashMap<Integer, Integer>();

    public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
        StringTokenizer itr = new StringTokenizer(value.toString());

        while (itr.hasMoreTokens()) {
            String token = itr.nextToken();
            int length = token.length();

            if(count.containsKey(length)) {
                int sum = (int) count.get(length) + 1;
                count.put(length, sum);
            }
            else {
                count.put(length, 1);
            }
        }
    }

    public void cleanup(Context context) throws IOException, InterruptedException {
        Iterator<Map.Entry<Integer, Integer>> temp = count.entrySet().iterator();

        while(temp.hasNext()) {
            Map.Entry<Integer, Integer> entry = temp.next();
            String keyVal = entry.getKey()+"";
            Integer countVal = entry.getValue();

            context.write(new Text(keyVal), new IntWritable(countVal));
        }
    }
}
{% endhighlight %}

As you can see HashMap is used to do local aggregation. After the local aggregation is finished, the cleanup method will go through the HashMap with Iterator to flush out the accumulated results. As I mentioned there is type overriding in some places due to the same reason I have stated previously.

---

## Final Thoughts

It's actually possible to see the actual time reduced when you run the In-Mapper Combiner. For a 40MB plain text file in a 1 namenode + 2 datanode distributed environment, the Traditional Combiner process was

{% highlight text %}
18/10/21 09:12:31 INFO mapreduce.Job: Running job: job_1538995042638_0062
18/10/21 09:12:44 INFO mapreduce.Job:  map 0% reduce 0%
18/10/21 09:13:00 INFO mapreduce.Job:  map 22% reduce 0%
18/10/21 09:13:03 INFO mapreduce.Job:  map 36% reduce 0%
18/10/21 09:13:09 INFO mapreduce.Job:  map 56% reduce 0%
18/10/21 09:13:10 INFO mapreduce.Job:  map 73% reduce 0%
18/10/21 09:13:12 INFO mapreduce.Job:  map 77% reduce 0%
18/10/21 09:13:15 INFO mapreduce.Job:  map 83% reduce 0%
18/10/21 09:13:16 INFO mapreduce.Job:  map 100% reduce 0%
18/10/21 09:13:17 INFO mapreduce.Job:  map 100% reduce 100%
18/10/21 09:13:18 INFO mapreduce.Job: Job job_1538995042638_0062 completed successfully
{% endhighlight %}

The In-Mapper Combiner process was

{% highlight text %}
18/10/21 09:14:31 INFO mapreduce.Job: Running job: job_1538995042638_0063
18/10/21 09:14:41 INFO mapreduce.Job:  map 0% reduce 0%
18/10/21 09:14:53 INFO mapreduce.Job:  map 50% reduce 0%
18/10/21 09:14:54 INFO mapreduce.Job:  map 100% reduce 0%
18/10/21 09:14:59 INFO mapreduce.Job:  map 100% reduce 100%
18/10/21 09:15:00 INFO mapreduce.Job: Job job_1538995042638_0063 completed successfully
{% endhighlight %}

47 seconds and 29 seconds. Quite an improvement eh?

However, I'd like to re-highlight the part that local aggregation **will use more memory overhead**. I mean I'm pretty sure if you have coded before there's a balance between space and time. When one is improving the other one is probably diminishing. It's not a strict 1:1 trade, so if given infinite values for both, you can do whatever that suits your needs, but in real life, you will have limitation and importance that you will have to balance to get it just right ;).
