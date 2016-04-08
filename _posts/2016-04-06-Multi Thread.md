---
layout: post
title: Multi Thread(멀티 스레드)에 대하여
categories: java
---

이번에 Java를 전체적으로 다시 한 번 보는데에는, Java8에 추가된 것들을 보기 위한 목적도 있지만 업무를 진행하면서 제대로 경험해보지 못했고, 내 자신이 가장 약하다고 생각되는 부분인 스레드, 네트워크, 파일 입출력에 대한 부분을
다시 확인해보고자 하는 것도 있었다. 그 중에 이번에는 Thread와 관련된 부분을 전체적으로 정리 하려고 한다.

```
* 목차
1. 멀티 스레드 개념
2. 작업 스레드 생성과 실행
3. 스레드 우선순위
4. 동기화 메소드와 동기화 블록
5. 스레드 상태
6. 스레드 상태 제어
7. 데몬 스레드
8. 스레드 그룹
9. 스레드 풀
```

# 1. 멀티 스레드 개념

## 1.1. 프로세스와 스레드

* 프로세스(Process) : 실행 중인 하나의 애플리케이션(애플리케이션인 Chrome 브라우저를 두 개 실행 시키면, 두 개의 Chrome Process가 생성된 것)
* 멀티 스레드(Multi thread) : 하나의 Process가 두 가지 이상의 작업을 처리할 수 있도록 하는 것.

## 1.2. 메인 스레드

* 자바 애플리케이션은 메인 스레드가 main( ) 메소드를 실행하면서 시작됨.
* 메인 스레드에서 다른 작업 스레드들을 만들어서 실행할 수 있으며, 모든 스레드가 종료되어야 프로세스가 종료됨.

# 2. 작업 스레드 생성과 실행

## 2.1. 작업 스레드 생성과 실행

### 2.1.1. java.lang.Thread 클래스로부터 직접 생성

* Runnable 인터페이스를 구현하는 방법. 

```java
public class Task implements Runnable{

    public void run(){
        // 스레드가 실행할 코드;
    }

}
```

```java
Runnable task = new Task();
Thread thread = new Thread(task);
```

* 익명 구현 객체를 사용하는 방법

```java
Thread thread = new Thread(new Runnable(){
    public void run(){
        // 스레드가 실행할 코드;
    }
});
```

* Java8의 람다식을 사용하는 방법

```java
Thread thread = new Thread(()->{
    // 스레드가 실행할 코드;
});
```

* Thread 시작

```java
thread.start();
```

### 2.1.2. Thread 하위 클래스로부터 생성

* Thread 클래스를 상속한 후 run( ) 메소드를 재정의 하는 방법이다.

```java
public class myThread extends Thread{
    @Override
    public void run(){
        // 스레드가 실행할 코드
    }
}
```

* 아래는 익명 Thread 익명 객체로 생성하는 방법이다.

```java
Thread thread = new Thread(){
    @Override
    public void run(){
        // 스레드가 실행할 코드
    }
}
```

### 2.1.3. 스레드의 이름

Thread 클래스의 setName("스레드 이름") 메소드를 사용해서 변경하면 된다.

```java
thread.setName("스레드 이름");   // 스레드 이름 설정
thread.getName();             // 스레드의 이름 가져오기
```

현재 스레드의 객체 참조를 얻는 방법이다.

```java
Thread thread = Thread.currentThread();
```

# 3. 스레드 우선순위

* 동시성(Concurrency) : 멀티 작업을 위해 하나의 코어에서 멀티 스레드가 번갈아가며 실행하는 것.
* 병렬성(Paralleism) : 멀티 작업을 위해 멀티 코어에서 각각의 스레드를 동시에 실행하는 것.
* 스레드 스케쥴링 : 코어의 수 < 스레드의 수 인 경우, 스레드를 어떤 순서에 의해 동시성으로 실행할 것인가를 결정하는 것.
    * 우선순위(Priority) 방식
        * 우선 순위가 높은 스레드가 더 많이 실행 상태를 가지도록 하는 것
        * 개발자가 우선 순위 번호를  `thread.setPriority(1(낮음) ~ 5(기본) ~ 10(높음));`로 부여할 수 있다. 
    * 순환 할당(Round-Robin) 방식
        * 시간 할당량(Time Slice)를 정해서 정해진 시간만큼만 각각의 스레드를 실행시키는 방법, JVM에 의해서 정해지기 때문에 제어 불가능.

# 4. 동기화 메소드와 동기화 블록

## 4.1. 공유 객체를 사용할 때의 주의점

멀티 스레드 애플리케이션의 경우 스레드들이 객체(아래에서 Calculator)를 공유해서 작업하는 경우 문제가 생길 수 있다.

