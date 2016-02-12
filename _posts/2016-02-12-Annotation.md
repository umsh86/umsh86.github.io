---
layout: post
title: Annotation
categories: java
---

# 1. Annotation

어노테이션은 컴파일 과정과 실행 과정에서 코드를 어떻게 컴파일하고 처리할 것인지를 알려주는 정보이고, 아래의 세 가지 용도로 사용된다

* 컴파일러에게 코드 문법 에러를 체크하도록 정보를 제공
* 소프트웨어 개발 툴이 빌드나 배치 시 코드를 자동으로 생성할 수 있도록 정보를 제공
* 실행 시(런타임 시) 특정 기능을 실행하도록 정보를 제공

## 1.1. 어노테이션 타입 정의와 적용

`@interface`를 사용해서 어노테이션을 정의한다

```java
public @interface AnnotationName{
}
```

이렇게 정의한 어노테이션은 `@AnnotationName` 으로 사용한다.

어노테이션은 엘리먼트를 멤버로 가질 수 있는데 각각의 엘리먼트는 타입과 이름으로 구성되며 디폴트 값을 가질 수 있다.

```java
public @interface AnnotationName{
    String elementName1();
    int elementName2() default 5;
}
```

이렇게 정의한 어노테이션을 코드에서 사용할 때는 아래와 같이 사용한다

```java
@AnnotationName(elementName1 = "값", elementName2 = 3)

또는 

@AnnotationName(elementName1 = "값");
```

elementName1은 디폴트 값이 없기 때문에 반드시 값을 넣어줘야 하고, elementName2는 디폴트 값이 있기 때문에 생략이 가능하다.

어노테이션은 기본 엘리먼트인 value를 가질 수 있다

```java
public @interface AnnotationName{
    String value();     // 기본 엘리먼트 선언
    int elementName() default 5;
}
```

```java
@AnnotationName("값"); // value 값으로 자동 설정
```

## 1.2. 어노테이션 적용 대상

어노테이션을 적용할 수 있는 대상은 java.lang.annotation.ElementType 열거 상수로 다음과 같이 정의되어 있다.

|ElementType 열거상수|적용 대상|
|------------------|-------|
|TYPE               |클래스, 인터페이스, 열거 타입|
|ANNOTATION_TYPE    |어노테이션|
|FIELD              |필드|
|CONSTRUCTOR        |생성자|
|METHOD             |메소드|
|LOCAL_VARIABLE     |로컬 변수|
|PACKAGE            |패키지|

어노테이션이 적용될 대상을 지정할 때에는 @Target 어노테이션을 사용하며, @Target의 기본 엘리먼트인 value는 ElementType 배열을 값으로 가진다.

```java
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD})
public @interface AnnotationName{
}
```

다음과 같이 클래스, 필드, 메소드만 어노테이션을 적용할 수 있고 생성자에는 적용할 수 없다

```java
@AnnotationName
public class ClassName{
    @AnnotationName
    private String fieldName;
    
    public ClassName(){
    }
    
    @AnnotationName
    public void methodName(){}
}
```

## 1.3. 어노테이션 유지 정책

어노테이션 사용 용도에 따라 어느 범위까지 유지할 것인지를 지정해야 한다. 소스상에서만 유지할 건지 컴파일된 클래스까지 유지할 건지 런타임 시에도 유지할 건지를 지정해야하는데,
 이 어노테이션 유지 정책은 java.lang.annotation.RetentionPolicy 열거 상수로 다음과 같이 정의 되어 있다

|RetentionPolicy 열거 상수|설명|
|-------|-------------------|
|SOURCE |소스상에서만 어노테이션 정보를 유지. 소스 코드를 분석할 때만 의미가 있으며, 바이트 코드 파일에는 정보가 남지 않는다.|
|CLASS  |바이트 코드 파일까지 어노테이션 정보를 유지한다. 하지만 리플렉션을 이용해서 어노테이션 정보를 얻을 수는 없다.|
|RUNTIME|바이트 코드 파일까지 어노테이션 정보를 유지하면서 리플렉션을 이용해서 런타임 시에 에노테이션 정보를 얻을 수 있다.|

**리플렉션(Reflection)** 이란 런타임 시에 클래스의 메타 정보를 얻는 기능을 말하는데, 예를 들어 클래스가 가지고 있는
필드, 생성자 메소드, 적용된 어노테이션이 무엇인지 알아내는 것을 말한다.

이 어노테이션 유지 정책을 적용하기 위해서는 `@Retention`을 사용한다

