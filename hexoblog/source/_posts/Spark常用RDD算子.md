---
title: Spark常用RDD算子
date: 2022-07-10 20:01:11
img: /images/spark_rdd.jpg
cover: true
coverImg: /images/spark_rdd.jpg
categories: hadoop生态圈
tags:
	- Spark
	- RDD
---

## RDD 创建

 在 Spark 中创建 RDD 的创建方式可以分为四种：

 1) 从集合（内存）中创建 RDD

 从集合中创建 RDD，Spark 主要提供了两个方法：parallelize 和 makeRDD

```scala
 val sparkConf = new SparkConf().setMaster("local[*]").setAppName("spark") val sparkContext = new SparkContext(sparkConf) val rdd1 = sparkContext.parallelize( List(1,2,3,4) ) val rdd2 = sparkContext.makeRDD( List(1,2,3,4) ) rdd1.collect().foreach(println) rdd2.collect().foreach(println) sparkContext.stop()
```

从底层代码实现来讲，makeRDD 方法其实就是 parallelize 方法

2) 从外部存储（文件）

创建 RDD 由外部存储系统的数据集创建 RDD 包括：本地的文件系统，所有 Hadoop 支持的数据集， 比如 HDFS、HBase 等。

```scala
 val sparkConf = new SparkConf().setMaster("local[*]").setAppName("spark") val sparkContext = new SparkContext(sparkConf) val fileRDD: RDD[String] = sparkContext.textFile("input") fileRDD.collect().foreach(println) sparkContext.stop() 
```

3) 从其他 RDD 创建 主要是通过一个 RDD 运算完后，再产生新的 RDD。 

4) 直接创建 RDD（new） 使用 new 的方式直接构造 RDD，一般由 Spark 框架自身使用。

## RDD 转换算子 

RDD 根据数据处理方式的不同将算子整体上分为 Value 类型、双 Value 类型和 Key-Value 类型 

### Value 类型

 1) map

将处理的数据逐条进行映射转换，这里的转换可以是类型的转换，也可以是值的转换。

```scala
val dataRDD: RDD[Int] = sparkContext.makeRDD(List(1,2,3,4))
val dataRDD1: RDD[Int] = dataRDD.map(
 num => {
 num * 2
 }
)
val dataRDD2: RDD[String] = dataRDD1.map(
 num => {
 "" + num
 }
)
```

2) mapPartitions 

 将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处 理，哪怕是过滤数据。

```scala
 val dataRDD1: RDD[Int] = dataRDD.mapPartitions( datas => { datas.filter(_==2) } )
```

```scala
a = sc.parallelize(range(10),2)
a.mapPartitions(lambda it:iter([list(it)])).collect()
```

```
[[0, 1, 2, 3, 4], [5, 6, 7, 8, 9]]
```

map 和 mapPartitions 的区别？

 ➢ 数据处理角度

 Map 算子是分区内一个数据一个数据的执行，类似于串行操作。而 mapPartitions 算子 是以分区为单位进行批处理操作。

 ➢ 功能的角度 Map 

算子主要目的将数据源中的数据进行转换和改变。但是不会减少或增多数据。 MapPartitions 算子需要传递一个迭代器，返回一个迭代器，没有要求的元素的个数保持不变， 所以可以增加或减少数据 

➢ 性能的角度

 Map 算子因为类似于串行操作，所以性能比较低，而是 mapPartitions 算子类似于批处 理，所以性能较高。但是 mapPartitions 算子会长时间占用内存，那么这样会导致内存可能 不够用，出现内存溢出的错误。所以在内存有限的情况下，不推荐使用。使用 map 操作。

3) mapPartitionsWithIndex

 将待处理的数据以分区为单位发送到计算节点进行处理，这里的处理是指可以进行任意的处 理，哪怕是过滤数据，在处理时同时可以获取当前分区索引。

```scala
 val dataRDD1 = dataRDD.mapPartitionsWithIndex( (index, datas) => { datas.map(index, _) } )
```

```python
#mapPartitionsWithIndex可以获取两个参数
#即分区id和每个分区内的数据组成的Iterator
a = sc.parallelize(range(11),2)

def func(pid,it):
    s = sum(it)
    return(iter([str(pid) + "|" + str(s)]))
    [str(pid) + "|" + str]
b = a.mapPartitionsWithIndex(func)
b.collect()
```