```java

// main
public class MainThreadExample{
    public static void main(String[] args){
        
        Calculator calculator = new Calculator();   // 공유하게 되는 객체
        
        User1 user1 = new User1();
        user1.setCalculator(calculator);
        user1.start();
        
        User2 user2 = new User2();
        user2.setCalculator(calculator);
        user2.start();
        
    }
}

// 공유 객체
public class Calculator{
    private int memory;
    
    public int getMemory(){
        return memory;
    }
    
    public void setMemory(int memory){
        this.memory = memory;
        
        try{
            Thread.sleep(2000);
        }catch(InterruptedException e){
            System.out.println(Thread.currentThread().getName() + " : " + this.memory);
        }
    }
}

// User1
public class User1 extends Thread{
    private Calculator calculator;
    
    public void setCalculator(Calculator calculator){
        this.setName("CalculatorUser1");
        this.calculator = calculator;
    }
    
    public void run(){
        calculator.setMemory(100);  // 공유 객체에 100 저장
    }
    
}


// User2
public class User2 extends Thread{
    private Calculator calculator;
    
    public void setCalculator(Calculator calculator){
        this.setName("CalculatorUser2");
        this.calculator = calculator;
    }
    
    public void run(){
        calculator.setMemory(50);   // 공유 객체에 50 저장
    }
    
}

```

## 4.2. 동기화 메소드 및 동기화 블록

스레드가 사용 중인 객체를 다른 스레드가 변경할 수 없도록 객체에 잠금을 걸어야 한다. synchronized 키워드를 사용하여 동기화 블록을 만든 후 단 하나의 스레드만 실행되도록 하면 만들어 주면 된다.
 
```java
public synchronized void method(){
    // 단 하나의 스레드만 실행 가능
}
```

또는 

```java
public void method(){
    
    // 여러 스레드가 실행 가능
    
    synchronized(공유객체){
        // 단 하나의 스레드만 실행 가능
    }
    
    // 여러 스레드가 실행 가능
    
}
```


# 5. 스레드 상태

스레드 객체 생성 -> start( ) 메소드 호출 -> 실행 대기 상태(스케줄링이 되지 않아서 실행을 기다리고 있음) -> 실행 상태 (스케줄링으로 선택된 스레드가 CPU를 점유하고 run( ) 메소드를 실행) -> 실행 대기와 실행 상태를 반복하면서 run( ) 메소드를 조금씩 실행하여 run( ) 메소드 종료 -> 종료 상태

Thread 클래스의 getState() 메소드를 사용하여 스레드의 상태를 알 수 있다.

|상태      |열거상수         |설명                                              |
|---------|---------------|------------------------------------------------|
|객체 생성  |NEW            |스레드 객체가 생성되었으나 start() 메소드가 호출되지 않은 상태|
|실행 대기  |RUNNABLE       |실행 상태로 언제든지 갈 수 있는 상태                     |
|일시정지   |WAITING        |다른 스레드가 통지할 때까지 기다리는 상태                  |
|일시정지   |TIMED_WAITING  |주어진 시간 동안 기다리는 상태                          |
|일시정지   |BLOCKED        |사용하고자 하는 객체의 락이 풀릴 때까지 기다리는 상태         |
|종료      |TERMINATED     |실행을 마친 상태                                     |

# 6. 스레드 상태 제어

아래 내용에 나오는 것들은 모두 Thread 클래스의 메소드들이다.

## 6.1. 주어진 시간동안 일시 정지(sleep( ))

```java
try{
    Thread.sleep(3*1000);   // 3초 동안 일시 정지 상태로 있는다.
}catch(InterruptedException e){
    // interrupt() 메소드가 호출되면 실행 된다.
}
```

## 6.2. 다른 스레드에게 실행 양보(yield( ))

yield( ) 메소드를 호출한 스레드는 `실행 대기 상태`로 돌아가고, 동일한 우선순위 또는 높은 우선순위를 갖는 다른 스레드가 실행 기회를 가질 수 있게 해준다.

```java
public class YieldExample{
    public static void main(String[] args){
        ThreadA threadA = new ThreadA();
        ThreadB threadB = new ThreadB();
        
        // ThreadA, ThreadB 모두 실행
        threadA.start();
        threadB.start();
        
        try{
            Thread.sleep(3000);
        }catch(InterruptedException e){
            // ...    
        }
        threadA.work = false;       // ThreadB만 실행
        
        try{
            Thread.sleep(3000);
        }catch(InterruptedException e){
            // ...    
        }
        threadA.work = true;        // ThreadA, ThreadB 모두 실행
        
        try{
            Thread.sleep(3000);
        }catch(InterruptedException e){
            // ...    
        }
        threadA.stop = true;        // ThreadA, ThreadB 모두 종료
        threadB.stop = true;
        
    }
}
```

