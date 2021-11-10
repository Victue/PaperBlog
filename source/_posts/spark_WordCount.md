---
title: Spark RDD 11种方式 WordCount
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

### 11.Accumulator
该方式较为复杂, 重写了AccumulatorV2
```scala
val rdd = sc.makeRDD(
      List(
        "scala",
        "scala",
        "scala",
        "scala",
        "scala",
        "spark",
        "spark",
        "spark",
      )
    )

    // TODO 创建累加器
    val acc = new WordCountAccumulator()

    // TODO 向Spark注册
    sc.register(acc, "WordCount")

    rdd.foreach(
      word => {
        // TODO 将单词放入累加器中
        acc.add(word)
      }
    )

    // TODO 获取累加器的结果
    println(acc.value)


    sc.stop()
  }

  // 自定义数据累加器
  // 1.继承AccumulatorV2
  // 2.定义泛型
  //    IN: String
  //    OUT: Map[K, V]
  // 3.重写方法(3+3)
  class WordCountAccumulator extends AccumulatorV2[String, mutable.Map[String, Int]] {

    private val wcMap = mutable.Map[String, Int]()

    // 判断累加器是否为初始状态
    // TODO 3.true
    override def isZero: Boolean = {
      wcMap.isEmpty
    }

    // TODO 1.
    override def copy(): AccumulatorV2[String, mutable.Map[String, Int]] = {
      new WordCountAccumulator()
    }

    // 重置累加器
    // TODO 2.
    override def reset(): Unit = {
      wcMap.clear()
    }

    // 从外部向累加器中添加数据
    override def add(word: String): Unit = {
      val oldCnt = wcMap.getOrElse(word, 0)
      wcMap.update(word, oldCnt + 1)
    }

    override def merge(other: AccumulatorV2[String, mutable.Map[String, Int]]): Unit = {
      other.value.foreach {
        case (word, cnt) => {
          val oldCnt = wcMap.getOrElse(word, 0)
          wcMap.update(word, oldCnt + cnt)
        }
      }
    }

    // 向结果返回到外部
    override def value: mutable.Map[String, Int] = wcMap
  }
```
