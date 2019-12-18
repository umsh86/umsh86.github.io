---
layout: post
title: Kotlin in Action - 2장 코틀린 기초
categories: kotlin
---

kotlin in action - 2장. 코틀린 기초를 다시 보며 정리하는 내용입니다.

## 1. 함수

### 블록({ })이 본문인 함수

* 본문이 중괄호 ({ })로 둘러싸인 함수를 말함
* 함수 선언은 fun 키워드를 사용
* a:Int -> 파라미터 이름 뒤에 파라미터 타입을 선언
* Int : Return Type
* 세미콜론 ( ; )은 사용하지 않아도 됨
* 아래와 같이 블록( { } )이 본문인 함수는 return 을 명시해줘야함

```kotlin
fun max(a:Int, b:Int): Int {
	return if(a>b)  a else b
}
```



### 식이 본문인 함수

* 등호와 식으로 이뤄진 함수를 말함

```kotlin
fun max(a:Int, b:Int): Int = if(a>b) a else b
```

* 컴파일러가 타입을 분석해 타입을 정해주는 **타입추론**이 가능하기 때문에 **식이 본문인 함수에서는** Return Type은 생략 가능하다. 

  ```kotlin
  fun max(a:Int, b:Int) = if(a>b) a else b
  ```



## 2. 변수

### 코틀린에서는 타입 지정을 생략하는 경우가 흔하다(타입추론 덕분)

```kotlin
val answer = 42      // 타입 지정 생략. 단, 초기화 식이 있어야지만 생략이 가능하다
val answer: Int = 42 // 타입 지정
```



### 변경 가능한 변수와 변경 불가능한 변수

* val(value) : 변경 불가능(immutable). 자바의 final 에 해당됨

  * 블록을 실행할 때 정확히 한 번만 초기화되면 된다

    ```kotlin
    val message: String
    if (messageSend()) {
      message = "success"
    } else {
      message = "Fail"
    }
    ```

  * val 참조는 불변이지만, 참조가 가리키는 객체 내부의 값은 변경될 수 있다

    ```kotlin
    val languages = arrayListOf("Java") // 불변
    languages.add("Kotlin") // 객체 내부의 값은 변경 가능
    ```

* var(variable) : 변경 가능(mutable). 자바의 일반 변수

* 기본적으로 모든 변수를 val 를 사용해서 불변 변수로 선언하고, 필요한 부분만 var로 변경하자



### 변수사용(문자열 템플릿)

* Hello, World! 출력하기

  ```kotlin
  fun main(args: Array<String>) {
  	val name = "World"  
  	println("Hello, $name!") // $name : name의 내용이 출력됨
  }
  ```

* 문자열 이스케이프

  ```kotlin
  println("\$x") // $x 가 출력됨
  ```

* 복잡한 식은 중괄호( { } )로 둘러싸서 사용할 수 있다

  ```kotlin
  println("Hello, ${ if (args.size > 0) args[0] else "someone" }!") // 큰 따옴표도 사용가능, 배열의 값도 사용가능, if문 사용가능
  ```



## 3. 클래스와 프로퍼티

* Person class의 Java와 Kotlin 비교

  * Java

    ```java
    public class Person {
      
      private final String name;
      
      pubilc Person(String name) {
        this.name = name;
      }
      
      public String getName() {
        return name;
      }
      
    }
    ```

  * Kotlin

    ```kotlin
    class Person(val name: String)
    ```

    * 위와 같이 코드가 없이 데이터만 저장하는 클래스를 Value Object(값 객체)라고 함
    * kotlin의 기본 가시성은 public 이므로 생략

    

### 프로퍼티

* val 로 선언한 프로퍼티 : 읽기 전용(getter)

* var 로 선언한 프로퍼티 : 읽기/쓰기 가능

  ```kotlin
  class Person (
  	val name: String,  // private field, 읽기 전용이므로 kotlin은 public getter를 생성해준다
    var isMarried: Boolean // private field, 읽기/쓰기 가능하므로 kotlin은 public setter, public getter를 생성해준다
  )
  ```

* 사용하는 방법

  ```kotlin
  val person = Person("Bob", true) // new 사용하지 않고, 생성자를 호출함
  println(person.name) // Bob 출력. 프로퍼티 이름을 직접 사용하면 자동으로 getter를 호출해준다
  println(person.isMarried) // true 출력
  person.isMarrie = false // setter를 호출해서 값을 변경할 수 있음
  ```

  

### 커스텀 접근자(Getter)

* 자신이 정사각형인지 아닌지를 알려주는 class

  ```kotlin
  class Rectangle(val height:Int, val width: Int) {
    val isSquare: Boolean
    	get() {  // 커스텀 접근자 
        return height == width
      }
    	// 블록이 아닌 식으로도 구현 가능
    	// get() = height == width
  }
  ```

  

### 코틀린의 디렉터리와 패키지

* kotlin에서는 함수도 클래스처럼 임포트해서 사용할 수 있다

  ```kotlin
  package com.eomdev.kotlin.basic // package 선언
  
  class Rentangle(val height: Int, val width: Int) {
    val isSquare: Boolean
    	get() = height == width
  }
  
  fun createRandomRectangle(): Rentangle {
    val random =  Random()
    return Rentangle(random.nextInt(), random.nextInt())
  }
  ```

  ```kotlin
  import com.eomdev.kotlin.basic.createRandomRectangle // createRandomRectangle 함수 임포트
  
  fun main(args: Array<String>) {
    println(createRandomRectangle().isSquare) // import한 함수 사용
  }
  ```

* kotlin에서는 한 파일에 여러 클래스를 넣을 수 있다. 파일 이름도 마음대로 정의 가능.



## 4. enum과 when

### enum class