```java
public class ThreadA extends Thread{
    public boolean stop = false;    // 종료 플래그
    public boolean work = true; // 작업 진행 여부 플래그
    
    public void run(){
        while(!stop){
            if(work){
                // 작업 내용
            }else{
                Thread.yield(); // work가 false가 되면, 다른 스레드에게 실행을 양보
            }
        }
        System.out.println("ThreadA 종료");
    }
}

public class ThreadB extends Thread{
    public boolean stop = false;    // 종료 플래그
    public boolean work = true; // 작업 진행 여부 플래그
    
    public void run(){
        while(!stop){
            if(work){
                // 작업 내용
            }else{
                Thread.yield(); // work가 false가 되면, 다른 스레드에게 실행을 양보
            }
        }
        System.out.println("ThreadB 종료");
    }
}
```
## 6.3. 다른 스레드의 종료를 기다림(join( ))

다른 스레드가 종료될 때까지 기다렸다가 실행해야 하는 경우에 사용한다.

```java
public class JoinExample{
    public static void main(String[] args){
        SumThread sumThread = new SumThread();
        sumThread.start();
        
        try{
            sumThread.join();   // sumThread가 종료할 때까지 main스레드를 일시 정지시킨다.
        }catch(InterruptedException e){
        
        }
        
        System.out.println("결과 : " + sumThread.getSum(); 
    }
}
```

## 6.4. 스레드 간의 협업(wait( ), notify( ), notifyAll( ))

아래의 메소드들은 Thread 클래스가 아닌 Object 클래스의 메소드이므로, 모든 공유 객체에서 호출이 가능하고, 동기화 메소드 또는 동기화 블록 내에서만 사용이 가능하다.

* notify( ) : wait( )에 의해서 일시 정지된 스레드 중 한 개를 실행 대기 상태로 만든다.
* notifyAll( ) : wait( )에 의해서 일시 정지된 모든 스레드들을 실행 대기 상태로 만든다.
* wait( ) : 자신을 일시 정지 상태로 만든다.

아래는 공유 객체를 사용해서 데이터를 읽고, 저장하는 예제이다.

```java
public class DataBox{
    private String data;
    
    public synchronized String getData(){
        if(this.data == null){
            try{
                wait();     // data가 null인 경우, 스레드를 일시 정지 상태로 만든다.
            }catch(InterruptedExcetion e){
                // ...
            }
        }
            
        String returnValue = data;
        System.out.println("ConsumerThread가 읽은 데이터 : " + returnValue);
        
        data = null;
        notify();       // ProducerThread를 실행 대기 상태로 만듬
        return returnValue;
        }
    }
    
    public synchronized String setData(String data){
        if(this.data != null){
            try{
                wait();     // data가 null이 아니면, 스레드를 일시 정지 상태로 만든다.
            }catch(InterruptedExcetion e){
                // ...
            }
        }
            
        this.data = data;
        System.out.println("ProducerThread가 생성한 데이터 : " + data);
        notify();       // wait()에 의해서 일시 정지된 Thread를 실행 대기 상태로 만듬
        
        }
    }
}
```

데이터를 생성하고 저장하는 클래스

```java
public class ProducerThread extends Thread{
    private DataBox dataBox;
    
    public ProducerThread(DataBox dataBox){
        this.dataBox = dataBox;
    }
    
    @Override
    public void run(){
        for(int i=1; i<=3; i++){
            String data = "Data - "+ i;
            dataBox.setData(data);
        }
    }
}
```

데이터를 읽어들이는 클래스

```java
public class ConsumerThread extends Thread{
    private DataBox dataBox;
    
    public ConsumerThread(DataBox dataBox){
        this.dataBox = dataBox;
    }
    
    @Override
    public void run(){
        for(int i=1; i<=3; i++){
            String data = dataBox.getData();
        }
    }
}
```

두 스레드를 생성하고 실행하는 메인 스레드

```java
public class WaitNotifyExample{
    public static void main(String[] args){
        DataBox dataBox = new DataBox();
        
        ProucerThread producerThread = new ProducerThread(dataBox);
        ConsumerThread consumerThread = new ConsumerThread(dataBox);
        
        producerThread.start();
        consumerThread.start();
        
    }
}
```

## 6.5. 스레드의 안정한 종료(stop 플래그와 interrupt() 메소드)

실행 중인 스레드를 즉시 종료해야하는 경우 두 가지 방법을 사용할 수 있다.