```
['0|10', '1|45']
```

```
#利用TaskContext可以获取当前每个元素的分区
from pyspark.taskcontext import TaskContext
a = sc.parallelize(range(5),3)
c = a.map(lambda x:(TaskContext.get().partitionId(),x))
c.collect()
```

```python
[(0, 0), (1, 1), (1, 2), (2, 3), (2, 4)]
```

foreachPartition：类似foreach，但每次提供一个Partition的一批数据

```python
#范例：求每个分区内最大值的和
total = sc.accumulator(0.0)
a = sc.parallelize(range(1,101),3)
def func(it):
    total.add(max(it))
a.foreachPartition(func)
total.value
```

199.0



4) flatMap 

 将处理的数据进行扁平化后再进行映射处理，所以算子也称之为扁平映射 

```scala
val dataRDD = sparkContext.makeRDD(List( List(1,2),List(3,4) ),1) val dataRDD1 = dataRDD.flatMap( list => list ) 
//----------
rdd.flatMap(lambda x: range(1,x))

def test(items):
  for item in items:
    yield item+1
rdd.flatMap(test)

```

yield是一个类似return的关键字。当我们调用这个函数的时候并不是返回计算的结果。而是返回一个生成器。只有迭代这个生成器的时候才会计算结果。yield可以返回值，但是不会结束函数的执行，如果函数后面还有代码，同样是可以执行的。

yield的好处

1、不会将所有数据取出来存入内存中，而是返回了一个对象，可以通过对象获取数据，用多少取多少，可以节省内存空间。
2、除了能返回一个值，还不会终止循环的运行,再用yield的写法效率比普通的列表效率高

```python
def test():
  for i in range(1,111000000):
   if i%2 ==0:
    yield i
def test1():
  result =[]
  for i in range(1,111000000):
    if i%2 ==0:
    	result.append(i)
    	return result
```

5) glom 

 将同一个分区的数据直接转换为相同类型的内存数组进行处理，分区不变

```scala
val dataRDD = sparkContext.makeRDD(List(
 1,2,3,4
),1)
val dataRDD1:RDD[Array[Int]] = dataRDD.glom()
```

6) groupBy 

 将数据根据指定的规则进行分组, 分区默认不变，但是数据会被打乱重新组合，我们将这样 的操作称之为 shuffle。极限情况下，数据可能被分在同一个分区中 一个组的数据在一个分区中，但是并不是说一个分区中只有一个组 

```scala
val dataRDD = sparkContext.makeRDD(List(1,2,3,4),1) val dataRDD1 = dataRDD.groupBy( _%2 )
```

7) filter 

将数据根据指定的规则进行筛选过滤，符合规则的数据保留，不符合规则的数据丢弃。 当数据进行筛选过滤后，分区不变，但是分区内的数据可能不均衡，生产环境下，可能会出 现数据倾斜。

```scala
val dataRDD = sparkContext.makeRDD(List( 1,2,3,4 ),1)
val dataRDD1 = dataRDD.filter(_%2 == 0)
```

8) sample 

根据指定的规则从数据集中抽取数据

（1）第一个参数：`withReplacement` 代表一个数据是否能被采样多次；
（2）第二个参数：`fraction` 代表数据采样的比例；

```scala
val dataRDD = sparkContext.makeRDD(List(
 1,2,3,4
),1)
// 抽取数据不放回（伯努利算法）
// 伯努利算法：又叫 0、1 分布。例如扔硬币，要么正面，要么反面。
// 具体实现：根据种子和随机算法算出一个数和第二个参数设置几率比较，小于第二个参数要，大于不
要
// 第一个参数：抽取的数据是否放回，false：不放回
// 第二个参数：抽取的几率，范围在[0,1]之间,0：全不取；1：全取；
// 第三个参数：随机数种子
val dataRDD1 = dataRDD.sample(false, 0.5)
// 抽取数据放回（泊松算法）
// 第一个参数：抽取的数据是否放回，true：放回；false：不放回
// 第二个参数：重复数据的几率，范围大于等于 0.表示每一个元素被期望抽取到的次数
// 第三个参数：随机数种子
val dataRDD2 = dataRDD.sample(true, 2)
```