```java
@Target({ElementType.TYPE, ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface AnnotationName{
}
```

## 1.4. 런타임 시 어노테이션 정보 사용하기

클래스에 적용된 어노테이션 정보를 얻기 위해서 java.lang.Class를 이용하면 되지만, 필드, 생성자, 메소드에 적용된 어노테이션 정보를 얻으려면
Class의 다음 메소드를 통해서 java.lang.reflect 패키지의 Field, Constructor, Method 타입의 배열을 얻어야 한다.

|리턴 타입      |메소드명(매개 변수)      |설명|
|-------------|--------------------|---|
|Field[]      |getFields()         |필드 정보를 Field 배열로 리턴|
|Constructor[]|getConstructors()   |생성자 정보를 Constructor 배열로 리턴|
|Method[]     |getDeclaredMethods()|메소드 정보를 Method 배열로 리턴|

그 다음에 Class, Field, Constructor, Method가 가지고 있는 다음 메소드를 호출해서 적용된 어노테이션 정보를 얻을 수 있다.

|리턴 타입|메소드명(매개 변수)|
|------|---------------|
|boolean| `isAnnotationPresent(Class<? extends Annotation> annotationClass)` 지정한 어노테이션이 적용되었는지 여부. Class에서 호출했을 때 상위 클래스에 적용된 경우에도 true를 리턴함|
|Annotation|`getAnnotation(Class<T> annotationClass)` 지정한 어노테이션이 적용되어 있으면 어노테이션을 리턴하고 그렇지 않다면 null을 리턴한다. Class에서 호출했을 때 상위 클래스에 적용된 경우에도 어노테이션을 리턴한다.|
|Annotation[]| `getAnnotations()` 적용된 모든 어노테이션을 리턴한다. Class에서 호출했을 때 상위 클래스에 적용된 어노테이션도 모두 포함된다. 적용된 어노테이션이 없을 경우 길이가 0인 배열을 리턴|
|Annotation[]|`getDeclaredAnnotations()` 직접 적용된 모든 어노테이션을 리턴한다. Class 에서 호출했을 때 상위 클래스에 적용된 어노테이션은 포함되지 않는다.|

## 1.5. 어노테이션과 리플렉션을 이용한 예제

PrintAnnotation 정의

```java
@Target({ElementType.METHOD})   // 이 어노테이션은 메소드에만 적용 가능
@Retention(RetentionPolicy.RUNTIME) // 런타임 시까지 어노테이션 정보를 유지
public @interface PrintAnnotation{
    String value() default "-";     // 구분선에 사용될 문자
    int number() default 15;        // 반복 출력 횟수
}
```

PrintAnnotation을 적용한 Service 클래스

```java
public class Service{
    @PrintAnnotation
    public void method(){
        System.out.println("실행 내용1");
    }
    
    @PrintAnnotation("*")
    public void method2(){
        System.out.println("실행 내용2");
    }
    
    @PrintAnnotation(value="#", number=20)
    public void method(){
        System.out.println("실행 내용3");
    }
}
```

리플렉션을 이용해서 Service 클래스에 적용된 어노테이션 정보를 읽고 엘리먼트 값에 따라 출력할 문자와 출력 횟수를 콘솔에 출력 후 해당 메소드를 호출하는 PrintAnnotationExample 클래스.

```java
public class PrintAnnotationExample{
    public static void main(String[] args){
        
        // Service 클래스로부터 메소드 정보를 얻음
        Method[] declaredMethods = Service.class.getDeclaredMethods(); // 리플렉션을 통해서 Service 클래스에 선언된 메소드 얻기
        
        // Method 객체를 하나씩 처리
        for(Method method : declaredMethods){
            
            // PrintAnnotation이 적용되었는지 확인
            if(method.isAnnotationPresent(PrintAnnotation.class)){
            
                // PrintAnnotation 객체 얻기
                PrintAnnotation printAnnotation = method.getAnnotation(PrintAnnotation.class));
                
                // 메소드 이름 출력
                System.out.println("[" + method.getName() + "]");
                
                // 구분선 출력
                for(int i=0; i<printAnnotation.number(); i++){
                    System.out.print(printAnnotation.value());
                }
                System.out.println();
                
                try{
                    // Service 객체를 생성하고, 생성된 Service 객체의 메소드를 호출
                    method.invoke(new Service());
                }catch(Exception e){
                    e.printStackTrace();
                }
                System.out.println();
                
                
            }
        }
        
    }
}
```