### stop 플래그를 이용하는 방법

```java
public class StopFlagExample{
    public static void main(String[] args){
        PrintThread1 printThread = new PrintThread1();
        printThread.start();
        
        try{ 
            Thread.sleep(1000);
        }catch(InterruptedException e){
            // ...
        }
        
        printThread.setStop(true);  // 스레드를 종료시키기 위해 stop 필드를 true로 변경
        
    }
}
```

```java
public class PrintThread1 extends Thread{
    private boolean stop;
    
    public void setStop(boolean stop){
        this.stop = stop;
    }
    
    public void run(){
        while(!stop){
            System.out.println("실행 중");
        }
        
        System.out.println("자원 정리");
        System.out.println("실행 종료");
    }
    
}
```

### interrupt( ) 메소드를 사용하는 방법

스레드가 일시 정지 상태에 있는 경우, interrupted( )메소드를 사용해서 InterruptedException 을 발생시켜 run( ) 메소드를 정상 종료시키는 방법이다.

```java
public class InterruptExample{
    public static void main(String[] args){
        Thread thread = new PrintThread2();
        thread.start();
        
        try{
            Thread.sleep(1000);
        }catch(InterruptedException e){
            // ...
        }
        
        thread.interrupt();     // PrintThread2를 종료시키기 위해 InterruptedException을 발생 시킨다.
        
    }
}
```

`Thread.interrupted();` 메소드를 사용해서 현재 스레드가 interrupted 되었는지 확인하거나, 인스턴스 메소드인 `objThread.isInterrupted();`를 사용해서 현재 스레드가 interrupted 되었는지 확인할 수 있다.
 
 ```java
 public class PrintThread2 extends Thread{
    public void run(){
        while(true){
            System.out.println("실행 중");
            if(Thread.interrupted()){
                break;
            }
        }
        
        System.out.println("자원 정리");
        System.out.println("실행 종료");
        
    }
 }
 ```

# 7. 데몬 스레드

주 스레드의 작업을 돕는 보조적인 역할을 수행하며, 주 스레드가 종료되면 데몬 스레드는 강제적으로 종료된다.

```java
public class AutoSaveThread extends Thread{
    public void save(){
        System.out.println("자동 저장 합니다.");
    }
    
    @Override
    public void run(){
        while(true){
            try{
                Thread.sleep(1000);
            }catch(InterruptedException e){
                break;
            }
            save();
            
        }
    }
}
```

```java
public class DaemonExample{
    public static void main(String[] args){
        AutoSaveThread autoSaveThread = new AutoSaveThread();
        autoSaveThread.setDaemon(true);     // AutoSaveThread를 데몬 스레드로 만든다.
        autoSaveThread.start();
        
        try{
            Thread.sleep(3000);
        }catch(InterruptedException e){
        
        }
        
        System.out.println("메인 스레드 종료");
    }
}
```

# 8. 스레드 그룹

JVM이 실행되면 JVM 운영에 필요한 스레드들을 생성해서 system 스레드 그룹에 포함시키고, system의 하위 스레드 그룹으로 main을 만들고 main 스레드 그룹에 포함시킨다.

## 8.1. 스레드 그룹 이름 얻기

```java
ThreadGroup group = Thread.currentThread.getThreadGroup();
String groupName = group.getName();
```

Thread의 static 메소드인 `getAllStackTraces()`를 이용하면 프로세스 내에서 실행하는 모든 스레드에 대한 정보를 얻을 수 있다.

```java
Map<Thread, StackTraceElement[]> map = Thread.getAllStackTraces();
Set<Thread> threads = map.keySet();
for(Thread thread : threads){
    System.out.println("Name : " + thread.getName() + ((thread.isDaemon()) ? "데몬" : "주"));
    System.out.println("스레드 그룹 : thread.getThreadGroup().getName());
}
```

## 8.2. 스레드 그룹 생성

스레드 그룹을 먼저 생성하고,

```java
ThreadGroup tg = new ThreadGroup(String name);  // 현재 스레드가 속한 그룹의 하위 그룹으로 생성됨.
ThreadGroup tg = new ThreadGroup(ThreadGroup parent, String name);  // 지정해준 스레드 그룹의 하위 스레드 그룹으로 생성됨
```

스레드 생성시, 스레드가 들어갈 스레드 그룹을 지정해주면 된다.

```java
Thread t = new Thread(ThreadGroup group, Runnable target, target());
Thread t = new Thread(ThreadGroup group, Runnable target, String name);  // name : 스레드의 이름
Thread t = new Thread(ThreadGroup group, String name);
```

