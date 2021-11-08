---
title: Spark RDD 10种方式 WordCount
author: Li Pie
categories: big data
tags:
- Scala
- Spark
- RDD
date: 2021-11-08
---

### 1. groupBy

```scala
// 读取文件
val lines = sc.textFile("data/word1.txt")
// 分词
val words = lines.flatMap(_.split(" "))
//分组
val wordGroup = words.groupBy(word => word)
// 计数
val wordCount = wordGroup.mapValues(_.size)

wordCount.collect().foreach(println)
```

### 2. reduceByKey

```scala
val words = lines.flatMap(_.split(" "))
val word = words.map((_,1))
val wordCount = word.reduceByKey(_ + _)
wordCount.collect().foreach(println)
```

### 3. groupByKey

```scala
val groupRDD = word.groupByKey()
val wordCount = groupRDD.mapValues(_.size)
```

### 4. aggregateByKey

```scala
val wordCount = word.aggregateByKey(0)(_+_,_+_)
```

### 5. foldByKey

```scala
val wordCount = word.foldByKey(0)(_+_)
```

### 6. combineByKey

```scala
val wordCount = word.combineByKey(
  num => num,
  (x: Int, y) => {
    x + y
  },
  (x: Int, y: Int) => {
    x + y
  }
)
```

### 7. countByKey

```scala
val wordCount = word.countByKey()
```

### 8. countByValue

```scala
val wordCount = words.countByValue()
```

### 9. cogroup

```scala
val cog_word = word.cogroup(word)
// value
// (Hello,(CompactBuffer(1, 1, 1, 1),CompactBuffer(1, 1, 1, 1)))
// (Scala,(CompactBuffer(1, 1),CompactBuffer(1, 1)))
// (Spark,(CompactBuffer(1, 1, 1),CompactBuffer(1, 1, 1)))
val wordCount = cog_word.map {
  case (x, (y, z)) => {
    (x, y.sum)
  }
}
```

###  10. reduce

```scala
val word = words.map(
  x => {
    mutable.Map((x, 1))
  }
)

val wordCount = word.reduce(
  (x, y) => {
    y.foreach{
      case(word, count) =>{
        val newCount = x.getOrElse(word, 0) + count
        x.update(word, newCount)
      }
    }
    x
  }
)
```