* 간단한 enum class

  ```kotlin
  enum class Color { // enum class 키워드를 사용한다
    BLACK, WHITE
  }
  ```

> class 는 키워드이므로, 클래스를 표현하는 변수 등을 정의할 때는 clazz나 aClass 와 같은 이름을 사용한다

* 프로퍼티와 메소드가 있는 enum 클래스

  ```kotlin
  enum class Color(
    // 상수 프로퍼티 정의
    val r: Int,
    val g: Int,
    val b: Int
  ) {
  
    RED(255, 0,0), // 각 상수를 생성할 때 그에 대한 프로퍼티 값
    YELLOW(255, 255, 0),
    GREEN(0, 255, 0);  // 주의!! enum 상수 목록과 메소드 정의 사이에 반드시 세미콜론을 사용해야함!!
    
    fun rgb() = (r * 256 + g) * 256 + b // 메소드 정의
  }
  ```

  

### when

* java의 switch를 대신함
* if와 마찬가지로 값을 만들어 내는 **식** 임

```kotlin
fun findName(color: Color) = // 식이므로 = 연산자로 사용 가능하다
	when (color) {
    Color.RED -> "Richard"
    Color.YELLOW -> "York"
    Color.GREEN -> "Gave"
  }


println(findName(Color.YELLOW))
```

* 한 when 분기 안에서 여러 값을 사용하는 방법

  ```kotlin
  fun getWarmth(color: Color) = 
  	when (color) {
      Color.RED, Color.YELLOW -> "warm" // , 로 구분해서 여러 값을 사용할 수 있다
      Color.GREEN -> "natural"
    }
  ```

* 임의의 객체와 함께 사용하기

  * 예) 두 색을 혼합했을 때, 정해진 색을 알려주는 함수

    ```kotlin
    fun mix(c1: Color, c2: Color) =
    	when(setOf(c1, c2)) {						// when 식의 인자로 아무 객체나 사용 가능. 여기서는 Set객체로 만드는 setOf 함수를 사용함
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        
        else -> throw Exception("Dirty Color") // 매치되는 조건이 없는 경우
        
      }
    ```

    

* when에 인자를 사용하지 않는 방법

  * 코드는 읽기 어려워지지만, 성능을 향상 시키기 위한 방법

  ```kotlin
  fun mix(c1: Color, c2: Color) =
  	when {					// 인자가 없음
      (c1 == RED && c2 == YELLOW) ||
      (c1 == YELLOW && c2 == RED) ->
      	ORANGE
  
      (c1 == YELLOW && c2 == BLUE) ||
      (c1 == BLUE && c2 == YELLOW) ->
      	GREEN
      
      else -> throw Exception("Dirty Color") // 매치되는 조건이 없는 경우
      
    }
  ```

* when 에 블록문을 사용할 수 도 있다

  ```kotlin
  fun mix(c1: Color, c2: Color) =
  	when(setOf(c1, c2)) {		
      setOf(RED, YELLOW) -> {
        ...
        ORANGE // 블록의 마지막 식이 반환된다
      } 
      setOf(YELLOW, BLUE) -> {
       ....
        GREEN
      }
      
      else -> throw Exception("Dirty Color")
      
    }
  ```

  

### 스마트 캐스트(타입검사 + 타입캐스트)

* 자바의 경우

  ```java
  if (c instanceof MessageDummyService) {
    MessageDummyService messageService = (MessageDummyService)c;
    messageService.send();
  }
  ```

* kotlin의 경우

  ```kotlin
  if (c is MessageDummyService) { // kotlin은 is가 자바의 instanceof와 비슷하다
      messageService.send(); // java에서는 타입변환을 해줘야하지만, kotlin에서는 컴파일러가 자동으로 캐스팅을 해준다. 이것을 스마트 캐스트라고 한다.
  }
  ```

* 스마트 캐스트 조건

  * is 로 타입을 검사한 다음에 값이 바뀔 수 없는 val 이어야함
  * 커스텀 접근자를 사용하지 않아야 함

* 명시적으로 타입 캐스팅을 위해서는 **as** 키워드를 사용해야한다

  ```kotlin
  val n = e as Num
  ```

  

### 수에 대한 이터레이션

* range

  ```kotlin
  val oneToTen = 1..10 // .. 연산자를 사용. 1 ~ 10 까지
  ```

  * 피즈버즈 게임(3,5로 나누어 떨어지면 피즈버즈, 3으로 나누어떨어지면 피즈, 5로 나누어 떨어지면 버즈)

    ```kotlin
    fun fizzBuzz(i: Int) =  when {
      i % 15 == 0 -> "FizzBuzz"
      i % 3 == 0 -> "Fizz"
      i % 5 == 0 -> "Buzz"
      else - "$i "
    }
    
    // for..in 루프문 사용
    for( i in 1..100) {
      println(fizzBuzz(i))
    }
    
    ```

* 증가 값 있는 for

  * downTo : 감소

  ```kotlin
  for ( i in 100 downTo 1 step 2) { // 100부터 1까지 감소하는데, 2씩 감소시킨다
    println(fizzBuzz(i))
  }
  ```

* map 이터레이션

  ```kotlin
  val list = arrayListOf("10", "11", "1001")
  for ((index, element) in list.withIndex()) {
    println("$index: $element")
  }
  ```

* in을 사용해서 컬렉션이나 범위의 원소 검사하기

  ```kotlin
  fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z' // in
  fun isNotDigit(c: Char) = c !in '0'..'9' // !in 사용해서 범위에 속하지 않은지 검사
  "Kotlin" in setOf("Java", "Scala") // Set에 Kotlin은 없다. false
  ```


> 출처 : Kotlin in Action http://acornpub.co.kr/book/kotlin-in-action