## 8.3. 스레드 그룹의 일괄 interrupted( )

ThreadGroup이 제공하는 interrupt( ) 메소드를 이용하면 그룹에 포함된 모든 스레드들을 interrupt 할 수 있다.

|리턴        |메소드                   |설명                                           |
|-----------|-----------------------|----------------------------------------------|
|int        |activeCount()          |현재 그룹 및 하위 그룹에서 활동 중인 모든 스레드의 수를 리턴|
|int        |activeGroupCount()     |현재 그룹에서 활동 중인 모든 하위 그룹의 수를 리턴|
|void       |checkAccess()          |현재 스레드가 스레드 구룹을 변경할 권한이 있는지 체크, 없다면 SecurityException을 발생 시킴|
|void       |destroy()              |현재 그룹 및 하위 그룹을 모두 삭제한다. 단 그룹 내에 포함된 모든 스레드들이 종료 상태가 되어야 한다.|
|boolean    |isDestroyed()          |현재 그룹이 삭제되었는지 여부를 리턴|
|int        |getMaxPriority()       |현재 그룹에 포함된 스레드가 가질 수 있는 최대 우선순위를 리턴|
|void       |setMaxPriority(int pri)|현재 그룹에 포함된 스레드가 가질 수 있는 최대 우선순위를 설정한다.|
|String     |getName()              |현재 그룹의 이름을 리턴|
|ThreadGroup|getParent()            |현재 그룹의 부모 그룹을 리턴|
|boolean    |parentOf(ThreadGroup g)|파라미터인 ThreadGroup이 부모인지를 리턴|
|boolean    |isDaemon()             |현재 그룹이 데몬 그룹인지 여부를 리턴|
|void       |setDaemon(boolean daemon)|현재 그룹을 데몬 그룹으로 설정|
|void       |list()                 |현재 그룹에 포함된 스레드와 하위 그룹에 대한 정보를 출력|
|void       |interrupt()            |현재 그룹에 포함된 모든 스레드들을 interrupt 한다|


# 9. 스레드 풀

갑작스런 병렬 작업의 증가로 인한 스레드의 폭증을 막기 위해서는 스레드풀(ThreadPool)을 사용해야 한다.

## 9.1. 스레드풀 생성 및 종료

### 스레드풀 생성

`java.util.concurrent` 패키지에 ExecutorService 인터페이스와 ExecutorService 구현 객체를 만들 수 있는 Executors 클래스를 제공해준다. 
ExecutorService 구현 객체는 Executors 클래스의 다음 두 가지 메소드 중 하나를 이용해서 생성할 수 있다.

* 초기 스레드 수 : ExecutorService 객체가 생성될 때 기본적으로 생성되는 스레드 수
* 코어 스레드 수 : 최소한 유지해야 할 스레드의 수

|메소드                               |초기 스레드 수|코어 스레드 수|최대 스레드 수       |
|-----------------------------------|-----------|-----------|-----------------|
|newCachedThreadPool()              |0          |          0|Integer.MAX_VALUE|
|newFixedThreadPool(int nThreads)   |0          |nThreads   |nThreads         |


아래 메소드로 생성한 스레드풀은 1개 이상의 스레드가 추가 되었을 경우 60초 동안 추가된 스레드가 아무 작업을 하지 않으면 추가된 스레드를 종료하고 풀에서 제거한다.

```java
ExecutorService executorService = Excutors.newCachedThreadPool(); 
```

아래 메소드로 생성한 스레드풀은 스레드가 작업을 처리하지 않고 놀고 있더라도 스레드 개수는 줄지 않는다.

```java
ExecutorService executorService = Executors.newFixedThreadPool(
    // CPU 코어의 수만큼 최대 스레드를 사용하는 스레드풀을 생성
    Runtime.getRuntime().availableProcessors() 
);
```

아래는 직접 ThreadPoolExecutor 객체를 생성하는 방법이다.

예)

* 초기 스레드 개수 : 0
* 최대 스레드 개수 : 100
* 코어 스레드 3개를 제외한 나머지 추가된 스레드가 120초 동안 놀고 있을 경우 해당 스레드를 제거

```java
ExecutorService threadPool = new ThreadPoolExecutor(
    3,      // 코어 스레드 개수
    100,    // 최대 스레드 개수
    120L,   // 놀고 있는 시간
    TimeUnit.SECONDS,   // 놀고 있는 시간 단위
    new SynchronousQueue<Runnable>()    // 작업 큐
);
```

### 스레드풀 종료