9) distinct 

 将数据集中重复的数据去重

```scala
val dataRDD = sparkContext.makeRDD(List( 1,2,3,4,1,2 ),1) val dataRDD1 = dataRDD.distinct() val dataRDD2 = dataRDD.distinct(2)//2代表numPartitions: Int
```

10) coalesce 

 根据数据量缩减分区，用于大数据集过滤后，提高小数据集的执行效率 当 spark 程序中，存在过多的小任务的时候，可以通过 coalesce 方法，收缩合并分区，减少 分区的个数，减小任务调度成本,保留原来的数据顺序不变；

```scala
 val dataRDD = sparkContext.makeRDD(List( 1,2,3,4,1,2 ),6) val dataRDD1 = dataRDD.coalesce(2)
//默认shuffle: Boolean = false
```

11) repartition 

该操作内部其实执行的是 coalesce 操作，参数 shuffle 的默认值为 true。无论是将分区数多的 RDD 转换为分区数少的 RDD，还是将分区数少的 RDD 转换为分区数多的 RDD，repartition 操作都可以完成，因为无论如何都会经 shuffle 过程。 

```scala
val dataRDD = sparkContext.makeRDD(List( 1,2,3,4,1,2 ),2) 
val dataRDD1 = dataRDD.repartition(4)
```

12) sortBy

该操作用于排序数据。在排序之前，可以将数据通过 f 函数进行处理，之后按照 f 函数处理 的结果进行排序，默认为升序排列。排序后新产生的 RDD 的分区数与原 RDD 的分区数一 致。中间存在 shuffle 的过程 

```scala
val dataRDD = sparkContext.makeRDD(List( 1,2,3,4,1,2 ),2) 
val dataRDD1 = dataRDD.sortBy(num=>num, false, 4)
//4:numPartitions
```

KeyBy

- 功能说明：创建一个元组，定义某个函数f生成key；

```shell
>>> x = sc.parallelize(range(0,3)).keyBy(lambda x: x*x)
>>> y = sc.parallelize(zip(range(0,5), range(0,5)))
>>> [(x, list(map(list, y))) for x, y in sorted(x.cogroup(y).collect())]
[(0, [[0], [0]]), (1, [[1], [1]]), (2, [[], [2]]), (3, [[], [3]]), (4, [[2], [4]])]
```

repartitionAndSortWithinPartitions：根据给定的分区程序对RDD重新分区，并在每个结果分区中，按其键对记录进行排序。这比repartition在每个分区内调用然后排序更为有效，因为它可以将排序推入洗牌机制

###  双 Value 类型 

13) intersection 

对源 RDD 和参数 RDD 求交集后返回一个新的 RDD 

```scala
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4)) 

val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6))

 val dataRDD = dataRDD1.intersection(dataRDD2) 
```

 14) union 

对源 RDD 和参数 RDD 求并集后返回一个新的 RDD 

```scala
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4)) 

val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6)) 

val dataRDD = dataRDD1.union(dataRDD2)
```

15) subtract 

以一个 RDD 元素为主，去除两个 RDD 中重复元素，将其他元素保留下来。求差集 

```scala
val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4)) 

val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6)) 

val dataRDD = dataRDD1.subtract(dataRDD2)
```

16) zip 

将两个 RDD 中的元素，以键值对的形式进行合并。其中，键值对中的 Key 为第 1 个 RDD 中的元素，Value 为第 2 个 RDD 中的相同位置的元素。

```scala
 val dataRDD1 = sparkContext.makeRDD(List(1,2,3,4)) 

val dataRDD2 = sparkContext.makeRDD(List(3,4,5,6)) 

val dataRDD = dataRDD1.zip(dataRDD2)
```

**zipWithIndex**

```
#将RDD和一个从0开始的递增序列按照拉链方式连接。
rdd_name =  sc.parallelize(["LiLei","Hanmeimei","Lily","Lucy","Ann","Dachui","RuHua"])
rdd_index = rdd_name.zipWithIndex()
print(rdd_index.collect())
```

```
[('LiLei', 0), ('Hanmeimei', 1), ('Lily', 2), ('Lucy', 3), ('Ann', 4), ('Dachui', 5), ('RuHua', 6)]
```

