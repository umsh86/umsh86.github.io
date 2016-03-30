---
layout: post
title: 자바8에 추가된 interface 기능에 대해서(default method와 익명 구현 객체)
categories: java
---

3월에 회사를 그만두고, 한 달은 푹 쉬자는 마음으로 해외여행을 계획하고 프랑스, 스위스를 다녀왔다.
여행을 다녀와서 어제까지는 밀린 잡다한 일들을 처리했고, 회사를 그만두면서 계획했던 자바8(계획한 건 이것 말고도 많지만..)에 대해 오늘 오전부터 공부하다가 괜찮은 내용이 있어서 정리 좀 해놓으려고 한다(이왕 하는김에 인터페이스에 대해서 전반적으로..).

개발을 진행하면서 interface는 여러 장점들 때문에 많이 사용해왔다. 객체의 내부 구조를 알 필요 없이 인터페이스의 메소드만 알고 있으면 된다는 점, 다형성을 이용한 interface의 구현 객체를 손쉽게 교체할 수 있다는 점 등..

이 유용한 자바 인터페이스가 가질수 있는 메소드는 자바7 이전까지는 실행 블록이 없는 추상메소드로만 선언이 가능했지만, 자바8 부터는 실행 블록이 존재하는 `디폴트 메소드`와 `정적 메소드`도 선언이 가능하게 변경되었다.

또한 인터페이스의 구현체가 존재하지 않더라도, `익명 구현 객체`를 만들어서 사용할 수 있는데, 인터페이스에 대해서 정리하면서 관련 내용도 같이 정리하려고 한다.

# 1. 자바8에서 인터페이스가 가질 수 있는 것들

## 1.1. 상수 필드(public static final)

```java
    public interface Parent{
        public static final MAX_VALUE = 100;
        public static final MIN_VALUE = 0;
    }
```

## 1.2. 추상 메소드(public abstract)

```java
    public interface Parent{
        // 상수 필드
        public static final MAX_VALUE = 100;
        public static final MIN_VALUE = 0;
        
        // 추상 메소드
        public abstract void run();
    }
```

## 1.3. 디폴트 메소드(public default)

```java
    public interface Parent{
        // 상수 필드
        public static final MAX_VALUE = 100;
        public static final MIN_VALUE = 0;
        
        // 추상 메소드
        public abstract void run();
        
        // 디폴트 메소드 : 실행 내용까지 작성이 가능하다.
        public default void setState(boolean state){
            
            if(state){
                System.out.println("현재 상태는 정상입니다");
            }else{
                System.out.println("현재 상태는 비정상입니다");
            }
            
        }
    }
```

해당 인터페이스 구현시, 디폴트 메소드를 재정의(오버라이딩)해서 사용할 수 있다.

이 디폴트 메소드는 모든 구현 객체에서 공유하는 기본 메소드 용도로 사용할 수도 있지만, 더 유용하게 활용할 수 있다. 바로 이전에 개발한 구현 클래스를 그대로 사용하고 변경하지 않으면서, 새롭게 개발하는 클래스는 디폴트 메소드를 활용해 새로운 기능을 만들수 있는 것이다.
아래와 같이 인터페이스와 그 구현체가 존재한다고 가정해보자.

```java
    public interface MyInterface{
        public abstract void method1();
    }
```

```java
    public class MyInterfaceImplA implements MyInterface{
        
        public void method1(){
        // 구현 내용 
        }
    
    }
```

그런데 만약에 나중에 MyInterface에 새로운 기능을 추가해야되는 경우가 생긴다면, 기존에 해당 인터페이스의 구현체였던 MyInterfaceImplA에도 해당 메소드의 내용을 구현해줘야하는 문제점이 발생한다.
만약 MyInterfaceA를 수정할 여건이 안된다면 MyInterface에 새로운 추상 메소드를 추가할 수 없게 된다. 이런 경우 MyInterface에 디폴트 메소드를 선언하고, 
필요에 따라서 새로 구현하는 구현체에서 디폴트 메소드를 재정의(오버라이딩)해서 구현해주면 된다.

