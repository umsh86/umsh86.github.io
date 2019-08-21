---
layout: post
title: Kotlin 기본 문법
categories: kotlin
---

jetbrains이 만들었다는 부분부터 매력으로 느끼고 있었는데(사랑합니다 jetbrains), 어느 순간부터 Spring 레퍼런스 문서에 Java 코드를 코틀린으로도 작성할 수 있다고 보여주기 시작하더니, Gradle 5.0에서도 Gradle Kotlin DSL을 지원하기 시작했다.

코틀린에 대해서 지속적인 관심은 가지고 있었으나, 어디서부터 어떻게 시작해야할지 몰라서, 다른것 하기도 바빠서라는 등등의 핑계로 시작은 못하고 "아 하긴 해야되는데.."하는 생각만 마음 한 구석에 불편하게 자리 잡고 있었는데...

오늘 6시간 전에 토비님이 페이스북에 [게시하신 글](https://www.facebook.com/tobyilee/posts/10216483380811617)이 있었으니..

![image01](https://user-images.githubusercontent.com/1261904/63427755-d7e44580-c450-11e9-9ce7-86920150d531.png)
감사합니다. 정말 감사합니다. 구독, 좋아요, 알림설정 모두 진행했어요 엉엉 ㅠㅠ

영상이 쌓이면 나중에 볼 엄두가 안날 것 같아서 가능하면 밀리지 않도록 바로 보고, 학습하고, 정리해놓으려고 한다.

아래는 첫 번째 영상을 보고 정리한 내용이다.
# Basic Syntax

## 1. import
* Java와 같다. 다른점은 static import는 따로 존재하지 않고, 똑같이 import로 사용하면 된다
  ```kotlin
    package my.demo

    import kotlin.text.*
  ```


## 2. Program entry point
* main 함수에 대한 내용. 아래와 같이 선언해서 사용하면 된다.
  ```kotlin
    fun main() {
      println("Hello world!")
    }
  ```


## 3. Functions
* functions 선언 및 사용

  ```kotlin
    fun sum(a: Int, b: Int): Int { // java와 다르게 fun sum(파라미터이름: 타입) 리턴타입 형태이다
        return a + b
    }

    fun main() {
        print("sum of 3 and 5 is ")
        println(sum(3, 5))
    }
  ```


* 아래와 같이 줄여서 반환해야하는 값의 식을 바로 사용할 수도 있다.
  ```kotlin
    fun sum(a: Int, b: Int) = a + b
  ```

* 자바에서의 void에 해당되는 키워드는 Unit 이다
  ```kotlin
    fun printSum(a: Int, b: Int): Unit {
      println("sum of $a and $b is ${a + b}")
    }
  ```

* 위의 Unit은 생략 가능하다
  ```kotlin
    fun printSum(a: Int, b: Int) {
      println("sum of $a and $b is ${a + b}")
    }

  ```


## 4. Variables
* 변수 정의에 대한 내용

* val : 재정의가 불가능한 상수(constant)에 해당. 자바의 final. immua
  ```kotlin
    fun main() {
      val a: Int = 1  // 즉시 할당해서 사용하는 경우
      val b = 2   // `Int` type 유추
      val c: Int  // Local 변수로 사용할 때는 초기화는 바로 하지 않아도 된다
      c = 3       // 하지만, 사용하기 전에는 반드시 값을 설정해줘야 한다
      println("a = $a, b = $b, c = $c")
  }
  ```

* var : 값을 재할당 가능한 변수
  ```kotlin
    fun main() {
      var x = 5 // `Int` type 유추
      x += 1
      println("x = $x")
  }
  ```


* 함수(fun)안에 함수를 정의 가능하다
  ```kotlin
    class A {

    }

    val PI = 3.14
    var x = 0

    fun incrementX() { 
        fun bar() { println("함수안에 함수 정의하는 것이 가능함") } // 가능하다
        bar();
        x += 1 
    }

    fun main() {
        println("x = $x; PI = $PI")
        incrementX()
        println("incrementX()")
        println("x = $x; PI = $PI")
    }
    
    // 출력 결과
    x = 0; PI = 3.14
    함수안에 함수 정의하는 것이 가능함
    incrementX()
  ```
  

## 5. Comments
* 주석은 자바와 같다
  ```kotlin
    // This is an end-of-line comment

    /* This is a block comment
       on multiple lines. */
  ```


## 6. String templates

  ```kotlin
    fun main() {
        var a = 1
        // simple name in template:
        val s1 = "a is $a"  // 변수에 저장된 값은 $a로 사용할 수도 있고,
        println(s1)

        val s3 = "a is ${a}" // ${a}와 같이 사용할 수도 있다.
        println(s3)

        a = 2
        // 메소드는 ${} 을 사용해야 한다
        val s2 = "${s1.replace("is", "was")}, but now is $a"
        println(s2)
    }
  ```


## 7. Conditional expressions
* if문은 아래와 같다
  ```kotlin
    fun maxOf(a: Int, b: Int): Int {
        if (a > b) {
            return a
        } else {
            return b
        }
    }

    fun main() {
        println("max of 0 and 42 is ${maxOf(0, 42)}")
    }
  ```

* 또는 아래와 같이 줄여서 사용할 수도 있다
  ```kotlin
    fun maxOf(a: Int, b: Int) = if (a > b) a else b

    fun main() {
        println("max of 0 and 42 is ${maxOf(0, 42)}")
    }
  ```

## 8. Nullable values and null checks
* return type 에 ? 가 붙으면, null이 return 될 수 있다는 의미이므로 null check를 해줘야한다.
  ```kotlin
    fun parseInt(str: String): Int? {
        return str.toIntOrNull()
    }

    fun printProduct(arg1: String, arg2: String) {
        val x = parseInt(arg1)
        val y = parseInt(arg2)

        // Using `x * y` yields error because they may hold nulls.
        if (x != null && y != null) {
            // x and y are automatically cast to non-nullable after null check
            println(x * y)
        }
        else {
            println("'$arg1' or '$arg2' is not a number")
        }    
    }


    fun main() {
        printProduct("6", "7")
        printProduct("a", "7")
        printProduct("a", "b")
    }
  ```


* 또는 아래와 같이 null일 경우 `?:`를 사용해서 대체할 수 있는 값을 정의해줄 수도 있다
  ```kotlin
    fun parseInt(str: String): Int? {
        return str.toIntOrNull()
    }

    fun printProduct(arg1: String, arg2: String) {
        val x = parseInt(arg1)
        val y = parseInt(arg2)

        // x and y are automatically cast to non-nullable after null check
        println((x?:0) * (y?:0)) // x가 null이면 0으로, y가 null이면 0으로 대체해준다.
    }

    fun main() {
        printProduct("6", "7")
        printProduct("a", "7")
        printProduct("99", "b")
    }
    
    // 결과
    42
    0
    0
  ```
  
  
## 9. Type checks and automatic casts

  ```kotlin
    fun getStringLength(obj: Any): Int? {
        if (obj is String) { // obj가 String Type인지 검사를 하고 나면,
            // `obj`는 자동으로 String으로 변환된다. java에서는 (String)obj.length 형식으로 사용했었음
            return obj.length
        }

        // `obj` is still of type `Any` outside of the type-checked branch
        return null
    }
  ```
  

  ```kotlin
    fun getStringLength(obj: Any): Int? {
        if (obj !is String) return null // obj가 String이 아닌것을 검사하고 나면

        // `obj`는 String 이므로 자동으로 cast 된다 
        return obj.length
    }
  ```


## 10. for loop

* 자바에 존재하던 전통적인 for loop는 존재하지 않는다
  ```java
    for(int i=0; i<3; i++) // 코틀린에는 존재하지 않음
  ```

* 대신 아래와 같이 사용함
  ```java
    val items = listOf("apple", "banana", "kiwifruit")
    for (item in items) {
        println(item)
    }


// items.forEach({item -> println(item)}) 도 사용 가능
  ```
  
  
* 컬렉션의 경우 `indices`를 사용해서 index 정보를 알 수 있다
  
  ```kotlin
    val items = listOf("apple", "banana", "kiwifruit")
    for (index in items.indices) {
        println("item at $index is ${items[index]}")
    }
    
    // 출력
    item at 0 is apple
    item at 1 is banana
    item at 2 is kiwifruit
  ```
  
  
## 11. while loop

* 컬렉션에 size 를 사용할 수 있고, while loop는 자바와 똑같이 사용할 수 있다.
  ```kotlin
    val items = listOf("apple", "banana", "kiwifruit")
    var index = 0
    while (index < items.size) {
        println("item at $index is ${items[index]}")
        index++
    }
  ```


## 12. when expression

* 자바의 switch 문에 해당되는 내용이다. `조건 -> 결과` 형태로 사용할 수 있으며 `결과`에 해당되는 타입은 모두 똑같아야 한다.
  ```kotlin
    fun describe(obj: Any): String =
        when (obj) {
            1          -> "One"             // 상수 비교
            "Hello"    -> "Greeting"        // 문자열 비교
            is Long    -> "Long"            // 타입 비교
            !is String -> "Not a string"    // 타입 비교
            else       -> "Unknown"         // java switch문의 default에 해당
        }

  ```


## 13. Ranges

* range 안에 포함인지 확인하는 경우
  ```kotlin
    val x = 10
    val y = 9
    if (x in 1..y+1) {  // .. 이 의미하는 것은 양쪽 다 포함한다는 뜻. 즉, x는 1<= x <=10 이라는 의미이다.
        println("fits in range")
    }
  ```

* out of range, range안에 포함되지 않는지 확인하는 경우
    ```kotlin
      val list = listOf("a", "b", "c")

      if (-1 !in 0..list.lastIndex) {    // !in 을 사용한다
          println("-1 is out of range")
      }
      if (list.size !in list.indices) {
          println("list size is out of valid list indices range, too")
      }
    ```


* step과 downTo
  ```kotlin
    for (x in 1..10 step 2) { // x는 1부터 2씩 증가하여 10까지 
        print(x) // 출력 : 13579
    }
    println()
    for (x in 9 downTo 0 step 3) { // x는 9부터 3씩 감소하여 0까지
        print(x) // 출력 : 9630
    }
  ```



## 13. Collections
* 컬렉션에는 list와 map이 존재한다
* list에는 immutable list와 mutable list가 존재한다
  ```kotlin
    fun main() {
        // immutable list는 그냥 listOf 를 사용하면 된다
        val items = listOf("apple", "banana", "kiwifruit")
        for (item in items) {
            println(item)
        }
        
        val immutableItems = listOf(1, 2, 3)
        println(immutableItems)  // 결과 : [1, 2, 3]

        // mutable list는 mutableListOf를 사용해야 한다     
        val mutableItems = mutableListOf(1, 2)
        println(mutableItems)     // 결과 : [1, 2]
        mutableItems += 3         // mutable 이므로 변경 가능
        println(mutableItems)     // 결과 : [1, 2, 3]
        
        // var를 사용해서 immutable List를 생성하면 변경할 수 있다
        var items2 = listOf("apple", "banana", "kiwifruit")
        items2 += "strawberry"
        println(items2)           // 결과 : [apple, banana, kiwifruit, strawberry]
        
    }
  ```


* 자바의 스트림처럼 사용 가능
  ```kotlin
    val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
    fruits
      .filter { it.startsWith("a") }
      .sortedBy { it }
      .map { it.toUpperCase() }
      .forEach { println(it) }
  ```


## 14. Creating basic classes and their instances

  ```kotlin
    fun main() {
        val rectangle = Rectangle(5.0, 2.0) //no 'new' keyword required
        val triangle = Triangle(3.0, 4.0, 5.0)
        println("Area of rectangle is ${rectangle.calculateArea()}, its perimeter is ${rectangle.perimeter}")
        println("Area of triangle is ${triangle.calculateArea()}, its perimeter is ${triangle.perimeter}")
    }

    abstract class Shape(val sides: List<Double>) { // class를 정의하면서 List<Doulbe> sides 파라ㅣ터를 갖는 생성자도 동시에 정의함.
        val perimeter: Double get() = sides.sum()
        abstract fun calculateArea(): Double
    }

    // interface
    interface RectangleProperties {
        val isSquare: Boolean
    }

    
    // class
    class Rectangle(
        var height: Double,
        var length: Double
    ) : Shape(listOf(height, length, height, length)), RectangleProperties {
        override val isSquare: Boolean get() = length == height
        override fun calculateArea(): Double = height * length
    }

    class Triangle(
        var sideA: Double,
        var sideB: Double,
        var sideC: Double
    ) : Shape(listOf(sideA, sideB, sideC)) { // : Shape의 의미는 Shape를 상속받는다는 의미.
        override fun calculateArea(): Double {
            val s = perimeter / 2
            return Math.sqrt(s * (s - sideA) * (s - sideB) * (s - sideC))
        }
    }
  ```


> 출처
> * Hyunsok Oh님의 [코틀린 기초] 1. 시작하며 - 프로그래밍에 필요한 기본 문법 정리 : https://www.youtube.com/watch?v=oVk2y26JCyk&t=6s
> * 토비(이일민)님 페이스북 글 : https://www.facebook.com/tobyilee/posts/10216483380811617
> * kotlin 레퍼런스 문서 : https://kotlinlang.org/docs/reference/basic-syntax.html