### Key - Value 类型

 17) partitionBy 

将数据按照指定 Partitioner 重新进行分区。Spark 默认的分区器是 HashPartitioner

```scala
 val rdd: RDD[(Int, String)] = sc.makeRDD(Array((1,"aaa"),(2,"bbb"),(3,"ccc")),3) import org.apache.spark.HashPartitioner
val rdd2: RDD[(Int, String)] =
 rdd.partitionBy(new HashPartitioner(2))
```

18) reduceByKey

 可以将数据按照相同的 Key 对 Value 进行聚合

```scala
 val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3))) 
val dataRDD2 = dataRDD1.reduceByKey(_+_) 
val dataRDD3 = dataRDD1.reduceByKey(_+_, 2) //2:numPartitions
```

19) groupByKey

 将数据源的数据根据 key 对 value 进行分组 

```scala
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3))) 
val dataRDD2 = dataRDD1.groupByKey() 
val dataRDD3 = dataRDD1.groupByKey(2) 
val dataRDD4 = dataRDD1.groupByKey(new HashPartitioner(2))
```

reduceByKey 和 groupByKey 的区别？

从 shuffle 的角度：reduceByKey 和 groupByKey 都存在 shuffle 的操作，但是 reduceByKey 可以在 shuffle 前对分区内相同 key 的数据进行预聚合（combine）功能，这样会减少落盘的 数据量，而 groupByKey 只是进行分组，不存在数据量减少的问题，reduceByKey 性能比较 高。

 从功能的角度：reduceByKey 其实包含分组和聚合的功能。GroupByKey 只能分组，不能聚 合，所以在分组聚合的场合下，推荐使用 reduceByKey，如果仅仅是分组而不需要聚合。那 么还是只能使用 groupByKey

20) aggregateByKey

  将数据根据不同的规则进行分区内计算和分区间计算 

```scala
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3))) 
val dataRDD2 = dataRDD1.aggregateByKey(0)(_+_,_+_)
```

取出每个分区内相同 key 的最大值然后分区间相加

```scala
// TODO : 取出每个分区内相同 key 的最大值然后分区间相加
// aggregateByKey 算子是函数柯里化，存在两个参数列表
// 1. 第一个参数列表中的参数表示初始值
// 2. 第二个参数列表中含有两个参数
// 2.1 第一个参数表示分区内的计算规则
// 2.2 第二个参数表示分区间的计算规则
val rdd =
 sc.makeRDD(List(
 ("a",1),("a",2),("c",3),
 ("b",4),("c",5),("c",6)
 ),2)
// 0:("a",1),("a",2),("c",3) => (a,10)(c,10)
// => (a,10)(b,10)(c,20)
// 1:("b",4),("c",5),("c",6) => (b,10)(c,10)
val resultRDD =
 rdd.aggregateByKey(10)(
 (x, y) => math.max(x,y),
 (x, y) => x + y
 )
resultRDD.collect().foreach(println)
```

21) foldByKey 

 当分区内计算规则和分区间计算规则相同时，aggregateByKey 就可以简化为 foldByKey 

```scala
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3)))
 val dataRDD2 = dataRDD1.foldByKey(0)(_+_)
```

22) combineByKey 

最通用的对 key-value 型 rdd 进行聚集操作的聚集函数（aggregation function）。类似于 aggregate()，combineByKey()允许用户返回值的类型与输入不一致。 

小练习：将数据 List(("a", 88), ("b", 95), ("a", 91), ("b", 93), ("a", 95), ("b", 98)),求每个 key 的平 均值

```scala
//    CombineByKey:当计算时，发现数据结构不满足要求时，
// 可以让第一个数据转换结构。分区内和分区间计算规则不相同。
val list: List[(String, Int)] = List(("a", 88), ("b", 95), ("a", 91), ("b", 93),  ("a", 95), ("b", 98)) 
val input: RDD[(String, Int)] = sc.makeRDD(list, 2) 
val combineRdd: RDD[(String, (Int, Int))] = input.combineByKey( 
v=>(v,1),
      (t, v) => {
        (t._1 + v, t._2 + 1)//(总量，次数)
      },
      (t1, t2) => {
        (t1._1 + t2._1, t1._2 + t2._2)
      }
)
 val resultRDD: RDD[(String, Int)] = newRDD.mapValues {
      case (num, cnt) => {
        num / cnt
      }
    }
```