```java
    public interface MyInterface{
        public abstract void method1();
        // default method 선언
        public deafult void method2(){
            // 구현 내용
        }
    }
```

```java
    // 기존에 있던 구현체들은 변경되는 내용 없음
    public class MyInterfaceImplA implements MyInterface{
        
        public void method1(){
        // 구현 내용 
        }
    
    }
```


```java
    // 새로 생성된 구현체
    public class MyInterfaceImplB implements MyInterface{
        
        public void method1(){
        // 구현 내용 
        }
        
        @Override
        public void method2(){
            // 디폴트 메소드 재정의
        }
    
    }
```



## 1.4. 정적 메소드(public static)

```java
    public interface Parent{
        // 상수 필드
        public static final MAX_VALUE = 100;
        public static final MIN_VALUE = 0;
        
        // 추상 메소드
        public abstract void run();
        
        // 디폴트 메소드 : 실행 내용까지 작성이 가능하다.
        public default void setState(boolean state){
            
            if(state){
                System.out.println("현재 상태는 정상입니다");
            }else{
                System.out.println("현재 상태는 비정상입니다");
            }
            
        }
        
        // 정적 메소드
        public static void change(){
            System.out.println("상태를 변경합니다.");
        }
    }
```

물론 사용할 때, `Parent.change();`와 같이 객체가 없어도 인터페이스만으로도 호출이 가능하다.

# 2. 인터페이스의 구현


## 2.1. 익명 구현 객체

해당 인터페이스의 구현체를 만들어서 사용하는 것이 일반적이고, 클래스를 재사용할 수 있기 때문에 편리하지만, 일회성의 구현 객체를 만들기 위해서는 비효율적이다. 자바8에서 지원하는 람다식은 인터페이스의 익명 구현 객체를 생성할 수 있도록 추가되었다.
UI프로그래밍에서 이벤트를 처리하거나, 임시 작업 스레드를 만들기 위해서 유용하게 사용될 수 있다.

아래와 같은 방법으로 사용한다.

```
    인터페이스 변수 = new 인터페이스(){
        // 인터페이스에 선언된 추상 메소드의 실제 메소드 선언        
    };
```

ex)

```java
    public class Example{
        public static void main(String[] args){
            
            Parent parent = new Parent(){
                public void run(){
                    // 실제 구현 내용
                }
            }
            
        }
    }
```

익명 구현 객체도 Example.java 컴파일시 Example$1.class와 같이 이름뒤에 $가 붙고 생성 번호가 붙는 class 파일이 생성된다.


## 2.2. 다중 인터페이스 구현 클래스

```java
    public class Child implements Parent1, Parent2 {
        // Parent1에 선언된 추상 메소드의 실제 메소드 선언 및 구현
        // Parent2에 선언된 추상 메소드의 실제 메소드 선언 및 구현
    }
```

# 3. 타입변환

# 3.1. 자동 타입 변환(Promotion)

`인터페이스 변수 = new 구현체();` 의 경우

ex) `Car car = new Bus();`

# 3.2. 강제 타입 변환(Casting)

구현 객체가 인터페이스 타입으로 자동 타입 변환 되는 경우, 인터페이스에 선언된 메소드만 사용이 가능하다. 만약에 구현 클래스에 선언된 필드와 메소드를 사용해야 하는 경우에는 강제 타입 변환을 이용하여 다시 구현 클래스 타입으로 변환해야한다.

`구현클래스 변수 = (구현클래스)인터페이스변수;`

ex) `Bus bus = (Bus)car;`

# 3.3. 객체 타입 확인(instanceof)

강제 타입 변환시 instanceof 연산자로 확인하고 안전하게 강제 타입 변환을 할 것.

```java
    if(car instanceof Bus){
        Bus bus = (Bus)car;
    }
```

# 4. 인터페이스 상속

클래스와 달리 다중 상속이 가능하다.

`public interface 하위인터페이스 extends 상위인터페이스1, 상위인터페이스2 { ... }`


 

