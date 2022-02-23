---
title: "Spark aggregation with native API's" 
date : "2022-02-24"
---
## Table of contents
1. Spark aggregation Overview
2. TypedImperativeAggregate[T] abstract class
3. Example


## Spark aggregation Overview

* [User Defined Aggregate Functions](https://spark.apache.org/docs/latest/sql-ref-functions-udf-aggregate.html) can be used. But are restrictive and require workarounds even for  basic requirements.
* Aggregates are unevaluable expressions and cannot have eval and doGenCode method.
* Basic requirement would be to use user defined java objects as internal spark aggregation buffer type.
* And, passing extra arguments to aggregates e.g aggregate(col, 0.24)
* Spark provides **TypedImperativeAggregate[T]** contract for such requirement (imperative as in expressed in terms of imperative initialize, update, and merge methods).

## TypedImperativeAggregate[T] abstract class
``` Scala
case TestAggregation(child: Expression) 
  extends TypedImperativeAggregate[T]  {

  // Check input types
  override def checkInputDataTypes(): TypeCheckResult

  // Initialize T
  override def createAggregationBuffer(): T

  // Update T with row
  override def update(buffer: T, inputRow: InternalRow): T

  // Merge Intermediate buffers onto first buffer
  override def merge(buffer: T, other: T): T
 
  // Final value
  override def eval(buffer: T): Any 

  override def withNewMutableAggBufferOffset(newOffset: Int): TestAggregation 

  override def withNewInputAggBufferOffset(newOffset: Int): TestAggregation 

  override def children: Seq[Expression]

  override def nullable: Boolean

  // Datatype of output
  override def dataType: DataType

  override def prettyName: String

  override def serialize(obj: T): Array[Byte] 

  override def deserialize(bytes: Array[Byte]): T 
}
```

## Example

* `case class Average` holds count and sum of elements and also acts as internal aggregate buffer.
* Aggregate takes in a numeric column and an extra argument n and return avg(column) * n.
* In SparkSQL this will look like :
```SQL
SELECT multiply_average(salary, 2) as average_salary FROM employees
```
* Spark alchemy's NativeFunctionRegistration is used to register functions to spark.
* Aggregate Code : 
``` Scala
import com.swoop.alchemy.spark.expressions.NativeFunctionRegistration
import org.apache.spark.sql.SparkSession
import org.apache.spark.{SparkConf, SparkContext}
import org.apache.spark.sql.catalyst.InternalRow
import org.apache.spark.sql.catalyst.analysis.TypeCheckResult
import org.apache.spark.sql.catalyst.expressions._
import org.apache.spark.sql.catalyst.expressions.aggregate.TypedImperativeAggregate
import org.apache.spark.sql.types._

import java.io.{ByteArrayInputStream, ByteArrayOutputStream, ObjectInputStream, ObjectOutputStream}


case class Average(var sum: Long, var count: Long)

case class AvgTest(
                  child: Expression,
                  nExpression : Expression,
                  override val mutableAggBufferOffset: Int = 0,
                  override val inputAggBufferOffset: Int = 0)
  extends TypedImperativeAggregate[Average]  {

//  private lazy val n: Long = nExpression.eval().asInstanceOf[Long]
  def this(child: Expression) = this(child, Literal(1), 0, 0)
  def this(child: Expression, nExpression: Expression) = this(child, nExpression, 0, 0)

  override def checkInputDataTypes(): TypeCheckResult = {
    child.dataType match {
      case LongType => TypeCheckResult.TypeCheckSuccess
      case _ => TypeCheckResult.TypeCheckFailure(s"$prettyName only supports long input")
    }
  }

  override def createAggregationBuffer(): Average = {
    new Average(0, 0)
  }

  override def update(buffer: Average, inputRow: InternalRow): Average = {
    val value = child.eval(inputRow)
    buffer.sum += value.asInstanceOf[Long]
    buffer.count += 1
    buffer
  }

  override def merge(buffer: Average, other: Average): Average = {
    buffer.sum += other.sum
    buffer.count += other.count
    buffer
  }

  override def eval(buffer: Average): Any = {
    val n: Int = nExpression.eval().asInstanceOf[Int]
    ((buffer.sum*n)/(buffer.count))
  }

  override def withNewMutableAggBufferOffset(newOffset: Int): AvgTest =
    copy(mutableAggBufferOffset = newOffset)

  override def withNewInputAggBufferOffset(newOffset: Int): AvgTest =
    copy(inputAggBufferOffset = newOffset)

  override def children: Seq[Expression] = Seq(child, nExpression)

  override def nullable: Boolean = true

  // The result type is the same as the input type.
  override def dataType: DataType = child.dataType

  override def prettyName: String = "avg_test"

  override def serialize(obj: Average): Array[Byte] = {
    val stream: ByteArrayOutputStream = new ByteArrayOutputStream()
    val oos = new ObjectOutputStream(stream)
    oos.writeObject(obj)
    oos.close()
    stream.toByteArray
  }

  override def deserialize(bytes: Array[Byte]): Average = {
    val ois = new ObjectInputStream(new ByteArrayInputStream(bytes))
    val value = ois.readObject
    ois.close()
    value.asInstanceOf[Average]
  }
}
```
* Driver code :
``` Scala 
object TestAgg {
  object BegRegister extends NativeFunctionRegistration {
    val expressions: Map[String, (ExpressionInfo, FunctionBuilder)] = Map(
      expression[AvgTest]("multiply_average")
    )
  }
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setMaster("local[*]").setAppName("FirstDemo")
    val sc = new SparkContext(conf)
    val spark = SparkSession.builder().appName("Demo").config(conf).getOrCreate()


    BegRegister.registerFunctions(spark)
      val df = spark.read.json("src/test/resources/employees.json")
      df.createOrReplaceTempView("employees")
      df.show()
      /*
      +-------+------+
      |   name|salary|
      +-------+------+
      |Michael|  3000|
      |   Andy|  4500|
      | Justin|  3500|
      |  Berta|  4000|
      +-------+------+
       */
      val result = spark.sql("SELECT multiply_average(salary) as average_salary FROM employees")
      result.show()
    /*
      +--------------+
      |average_salary|
      +--------------+
      |          3750|
      +--------------+
     */
      val result1 = spark.sql("SELECT multiply_average(salary, 2) as average_salary FROM employees")
      result1.show()
      /*
      +--------------+
      |average_salary|
      +--------------+
      |          7500|
      +--------------+
     */
      val result2 = spark.sql("SELECT multiply_average(salary, 3) as average_salary FROM employees")
      result2.show()
      /*
      +--------------+
      |average_salary|
      +--------------+
      |         11250|
      +--------------+
      */
  }
}
```
* Here, `nExpression` represents our `n` argument. Other lines are self-explanatory. 