reduceByKey、foldByKey、aggregateByKey、combineByKey 的区别？

 reduceByKey: 相同 key 的第一个数据不进行任何计算，分区内和分区间计算规则相同

 FoldByKey: 相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规则相 同

AggregateByKey：相同 key 的第一个数据和初始值进行分区内计算，分区内和分区间计算规 则可以不相同

 CombineByKey:当计算时，发现数据结构不满足要求时，可以让第一个数据转换结构。分区 内和分区间计算规则不相同。

23) sortByKey 

def sortByKey(ascending: Boolean = true, numPartitions: Int = self.partitions.length) : RDD[(K, V)]

在一个(K,V)的 RDD 上调用，K 必须实现 Ordered 接口(特质)，返回一个按照 key 进行排序 的 

val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3))) 

 val sortRDD1: RDD[(String, Int)] = dataRDD1.sortByKey(false)

24) join 

在类型为(K,V)和(K,W)的 RDD 上调用，返回一个相同 key 对应的所有元素连接在一起的 (K,(V,W))的 RDD 

```scala
val rdd: RDD[(Int, String)] = sc.makeRDD(Array((1, "a"), (2, "b"), (3, "c"))) 

val rdd1: RDD[(Int, Int)] = sc.makeRDD(Array((1, 4), (2, 5), (3, 6))) rdd.join(rdd1).collect().foreach(println)
```

25) leftOuterJoin 

类似于 SQL 语句的左外连接 

```scala
 val dataRDD1 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3))) 

val dataRDD2 = sparkContext.makeRDD(List(("a",1),("b",2),("c",3))) 

val rdd: RDD[(String, (Int, Option[Int]))] = dataRDD1.leftOuterJoin(dataRDD2)
```

26) cogroup 

 在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable,Iterable))类型的 RDD 

```scala
val dataRDD1 = sparkContext.makeRDD(List(("a",1),("a",2),("c",3))) 
val dataRDD2 = sparkContext.makeRDD(List(("a",1),("c",2),("c",3))) 
val value: RDD[(String, (Iterable[Int], Iterable[Int]))] =  dataRDD1.cogroup(dataRDD2)
```

```scala
x = sc.parallelize([("a",1),("b",2),("a",3)])
y = sc.parallelize([("a",2),("b",3),("b",5)])
result = x.cogroup(y).collect()
print(result)
print(list(result[0][1][0]))
```

```
[('a', (<pyspark.resultiterable.ResultIterable object at 0x7f55be4a48d0>, <pyspark.resultiterable.ResultIterable object at 0x7f55be4a4780>)), ('b', (<pyspark.resultiterable.ResultIterable object at 0x7f55be4a4a20>, <pyspark.resultiterable.ResultIterable object at 0x7f55be4adf28>))]
[1, 3]
```

cartesian：笛卡尔积，在类型T和U的数据集上调用时，返回（T，U）对（所有元素对）的数据集

```scala
boys = sc.parallelize(["LiLei","Tom"])
girls = sc.parallelize(["HanMeiMei","Lily"])
boys.cartesian(girls).collect()
```

```
[('LiLei', 'HanMeiMei'),
 ('LiLei', 'Lily'),
 ('Tom', 'HanMeiMei'),
 ('Tom', 'Lily')]
```

pipe：通过shell命令（例如Perl或bash脚本）通过管道传输RDD的每个分区。将RDD元素写入进程的stdin，并将输出到其stdout的行作为字符串的RDD返回

## RDD 行动算子

 1) reduce 

聚集 RDD 中的所有元素，先聚合分区内数据，再聚合分区间数据

 val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4)) // 聚合数据

 val reduceResult: Int = rdd.reduce(_+_) 

2) collect 

 在驱动程序中，以数组 Array 的形式返回数据集的所有元素

```scala
 val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4)) // 收集数据到 Driver 
rdd.collect().foreach(println)
```

3) count 

 返回 RDD 中元素的个数 

val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4)) // 返回 RDD 中元素的个数

```scala
 val countResult: Long = rdd.count() 
```