스레드풀의 스레드는 기본적으로 데몬 스레드가 아니기 때문에 main 스레드가 종료되더라도 계속 실행 상태로 남아있다. 
그렇기 때문에 애플리케이션을 종료하려면 스레드풀을 종료시키는 작업을 해줘야한다. ExecutorService는 종료와 관련해서 세 개의 메소드를 제공해준다.

|리턴 타입        |메소드명                                          |설명|
|---------------|-----------------------------------------------|----|
|void           |shutdown()                                     |작업 큐에 대기하고 있는 모든 작업을 처리한 뒤에 스레드풀을 종료시킨다.|
|List<Runnable> |shutdownNow()                                  |현재 작업 처리 중인 스레드를 interrupt해서 작업 중지를 시도하고 스레드풀을 종료 시킨다. 리턴값은 작업 큐에 있는 미처리된 작업의 목록이다.|
|boolean        |awaitTermination(long timeout, TimeUnit unit)  |shutdown() 메소드 호출 이후, 모든 작업 처리를 timeout 시간 내에 완료하면 true, 완료하지 못하면 interrupt하고 false를 리턴한다.|

## 9.2. 작업 생성과 처리 요청

### 작업 생성

* Runnable 구현 클래스 : 작업 완료 후 리턴값이 없다.

```java
Runnable task = new Runnable(){
    @Override
    public void run(){
        // 스레드가 처리할 작업 내용
    }
}
```

* Callable 구현 클래스 : 작업 완료 후 리턴값이 존재한다. `implements Callable<T>` 에서 지정한 T 타입이 리턴값이다.

```java
Callable<T> task = new Callable<T>{
    @Override
    public T call() throws Exception{
        // 스레드가 처리할 작업 내용
        return T;
    }
}
```

### 작업 처리 요청

ExecutorService의 작업 큐에 Runnable 또는 Callable 객체를 넣는 행위이다. ExecutorService는 작업 처리 요청을 위해 execute() 메소드와 submit() 메소드를 제공하지만,
**스레드의 생성 오버헤더를 줄이기 위해 submit()을 사용하는 것이 좋다.**

* submit( )
    * 작업 처리 결과를 받을 수 있도록 Future를 리턴한다.
    * 작업 처리 도중 예외가 발생하더라도 스레드는 종료되지 않고 다음 작업을 위해 재사용된다.
    
|리턴 타입        |메소드                             |
|---------------|---------------------------------|
|Future&lt;?&gt;|submit(Runnable task)            |
|Future&lt;V&gt;|submit(Runnable task, V result)  |
|Future&lt;V&gt;|submit(Callable<V> task)         |

## 9.3. 블로킹 방식의 작업 완료 통보

ExecutorService의 submit() 메소드는 스레드풀의 작업 큐에 저장하고 즉시 Future 객체를 리턴한다. Future 객체는 작업이 완료될때까지 기다렸다가 최종 결과를 얻는데 사용된다.
Future의 get() 메소드를 호출하면 스레드가 작업을 완료할 때까지 블로킹(지연)되었다가 작업이 완료되면 그 처리 결과를 리턴한다.

Future 객체가 제공하는 메소드

