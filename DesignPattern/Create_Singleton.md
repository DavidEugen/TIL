# 객체 생성과 관련된 패턴 - SingleTon

---

>인스턴스를 오직 한개만 만들어 제공.
>시스템 런타임, 환경 세팅에 대한 정보 등 인스턴스가 하나만 필요 할때

## 구현방법

### 	방법 1.  ```private``` 으로 new 생성자 막기

```java
public class Settings {
  
  private static Settings instance; 
  
  private Settings() { }
  
  public static Settings getInstance() { 
    if (instance == null) {
  		instance = new Settings(); 
    }
  return instance; 
  }
}
```

 생성자를 private 로 만드는 이유

- new 생성자로 생성되는것을 방지

getInstance() 메소드를 static으로 선언한 이유?

- 글로벌 하게 사용하기 위해?

getInstance() 가 멀티 쓰레드 환경에서 안전하지 않은 이유는?

- 쓰레드1이   ``` if (instance == null)``` 을  통과한 상태이고 쓰레드2가  ```if (instance == null) ``` 을 평가하면 인스턴스 생성 전이므로 새로 생성 될 수 있는 문제 있다.
- 

**NOTE**

언뜻 instance는 지역변수가 아니므로 힙 메모리를 참조하고 있으니 뒤에 오는 쓰레드가 instance에 값을 덮을 것 같음. 메모리가 아닌 캐시를 참조한다고 해도 각 쓰레드가 같은 캐시를 참조 할 것 같은데, 각 쓰레드가 다른 instance를 가지는 이유?

1. 쓰레드1이 new Settings()를 호출하고 instance에 할당한 다음 리턴
2. 쓰레드2가 new Settings()를 호출하고 instance에 덮어쓰고 리턴.

이때 쓰레드1을 통해 만들어진 인스턴스는 사라지지 않음. 쓰레드2 생성시의 쓰레드1에서 생성한 인스턴스의 레퍼런스를 가지고 진행된 프로세스 있을 수 있음.

가령 쓰레드1에서 가져간 Settings 인스턴스의 값을 아무리 변경해도 쓰레드2에서 덮어쓴 Settings에 대한 변경 사항만 반영이 될 수도 있음.



#### 문제점

쓰레드 safe 하지 않음



### 방법 2. 동기화 처리

멀티 쓰레드 환경에서 안전하기 위해서는 ``` synchronized ``` 처리해 준다.

```java
public class Settings2 {

    private static Settings2 instance;

    private Settings2() { }

    public static synchronized Settings2 getInstance() {
        if (instance == null) {
            instance = new Settings2();
        }
        return instance;
    }

}
```



멀티 쓰레드 환경에서 getInstance() 로 인스턴스 생성시 lock을 걸기때문에 비용은 발생한다.

만약 성능을 고려하고 싶다면?

>  만약 인스턴스를 만드는 비용이 크지 않다면?
>
>  꼭 나중에 만들어도 되지 않다면...

**NOTE**

1. java의 동기화 블럭 처리 방법은?
2. getInstance() 메소드 동기화시 사용하는 락(lock)은 인스턴스의 락인가 클래스의 락인가? 그 이유는?



#### 문제점

synchronized 로 lock이 걸려 비용 발생



### 방법 3. 이른 초기화 (Eager initialization)

처음 기동간 메모리에 올리는 과정에서 인스턴스를 생성.

``` Syncronized``` 를 쓸 필요가 없다.

```java
public class Settings2 {

    private static Settings2 instance = new Settings();

    private Settings2() { }

    public static Settings2 getInstance() {
        return instance;
    }

}
```

변경을 원하지 않을때는 final 을 붙여주면 좋겠죠. 

```java
private static final Settings2 INSTANCE = new Settings();
...
public static Settings2 getInstance() {
        return INSTANCE;
}
```

Settings2가 로딩 되는 시점에 static field가 초기화 되니깐 그때 인스턴스 생성.

**NOTE**

1. 이른 초기화가 단점이 될 수 있는 이유?

   인스턴스를 만드는 과정이 길고 메모리를 많이 사용한다면 , 애플리케이션 로딩할때 많은 리소스를 썼음에도 불구하고 실제 사용하는것은 한참 이후인경우