4) first

```scala
//返回 RDD 中的第一个元素
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val firstResult: Int = rdd.first()
println(firstResult)
```

5) take

 返回一个由 RDD 的前 n 个元素组成的数组

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 返回 RDD 中元素的个数
val takeResult: Array[Int] = rdd.take(2)
println(takeResult.mkString(","))
```

6) takeOrdered 

 返回该 RDD 排序后的前 n 个元素组成的数组

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,3,2,4))
// 返回 RDD 中元素的个数
val result: Array[Int] = rdd.takeOrdered(2)
```

7) aggregate 

 分区的数据通过初始值和分区内的数据进行聚合，然后再和初始值进行分区间的数据聚合 

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4), 8) // 将该 RDD 所有元素相加得到结果 //val result: Int = rdd.aggregate(0)(_ + _, _ + _) 
val result: Int = rdd.aggregate(10)(_ + _, _ + _)
```

```python
#aggregate比较复杂，先对每个分区执行一个函数，再对每个分区结果执行一个合并函数。
#例子：求元素之和以及元素个数
#三个参数，第一个参数为初始值，第二个为分区执行函数，第三个为结果合并执行函数。
rdd = sc.parallelize(range(1,21),3)
def inner_func(t,x):
    return((t[0]+x,t[1]+1))

def outer_func(p,q):
    return((p[0]+q[0],p[1]+q[1]))

rdd.aggregate((0,0),inner_func,outer_func)
```

```python
#aggregate比较复杂，先对每个分区执行一个函数，再对每个分区结果执行一个合并函数。
#例子：求元素之和以及元素个数
#三个参数，第一个参数为初始值，第二个为分区执行函数，第三个为结果合并执行函数。
rdd = sc.parallelize(range(1,21),3)
def inner_func(t,x):
    return((t[0]+x,t[1]+1))

def outer_func(p,q):
    return((p[0]+q[0],p[1]+q[1]))

rdd.aggregate((0,0),inner_func,outer_func)
```

(210,20)

8) fold 

折叠操作，aggregate 的简化版操作

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1, 2, 3, 4)) val foldResult: Int = rdd.fold(0)(_+_)
```

9) countByKey 

 统计每种 key 的个数

```scala
val rdd: RDD[(Int, String)] = sc.makeRDD(List((1, "a"), (1, "a"), (1, "a"), (2, 
"b"), (3, "c"), (3, "c")))
// 统计每种 key 的个数
val result: collection.Map[Int, Long] =rdd.countByKey()
```

10) save 相关算子 

 将数据保存到不同格式的文件中

```scala
// 保存成 Text 文件
rdd.saveAsTextFile("output")
// 序列化成对象保存到文件
rdd.saveAsObjectFile("output1")
// 保存成 Sequencefile 文件
rdd.map((_,1)).saveAsSequenceFile("output2")
```

控制返回文件数量，通常情况下返回文件为一个文件夹下的多个文件，可以使用SparkContext.wholeTextFiles控制返回文件的个数，例如返回一个文件
SparkContext.sequenceFile[Int, String]
SparkContext.hadoopRDD
SparkContext.objectFile



11) foreach 

➢ 函数签名 def foreach(f: T => Unit): Unit = withScope { val cleanF = sc.clean(f) sc.runJob(this, (iter: Iterator[T]) => iter.foreach(cleanF)) } 

分布式遍历 RDD 中的每一个元素，调用指定函数

```scala
val rdd: RDD[Int] = sc.makeRDD(List(1,2,3,4))
// 收集后打印
rdd.map(num=>num).collect().foreach(println)
println("****************")
// 分布式打印
rdd.foreach(println)
```

collectAsMap函数

- 功能说明：将rdd中的数据collect出来，rdd中元组第一个元素为key，第二个元素为value；

```shell
>>> m = sc.parallelize([(1, 2), (3, 4)]).collectAsMap()
>>> m[1]
2
>>> m[3]
4
```

takeSample：返回一个数组，该数组包含数据集num个元素的随机样本（是否替换），可以选择预先指定随机数生成器种子

可以随机取若干个到Driver,第一个参数设置是否放回抽样

```python
rdd = sc.parallelize(range(10),5) 
sample_data = rdd.takeSample(False,10,0)
```