|리턴 타입|메소드                            |설명|
|-------|--------------------------------|---|
|V      |get()                           |작업이 완료될 때까지 블로킹되었다가 처리 결과 V를 리턴|
|V      |get(long timeout, TimeUnit unit)|timeout 시간 전에 작업이 완료되면 결과 V를 리턴, 완료되지 않으면 TimeoutException을 발생시킴|
|boolean|cancel(boolean mayInterruptIfRunning|작업 처리가 진행 중일 경우 취소시킴|
|boolean|isCancelled()                   |작업이 취소 되었는지 여부|
|boolean|isDone()                        |작업 처리가 완료되었는지 여부|

Future를 이용한 블로킹 방식의 작업 완료 통보에는 작업을 처리하는 스레드가 작업을 완료하기 전까지는 get()메소드가 블로킹되므로 다른 코드를 실행할 수 없다. 
그렇기 때문에 새로운 스레드이거나 스레드풀의 또 다른 스레드를 사용해서 get() 메소드를 호출해야한다.

### 리턴값이 없는 작업 완료 통보

결과값이 없는 작업 처리 요청은 submit(Runnable task) 메소드를 이용하면 되는데, 결과값이 없음에도 불구하고 Future 객체를 리턴한다. 이것은 스레드가 작업 처리를 정상적으로 완료했는지, 아니면 작업 처리 도중에 예외가 발생했는지
확인하기 위해서이다.

* 작업 처리의 결과
    * 정상적으로 완료된 경우 : Future의 get() 메소드가 null을 리턴한다.
    * 작업 처리 도중에 interrupt된 경우 InterruptedException을 발생시키고, 예외가 발생한다면 ExecutionException을 발생시킨다.  

```java
Future future = executorService.submit(task);
```

필요한 예외 처리 코드

```java
try{
    future.get();
}catch(InterruptedException e){
    // 작업 처리 도중 스레드가 interrupt 될 경우 실행할 코드
}catch(ExecutionException e){
    // 작업 처리 도중 예외가 발생된 경우 실행할 코드
}
```

리턴값이 없고 단순히 1~10까지의 합을 출력하는 예제

```java
public class NoResultExample{
    public static void main(String[] args){
        
        ExecutorService executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );
        
        System.out.println("[작업 처리 요청]");
        
        Runnable runnable = new Runnable(){
            @Override
            public void run(){
                int sum = 0;
                for(int i=1; i<=10; i++){
                    sum += i;
                }
                System.out.println("[처리 결과] " + sum);
            }
        };
        
        Future future = executorService.submit(runnable);
        
        try{
            future.get();
            System.out.println("[작업 처리 완료]");
        }catch(Exception e){
            System.out.println("[실행 예외 발생] " + e.getMessage());
        }
        
        executorService.shutdown(); // 작업 큐에 대기하고 있는 모든 작업을 처리한 뒤에 스레드풀을 종료
        
    }
}
```

### 리턴값이 있는 작업 완료 통보

리턴값이 필요한 경우 작업 객체를 Callable로 생성하면 된다.

1~10까지의 합을 리턴하는 예제

```java
public class ResultByCallableExample{
    public static void main(String[] args){
        
        // 스레드풀 생성
        ExecutorService executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );
        
        System.out.println("[작업 처리 요청]");
                
        Callable callable = new Callable<Integer>(){
            @Override
            public Integer call() throws Exception{
                int sum = 0;
                for(int i=1; i<=10; i++){
                    sum += i;
                }
                
                return sum;
            }
        };
        
        Future<Integer> future = executorService.submit(runnable);
        
        try{
            int sum = future.get();
            System.out.println("[처리 결과] " + sum);
            System.out.println("[작업 처리 완료]");
        }catch(Exception e){
            System.out.println("[실행 예외 발생] " + e.getMessage());
        }
        
        executorService.shutdown(); // 작업 큐에 대기하고 있는 모든 작업을 처리한 뒤에 스레드풀을 종료
        
    }
}
```

### 작업 처리 결과를 외부 객체에 저장

외부 Result 객체에 두 개 이상의 스레드 작업을 취합할 목적으로 사용하는 경우. 생성자를 통해 Result 객체를 주입받아야 한다.

```java
public class ResultByRunnableExample{
    public static void main(String[] args){
        // 스레드풀 생성
        ExecutorService executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );
        
        System.out.println("[작업 처리 요청]");
        
        Result result = new Result();
        Runnable task1 = new Task(result);
        Runnable task2 = new Task(result);
        
        // 스레드풀의 작업 큐에 저장
        Future<Result> future1 = executorService.submit(task1, result);
        Future<Result> future2 = executorService.submit(task2, result);
        
        try{
            result = future1.get();
            result = future2.get();
            System.out.println("[처리 결과] " + result.accumValue);
            System.out.println("[작업 처리 완료]");
        }catch(Exception e){
            e.printStackTrace();
            System.out.println("[실행 예외 발생] " + e.getMessage());
        }
            
        executorService.shutdown(); // 작업 큐에 대기하고 있는 모든 작업을 처리한 뒤에 스레드풀을 종료
        
    }
}
```

```java
public class Task implements Runnable{
    Result result;
    public Task(Result result){
        this.result = result;
    }
    
    @Override
    public void run(){
        int sum = 0;
        for(int i=0; i<=10; i++){
            sum += i;
        }
        result.addValue(sum);       // Result 객체에 작업 결과 저장
    }
}
```

```java
public class Result{
    int accumValue;
    synchronized void addValue(int value){
        accumValue += value;
    }
}
```

### 작업 완료 순으로 통보

처리 결과를 순차적으로 이용할 필요가 없다면 CompletionService의 poll()과 take() 메소드를 이용하여 스레드풀에서 작업 처리가 완료된 것만 통보받아 사용할 수 있다.

|리턴 타입        |메소드                               |설명|
|---------------|-----------------------------------|---|
|Future&lt;V&gt;|poll()                             |완료된 작업의 Future를 가져오고, 완료된 작업이 없다면 즉시 null을 리턴한다|
|Future&lt;V&gt;|poll(long timeout, TimeUnit unit)  |완료된 작업의 Future를 가져오고, 완료된 작업이 없다면 timeout까지 블로킹 된다|
|Future&lt;V&gt;|take()                             |완료된 작업의 Future를 가져오고, 완료된 작업이 없다면 있을 때까지 블로킹 된다|
|Future&lt;V&gt;|submit(Callable&lt;V&gt; task)     |스레드풀에 Callable 작업 처리 요청|
|Future&lt;V&gt;|submit(Runnable task, V result)    |스레드풀에 Runnable 작업 처리 요청|

```java
public class CompletionServiceExample extends Thread{
    public static void main(String[] args){
        // 스레드풀 생성
        ExecutorService executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );
        
        // CompletionService 생성
        CompletionService<Integer> completionService =
            new ExecutorCompletionService<Integer>(executorService);
        
        System.out.println("[작업 처리 요청]");
        
        for(int i=0; i<3; i++){
            // 스레드풀에게 작업 처리 요청
            completionService.submit(new Callable<Integer>(){
                @Override
                public Integer call() throws Exception{
                    int sum = 0;
                    for( int i=0; i<=10; i++){
                        sum += i;
                    }
                    return sum;
                }
            });
        }
        
        System.out.println("[처리 완료된 작업 확인]");
        
        // 스레드풀의 스레드에서 실행하도록 함
        executorService.submit(new Runnable(){
            @Override
            public void run(){
                while(true){
                    try{
                        Future<Integer> future = completionService.take();  // 완료된 작업 가져오기
                        int value = future.get();
                        System.out.println("[처리 결과] " + value);
                    }catch(Exception e){
                        break;
                    }
                }
            }
        });
        
        
        try{
            Thread.sleep(3000);
        }catch(InterruptedException e){
        
        }
        
        executorService.shutdownNow();  // 3초 뒤에 take()에서 InterruptedException이 발생하고 catch 절에서 break가 되어 while문을 종료함.
        
    }
}
```

## 9.4. 콜백 방식의 작업 완료 통보

* callback : 애플리케이션이 스레드에게 작업 처리를 요청한 후, 스레드가 작업을 완료하면 특정 메소드를 자동 실행하는 기법.

NIO 패키지에 포함되어 있는 비동기 통신에서 콜백 객체를 만들 때 사용되는 `java.nio.channels.CompletionHanlder`를 이용해서 콜백 메소드 기능을 구현할 수 있다.

CompletionHandler 객체를 생성하는 코드 

```java
// V : 결과값 타입, A : 결과값 이외에 추가적으로 전달하는 객체, 필요 없다면 Void로 지정가능하다.
CompletionHandler<V, A> callback = new CompletionHandler<V, A>(){
    @Override
    public void completed(V result, A attachment){
        // 작업을 정상 처리 완료했을 때
    }
    
    @Override
    public void failed(Throwable exc, A attachment){
        // 작업 처리 도중 예외가 발생했을 때
    }
}
```

작업 처리 결과에 따라 콜백 메소드를 호출하는 Runnable 객체

```java
Runnable task = new Runnable(){
    @Override
    public void run(){
        try{
            // 작업 처리
            V result = ...;
            callback.completed(result, null);
        }catch(Exception e){
            callback.failed(e, null);
        }
            
    }
}
```

두 개의 문자열을 정수화해서 더하고 결과를 콜백 방식으로 통보하는 예제

```java
public class CallbackExample{
    private ExecutorService executorService;
    
    public CallbackExample(){
        executorService = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors();
        );
    }
    
    // 콜백 메소드를 가진 CompletionHandler 객체 생성. Integer : 결과 타입, Void : 첨부 타입
    private CompletionHandler<Integer, Void> callback = new CompletionHandler<Integer, Void>(){
        @Override
        public void completed(Integer result, Void attachment){
            System.out.println("completed() 실행 : " + result);
        }
        
        @Override
        public void failed(Throwable exc, Void attachment){
            System.out.println("failed() 실행 : " + exc.toString());
        }
    
    };
    
     public void doWork(final String x, final String y){
        
        // 작업을 정의
        Runnable task = new Runnable(){
            @Override
            public void run(){
                try{
                    int intX = Integer.parseInt(x);
                    int intY = Integer.parseInt(y);
                    int result = intX + intY;
                    callback.completed(result, null);
                }catch(NumberFormatException e){
                    callback.failed(e, null);
                }
            }
        };
        
        // 스레드풀에게 작업 요청
        executorService.submit(task);
        
    }
    
    public void finish(){
        executorService.shutdown(); // 스레드풀 종료
    }
    
    public static void main(String[] args){
        CallbackExample example = new CallbackExample();
        example.doWork("3", "3");
        example.doWork("3", "삼");
        example.finish();
    }
    
}
```