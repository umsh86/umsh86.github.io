---
layout: post
title: Exception 처리에 대해서(자바7에 추가된 try-catch-resources 포함)
categories: java
---

오늘 오전에는 예외 처리에 대해서 한 번 읽어보았다. 특별한 내용은 없었고, 예외 처리에 대해서 전반적인 정리와 자바7에서 추가된 `자동 리소스 닫기`라고 불리우는 `try-catch-resources`에 대해서 정리하려고 한다. 

자바7에서 추가된 내용만 확인하려면 `3.3. 멀티 catch(자바7에서 추가된 내용)` 과 `4. 자동 리소스 닫기(자바7에서 추가된 try-catch-resources)` 부분만 확인하면 된다.

# 1. Exception?

사용자의 잘못된 조작이나 개발자의 잘못된 코딩으로 인해 발생하는 프로그램 오류를 말하는 `예외(Exception)`는 발생시에 프로그램이 곧바로 종료된다는 점에서는 에러와 동일하다. 
하지만 예외는  예외 처리(Exception Handling)을 통해 프로그램을 종료하지 않고 정상 실행 상태가 유지되도록 할 수 있다.

자바가 제공하거나 개발자가 만드는 모든 예외들은 `java.lang.Exception` 클래스를 상속받는다.

예외는 크게 `Exception`을 상속받아 구현한 `일반 예외`와 `RuntimeException`을 상속받아 구현한 `실행 예외`로 구분한다.(RuntimeException도 Exception을 상속받지만 JVM은 RuntimeException을 상속했는지의 여부를 보고 실행예외를 판단한다)
`일반 예외`의 경우 자바 소스를 컴파일하는 과정에서 예외 처리 코드가 필요한지 검사하지만, `실행 예외`의 경우는 컴파일 과정에서 검사하지 않는다.

# 2. RuntimeException을 상속받는 실행 예외

프로그램을 개발하면서 빈번하게 일어나는 `NullPointerException`, `ArrayIndexOutOfBoundsException`, `NumberFormatException`, `ClassCastException` 등이 실행 예외에 속한다.

# 3. 예외 처리 코드(Exception Handling)

## 3.1. try-catch-finally


```java
    try{
        // 예외 발생 가능 코드
        Class clazz = Class.forName("java.lang.String2");
    }catch(ClassNotFoundException e){   // 예외클래스 
        // 예외 처리
    }finally{
        // 항상 실행되야하는 내용
    }
    
```

## 3.2. 다중 catch

catch 블록이 여러 개라 할지라도 단 하나의 catch 블록만 실행되므로 하위 예외 클래스를 위쪽에 위치하도록, 상위 예외 클래스를 아래쪽에 위치하도록 해야 한다.

```java
    try{
        // 1. ArrayIndexOUtOfBoundsException 발생
        // 2. NumberFormatException 발생
        // 3. 다른 Exception 발생
    }catch(ArrayIndexOutOfBoundsException e){ 
        // 예외 처리1
    }catch(NumberFormatException e){   
        // 예외 처리2
    }catch(Exception e){    
        // 예외 처리3
    }finally{
        // 항상 실행되야하는 내용
    }
    
```

## 3.3. 멀티 catch(자바7에서 추가된 내용)

자바7부터 하나의 catch 블록에서 여러 개의 예외를 처리할 수 있도록 멀티 catch 기능이 추가 되었다.
catch의 ( ) 안에 동일하게 처리 하고 싶은 예외를 | 로 연결하면 된다.

```java
    try{
        // 1. ArrayIndexOUtOfBoundsException 발생
        // 2. NumberFormatException 발생
        // 3. 다른 Exception 발생
    }catch(ArrayIndexOutOfBoundsException | NumberFormatException e){ 
        // 예외 처리1, 예외 처리2
    }catch(Exception e){    
        // 예외 처리3
    }finally{
        // 항상 실행되야하는 내용
    }
    
```

# 4. 자동 리소스 닫기(자바7에서 추가된 try-catch-resources)

자바7에서 추가된 내용인데, 예외 발생 여부와 상관없이 finally에서 항상 처리해줘야했던 리소스 객체(입출력 스트림, 소켓 등)의 close() 메소드를 호출해서 안전하게 리소스를 닫아준다.

자바6 이전까지 사용했던 코드

```java
    FileInputStream fis = null;
    try{
        fis = new FileInputStream("file.txt");
        // ... 
        
    }catch(IOException e){
        // ...
    }finally{
        if(fis != null){
            try{
                fis.close();
            }catch(IOException e) {
            
            }
        }
    }
    
```

자바7 부터 사용가능한 코드. 명시적으로 close()를 호출하지 않아도 자동으로 호출해준다.

```java
    try(
        FileInputStream fis = new FileInputStream("file.txt")
        FileOutputStream fos = new FileOutputStream("file2.txt")
    ){
        // ...
    }catch(IOException e){
        // ...
    }
```

try-catch-resources를 사용하기 위해서는 해당 리소스 객체가 java.lang.AutoCloseable 인터페이스를 구현하고 있어야 한다. 아래와 같이 API 도큐먼트에서 AutoCloseable 인터페이스를 찾아 보면 "All Known Implementing Classes:"에서
찾아볼 수 있다.

![image1](https://cloud.githubusercontent.com/assets/1261904/14164645/2a9ac442-f73c-11e5-8742-2708c1407d3d.png)

또는 아래와 같이 직접 AutoCloseable을 구현해서 사용할 수도 있다.

```java
    public class FileInputStream implements AutoCloseable {
    
        private String file;
    
        public FileInputStream(String file){
            this.file = file;
        }
    
        public void read(){
            System.out.println(file+"을 읽습니다.");
        }
    
        @Override
        public void close() throws Exception {
            System.out.println(file +"을 닫습니다.");
        }
    }
```

# 5. 예외 넘기기

try-catch 블록으로 예외 처리를 하는 방법도 있지만 메소드를 호출한 곳으로 throws를 사용해서 넘길 수도 있다.

```java
    
    public void method1(){
        try{
            method2();
        }catch(ClassNotFoundException e){
            // 예외 처리 코드
        }
        
    }

    public void method2() throws ClassNotFoundException {
        Class clazz = Class.forName("java.lang.String2");
    }
```

# 6. 사용자 정의 예외

## 6.1. 사용자 정의 예외 클래스 선언

일반 예외의 경우(컴파일러가 체크함) Exception을 상속받아 구현하면 되고, 실행 예외의 경우(컴파일러가 체크 안함) RuntimeException을 상속받아 구현 하면 된다.
보통 매개 변수가 없는 생성자와 예외 발생 원인(메시지)를 전달하기 위해 String 파라미터를 갖는 생성자 2개를 선언한다.

```java
    public class AccountNotFoundException extends RuntimeException{
    
        public AccountNotFoundException() {
        
        }
    
        public AccountNotFoundException(String message) {
            super(message);
        }
    
    }
```

## 6.2. 예외 발생 시키기

```java
    throw new AccountNotFoundException();
    throw new AccountNotFoundException("Unknown username/password");
```

## 6.3. 예외 정보 얻기

```java
    }catch(Exception e){
        // 예외가 가지고 있는 메시지 얻기
        String message = e.getMessage();
        
        // 예외의 발생 경로를 추적
        e.printStackTrace();
    }
```

---

예외 처리는 꼭 제대로 해주도록 하자. 예전에 유지보수 하는데, 다른 개발자가 catch에서 예외를 잡아놓기만하고 따로 처리를 해주지 않아서 원인 찾느라 고생했던 기억이 있다.

> 출처(참고) : 한빛미디어 이것이 자바다(저자 신용권)