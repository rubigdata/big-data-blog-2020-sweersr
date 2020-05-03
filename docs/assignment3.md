# Lets Spark things up!

In this blogpost we will take a first look into spark, specifically we will look into the Resilient Distributed Datasets (RDD) and lazy evaluation used by Spark.

## The RDD
Spark is designed specifically for 'Big Data' projects and uses Resilient Distributed Datasets to do so. The RDD is a dataset home to Spark that partitions its data such that parallelization can be used to up processing speed easily.

## Lazy evaluation
Spark makes a difference between so called 'actions' and 'transformations', The actions produce actual output whilst the transformations morph our dataset into a different shape. An action one can think of would be something like ``.count``, while a transformation could be something like ``map(s => s.length)`` which would map input strings into its length.

The lazyness of Spark comes from the fact that no work is actually done untill an action is encountered. This way Spark can decide on its own how (and if) all transformations must be applied. Which does not only make it run potentially faster, but no memory is used for intermediate steps.

## Shakespear again
Like in the [last blog](https://rubigdata.github.io/big-data-blog-2020-sweersr/assignment2.html) we are going to count words in the works of shakespear again, only this time using Spark. As we can see this looks way easier than in Java:

```scala
val words = lines.flatMap(line => line.split(" "))
              .filter(_ != "")
              .map(word => (word,1))
              
val wc = words.reduceByKey(_ + _)
```

Again like when using Java we have a mapper, and a reducer, however they are way easier in use. In the mapper we see the following steps happening: First each line in the text get split on spaces, secondly we filter out all "words" that are blank, and finally we map all words to a key value pair. We see both the ``.flatMap`` and ``.map`` occuring they have similar uses however the first of the two can map one input to multiple outputs, where the second always has a 1 to 1 relation.

After this mapper, we have just a single line reducer which simply takes all key value pairs by key and adds the values (which are all 1).

If we now for instance want to see the 10 most occuring words we can do this via:

```scala
val top10 = wc.takeOrdered(10)(Ordering[Int].reverse.on(x=>x._2))
```

Which shows us "the" isthe most occuring word in all of shakespears work.

Last week we already counted the occurences of Romeo and Juliet, so we take Macbeth this time. We can do so by using:

```scala
wc.filter(_._1 == "Macbeth").collect
```

However this ignores all occurences of the name followed (or preceded) by special characters or missing capital letters. To overcome this problem we simply change the wordcount in a way such that all letters are transformed to lowercase and we use a smart RegEx to delete all special characters:

```scala
val words = lines.flatMap(line => line.split(" "))
              .map(w => w.toLowerCase().replaceAll("(^[^a-z]+|[^a-z]+$)", ""))
              .filter(_ != "")
              .map(w => (w,1))
              .reduceByKey( _ + _ )
```

If we now use the same filter with "macbeth" instead of "Macbeth" we find 285 occurences of Macbeth in stead of the 30 we would have found otherwise.

Thanks for reading and stay healthy!

[Home](https://rubigdata.github.io/big-data-blog-2020-sweersr/)