2. 만약에 생성자에서 checked 예외를 던진다면 이 코드를 어떻게 변경해야 할까?



#### 문제점

인스턴스 생성이 메모리를 많이 사용하거나 오래 걸릴 겨우.



#### 방법4. Double Checked Locking

필요할때 만들되 ```synchronized```의 비용을 줄이고 싶다면?

멀티 쓰레드를 사용하는 극히 일부인 경우만을 대비하여 체킹

```java
public class Settings3 {

    private static volatile Settings3 instance;

    private Settings3() { }

    public static Settings3 getInstance() {
        if (instance == null) {
            synchronized (Settings3.class) {
                if (instance == null) {
                    instance = new Settings3();
                }
            }
        }
        return instance;
    }
}
```



```volatile```을 써 줘야 함 (JDK1.5 이상)

```getInstance()``` 호출때마다 lock걸리는 게 아니라 null 체크 이후 없는 경우에 lock이 걸리므로 비용을 줄일 수 있다.

**NOTE**

1. double check loking 이라 부르는 이유?
2. instance 변수는 어떻게 정의해야 하는가? 그 이유는?

JDK1.4 이전이거나 이보다 깔끔하게 짜고 싶으면?



#### 문제점

1.4 이전에 못쓰고 ```volatile``` 써야 함. 



#### 방법 5. (권장) [Static member class][1]

멀티 쓰레드에도 안전하고 ```getInstance()``` 호출 시 Settings4Holder 로딩되고 그때 인스턴스 생성.

```java
public class Settings4 {

    private Settings4() { }

    private static class Settings4Holder {
        private static final Settings4 INSTANCE = new Settings4();
    }

    public static Settings4 getInstance() {
        return Settings4Holder.INSTANCE;
    }

}

```



**NOTE**

1. 이 방법은 static final을 썼는데도 왜 지연 초기화(lazy initialization)라고 볼 수 있는가?

   ```getInstance()``` 를 호출 할때 Static member class (여기서는 Holder)를 호출하므로 이때 메모리에 올라가며
   final로 Static member class 의 필드 INSTANCE 를 변경 불가능 하게 만들었으므로

#### 문제점

1)리플렉션과, 2)직렬화 & 역직렬화로 싱글톤 패턴을 깨트릴 수 있다.

##### 리플렉션을 이용해 

```java
Settings settings = Settings.getInstance();
Constructor<Settings> declaredConstructor = Settings.class.getDeclaredConstructor();
declaredConstructor.setAccessible(true);
Settings settings1 = declaredConstructor.newInstance();
System.out.println(settings == settings1);
```



##### 직렬화 & 역직렬화를 이용해

```java
Settings settings = Settings.getInstance();
Settings settings1 = null;
try (ObjectOutput out = new ObjectOutputStream(new FileOutputStream("settings.obj"))) {
    out.writeObject(settings);
}
try (ObjectInput in = new ObjectInputStream(new FileInputStream("settings.obj"))) {
    settings1 = (Settings) in.readObject();
}
System.out.println(settings == settings1);
```

역직렬화에 대한 싱글톤 깨지는것 대비하는 방법은 싱글톤 클래스 작성시

```java
protected Object readResolve() {
  return getInstance(); // 인스턴스 반환 메소드
}
```

를 사용하면 된다.

 역직렬화를 하면 ```readResolve()```을 읽도록 함



### 방법6. (권장)enum 

리플렉션에 안전함.

(내부적으로 serializable을 구현하고 있음) 역직렬화에도 안정.

```java
public enum Settings { 
	INSTANCE;
}
```

#### 문제점

클래스 로딩시 인스턴스 미리 생성된다.

상속 사용할 수 없다.



**NOTE**

1. enum 타입의 인스턴스를 리팩토링을 만들 수 있는가?
2. enum으로 싱글톤 타입을 구현할 때의 단점은?
3. 직렬화 & 역직렬화 시에 별도로 구현해야 하는 메소드가 있는가?

---

참고

[1]:https://siyoon210.tistory.com/141	"내부 정적 클래스"