```python
#subtractByKey去除x中那些key也在y中的元素
x = sc.parallelize([("a",1),("b",2),("c",3)])
y = sc.parallelize([("a",2),("b",(1,2))])
x.subtractByKey(y).collect()
```

```
[('c', 3)]
```

## 缓存

如果一个rdd被多个任务用作中间量，那么对其进行cache缓存到内存中对加快计算会非常有帮助。

声明对一个rdd进行cache后，该rdd不会被立即缓存，而是等到它第一次被计算出来时才进行缓存。

persist：可以根据参数进行不同级别的缓存MEMORY_ONLY**（仅在内存中）**、MEMORY_AND_DISK**在内存和磁盘**、MEMORY_ONLY_SER**仅在内存中，但是经过序列化）**、MEMORY_AND_DISK_SER**（序列化后存于内存和磁盘）**、DISK_ONLY**（序列化后存于磁盘）**、OFF_HEAP（堆外存储）
cache：默认缓存级别MEMORY_ONLY
缓存级别选择：MEMORY_ONLY>MEMORY_ONLY_SER>MEMORY_AND_DISK

如果一个RDD后面不再用到，可以用unpersist释放缓存，unpersist是立即执行的。

缓存数据不会切断血缘依赖关系，这是因为缓存数据某些分区所在的节点有可能会有故障，例如内存溢出或者节点损坏。

这时候可以根据血缘关系重新计算这个分区的数据。

```python
#cache缓存到内存中，使用存储级别 MEMORY_ONLY。
#MEMORY_ONLY意味着如果内存存储不下，放弃存储其余部分，需要时重新计算。
a = sc.parallelize(range(10000),5)
a.cache()
sum_a = a.reduce(lambda x,y:x+y)
cnt_a = a.count()
mean_a = sum_a/cnt_a
print(mean_a)
4999.5
```

```python
#persist缓存到内存或磁盘中，默认使用存储级别MEMORY_AND_DISK
#MEMORY_AND_DISK意味着如果内存存储不下，其余部分存储到磁盘中。
#persist可以指定其它存储级别，cache相当于persist(MEMORY_ONLY)
from  pyspark.storagelevel import StorageLevel
a = sc.parallelize(range(10000),5)
a.persist(StorageLevel.MEMORY_AND_DISK)
sum_a = a.reduce(lambda x,y:x+y)
cnt_a = a.count()
mean_a = sum_a/cnt_a

a.unpersist() #立即释放缓存
print(mean_a)
```



## 共享变量

广播变量是不可变变量，实现在不同节点不同任务之间共享数据。

广播变量在每个机器上缓存一个只读的变量，而不是为每个task生成一个副本，可以减少数据的传输。

累加器主要是不同节点和Driver之间共享变量，只能实现计数或者累加功能。

累加器的值只有在Driver上是可读的，在节点上不可见。

广播变量：在所有节点上创建一个只读变量，在使用时不应该调用函数中的指定变量值，而是直接使用指定广播变量，而且防止修改节点上的广播变量，dataFrame和变量都可以使用broadcast进行广播，但是rdd不可以
val broadcastVar = sc.broadcast(Array(1, 2, 3))

val broadcastDF = functions.broadcast(df)

```python
#广播变量 broadcast 不可变，在所有节点可读
broads = sc.broadcast(100)
rdd = sc.parallelize(range(10))
print(rdd.map(lambda x:x+broads.value).collect())

print(broads.value)
```

```
[100, 101, 102, 103, 104, 105, 106, 107, 108, 109]
100
```

累加器
创建累加器

```python
#累加器 只能在Driver上可读，在其它节点只能进行累加
total = sc.accumulator(0)
rdd = sc.parallelize(range(10),3)

rdd.foreach(lambda x:total.add(x))
total.value
```

45

```python
# 计算数据的平均值
rdd = sc.parallelize([1.1,2.1,3.1,4.1])
total = sc.accumulator(0)
count = sc.accumulator(0)

def func(x):
    total.add(x)
    count.add(1)    
rdd.foreach(func)
total.value/count.value
```

2.6





参考sparkAPI链接：https://spark.apache.org/docs/latest/api/scala/org/apache/spark/index.html

