---
layout: post
title: Kotlin 기본 타입
categories: kotlin
---

## 1. Numbers
* 자바와 같이 long은 suffix로 L 또는 l을 붙인다
  ```kotlin
    val one = 1 // Int
    val threeBillion = 3000000000 // Long
    val oneLong = 1L // Long
    val oneByte: Byte = 1
  ```


* Float와 Double
  ```kotlin
    val pi = 3.14 // 아무것도 사용하지 않으면 Double
    val e = 2.7182818284 // Double
    val eFloat = 2.7182818284f // f를 붙여서 사용하면 Float, actual value is 2.7182817
  ```


* 자동 타입 변환에 대해서 
  ```kotlin
    val x:Byte = 1  // 더 작은 타입인 Byte에 저장되어 있는 값을
    val y:Int = x   // 더 큰 타입인 Int에 저장하려고 하면 오류가 난다
  ```


## 2. Literal constants
* Decimal : 123
* Long : 123L
* hex(16진수) : 0x0F
* `_` 언더스코어 지원 : 1_000_000_000
* binary : 0b101010
* 45919.toString(2) : 10진수 45919를 2진수로 변환


## 3. Explicit conversions
* 서로 타입이 다른 Int와 Long의 값을 비교할 수 있는가?
  * 없다. 사이즈가 작은 타입을 큰 타입으로 변경해주어야 한다
  ```kotlin
    val x:Int = 10
    val y:Long = 100000000
    x == y // 비교할 수 없다

    x.toLong() == y // 이런식으로 직접 타입을 변경해서 사용해야 한다
  ```

* 타입변환시 사용 가능한 메소드
  * toByte(): Byte
  * toShort(): Short
  * toInt(): Int
  * toLong(): Long
  * toFloat(): Float
  * toDouble(): Double
  * toChar(): Char

* 하지만, 산술연산에서는 자동으로 타입 변환이 이루어진다
  ```kotlin
    val l = 1L + 3 // Long + Int => Long 정상으로 계산이 된다
  ```


* bit 연산
  * shl(bits) – signed shift left
  * shr(bits) – signed shift right
  * ushr(bits) – unsigned shift right
  * and(bits) – bitwise and
  * or(bits) – bitwise or
  * xor(bits) – bitwise xor
  * inv() – bitwise inversion

  ```kotlin
    0b10000000.shl(1)
    // 결과 : 256
  ```

## 4. Characters
* Characters문자는 바로 Number로 변경할 수 없다

## 5. Booleans
true, false, ||, &&, ! 

## 6. Arrays
  ```kotlin
    arrayOf(1,2,3,4,5) // 배열 생성하는 방법 1.
    val x = Array<Int>(10) {println(it); it*it} // 배열 생성하는 방법 2
    x[9] // 출력은 81

    arraysOfNulls<Int>(10) // 요소를 null으로 채워서 생성하는 방법
  ```


* 다중 배열
  ```kotlin
    fun main() {
      val x = arrayOf(
            arrayOf(0,1,2),
            arrayOf(3,4,5),
            arrayOf(6,7,8)
        )

        println(x[0][0])    // 0
        println(x[2][1])    // 7
    }
  ```

## 7. Primitive type arrays
  ```kotlin
    val x: IntArray = intArrayOf(1, 2, 3)
    x[0] = x[1] + x[2]
  ```


## 8. Strings

```kotlin
  fun main() {
    val s = "abc" + 1
    println(s + "def")  // 결과 : abc1def. 1을 toString()해서 연산한 결과로 보여준다
  }
```

* data 라는 형식의 class를 정의해서 테스트 해 본 경우
  ```kotlin
    data class Data(val x:Int)

    fun main() {
        val str = "abc" + Data(100)
      println(str) // 결과 : abcData(x=100)
    }
  ```


* triple quote(""")

    ```kotlin
        fun main() {
          val str = """
          줄맞춤
          하는 역할을 하는 듯합니다.
          하하하
          """
          println(str)
    
          val text = """
          |Tell me and I forget.
          |Teach me and I remember.
          |Involve me and I learn.
          |(Benjamin Franklin)
          """
    
          println(text)
          println(text.trimMargin())
        }
    
    // 실행 결과
        줄맞춤
        하는 역할을 하는 듯합니다.
        하하하
        
        |Tell me and I forget.
        |Teach me and I remember.
        |Involve me and I learn.
        |(Benjamin Franklin)
        
    Tell me and I forget.
    Teach me and I remember.
    Involve me and I learn.
    (Benjamin Franklin)
    ```


* $ 를 사용하는 방법
  ```kotlin
    fun main() {
      println("\$abc")         // $abc
      println("${'$'}abc")     // $abc
      println("${"$"}abc")     // $abc
    }
  ```





> 출처 
> * Hyunsok Oh님의 [코틀린 기초] 2. 기본 타입 - 수타입, 불리언, 문자, 문자열, 배열 타입 : https://www.youtube.com/watch?v=aO-reo747Bc&t=1s
> * kotlin 레퍼런스 문서 : https://kotlinlang.org/docs/reference/basic-types.html
