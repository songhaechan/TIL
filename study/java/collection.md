## Collection

### List

순서 o, 중복 o

구현체 : **ArrayList**, LinkedList(Deque의 구현체이기도 하다), Stack, Vector

### Vector와 Stack 왜 쓰면 안될까?

Vector는 Collection 프레임워크가 나오기 이전의 구현체로 최적화가 이루어지지않았다.

synchronized 키워드가 곳곳에 붙어있어 멀티스레드환경에서 thread-safe하긴하지만 성능이 많이 저하된다.

```java
//get
public synchronized E get(int index) {
        if (index >= elementCount)
            throw new ArrayIndexOutOfBoundsException(index);

        return elementData(index);
    }

//add
public synchronized boolean add(E e) {
        modCount++;
        add(e, elementData, elementCount);
        return true;
    }

//remove
public synchronized boolean removeElement(Object obj) {
        modCount++;
        int i = indexOf(obj);
        if (i >= 0) {
            removeElementAt(i);
            return true;
        }
        return false;
    }

// 이 외에도 거의 모든 메서드가 synchronized 처리돼있음
```

그런데 아이러니하게도 메서드는 동기화처리돼있지만 객체 자체는 동기화처리가 돼있지않아서 동시성 문제는 여전히 있다.

성능저하 + 동시성 이슈 라면 절대 사용하지말자.

**Vector를 상속받는 Stack 또한 같은 맥락이다.**

> 스택을 사용하고싶다면 Deque의 구현체 ArrayDeque를 사용해라 - Java api 공식문서

### ArrayList를 사용하자

ArrayList는 동기화처리가 돼있지않아서 thread-safe하지는 않다.

그래서 멀티스레드 환경에서는 동기화처리를 한 ArrayList를 사용해야한다.

```java
List<String> l1 = Collections.synchronizedList(new ArrayList<>());
```

> 실무에서 이런 방식으로 많이 사용한다고한다.

> 추가로 동기화 처리된 ArrayList는 객체자체를 동기화처리한다.

결론

싱글스레드환경 -> ArrayList 사용 (Vector보다 2배 빠름)

멀티스레드환경 -> 동기화 처리된 ArrayList 사용

### Set

순서 x , 중복 x

구현체 : HashSet, TreeSet

HashSet은 내부에 해시 알고리즘 적용

TreeSet 완전이진트리(레드-블랙트리)로 검색에서 성능이 좋음

### Map

순서 x, 키는 중복 x 값은 중복 x

구현체 : HashTable, HashMap, ConcurrentHashMap

HashTable는 위 Vector와 마찬가지로 곳곳에 syncronized키워드 적용돼있음(thread-safe)

HashMap은 동기화처리가 전혀 돼있지않음 (not thread-safe)

멀티스레드환경에 사용하기 적합한 **ConcurrentHashMap**을 사용하면 됨

> 실무에서 많이 사용된다고한다.

### ConcurrentHashMap은 어떻게 구현돼있을까

```java
public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                return (p = e.find(h, key)) != null ? p.val : null;
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

get() 메서드, 조회 메서드는 동기화처리가 돼있지않다.

하지만

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh; K fk; V fv;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                    break;
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else if (onlyIfAbsent
                     && fh == hash
                     && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                     && (fv = f.val) != null)
                return fv;
            else {
                V oldVal = null;

                // 여기  ~~
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
```

값을 넣는 메서드는 **일부 동기화처리**가 돼있다.

HashCode가 같은 버킷에 값을 넣을땐 동기화처리를 하고 다른 버킷이라면 전혀 동기화처리를 하지않는다.

## HashMap , ConcurrentHashMap 동시성 테스트

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class Main {
    public static void main(String[] args) {
        ConcurrentHashMap<String,String> chm = new ConcurrentHashMap<>();
        Map<String,String> hm = new HashMap<>();

        new Thread(()->{
            System.out.println("Thread 1 시작");
            for(int i=0; i<10000; i++){
                String key = String.valueOf(i);
                chm.put(key,"some value");
                hm.put(key,"some value");
            }
        }).start();

        new Thread(()->{
            System.out.println("Thread 2 시작");
            for(int i=0; i<10000; i++){
                String key = String.valueOf(i);
                chm.put(key,"some value");
                hm.put(key,"some value");
            }
        }).start();

        new Thread(() -> {
            try {
                Thread.sleep(2000);  // 쓰레드가 다 돌때까지 2초 대기

                System.out.println("ConcurrentHashMap 의 추가된 요소 갯수 size : " + chm.size());
                System.out.println("HashMap 의 추가된 요소 갯수 size : " + hm.size());
            } catch (InterruptedException ignored) {}
        }).start();
    }
}

//결과

Thread 1 시작
Thread 2 시작
ConcurrentHashMap 의 추가된 요소 갯수 size : 10000
HashMap 의 추가된 요소 갯수 size : 10286

```

동시성 문제로 데이터가 겹쳐서 출력되는걸 볼 수 있다.

Thread 1 이 key = 1에대한 중복검사를 통과하고 다시 대기열로 들어가고 Thread2가 같은 키로 중복검사를 하게되면 통과된다.

Thread1이 아직 데이터를 삽입하지 않았기 때문이다.

결국 데이터는 중복저장된다.


반면 ConcurrentHashMap은 동기화처리로 예측된 값을 출력하고있다.

> 조회시에도 당연히 문제는 발생한다. HashMap이 resize()메서드를 호출하는 중간에 get()을 호출하면 null이 나올 가능성이 있다.

> 이 테스트는 Vector와 ArrayList도 동일하다. 이 테스트와는 반대로 ArrayList는 3만건보다 몇 백건 적은 데이터를 가진다.
> add()가 동시에 호출되면 두 개의 스레드가 삽입할 인덱스를 같은 인덱스로 계산하게된다. 이런 문제때문에 같은 인덱스에 값을 덮어 씌워버릴 수 있다.

### HashMap ConcurrentHashMap 성능 측정

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class Main {
    public static void main(String[] args) {
        ConcurrentHashMap<String, String> chm = new ConcurrentHashMap<>();
        Map<String, String> hm = new HashMap<>();
        long startTime1 = System.currentTimeMillis();
        for (int i = 0; i < 30000; i++) {
            String key = String.valueOf(i);
            chm.put(key, "some value");
        }
        long estimatedTime1 = System.currentTimeMillis() - startTime1;
        System.out.println("ConcurrentHashMap 10000개 put = " + estimatedTime1 / 1000.0);

        long startTime = System.currentTimeMillis();
        for (int i = 0; i < 30000; i++) {
            String key = String.valueOf(i);
            hm.put(key, "some value");
        }
        long estimatedTime = System.currentTimeMillis() - startTime;
        System.out.println("HashMap 10000개 put = " + estimatedTime / 1000.0);

    }
}
//결과
ConcurrentHashMap 30000개 put = 0.016
HashMap 30000개 put = 0.002
```

3만개 데이터 기준 약 8배정도 차이가 났다.

> Vector와 ArrayList의 성능차이도 2배정도 발생한다.

## 람다식 vs 일반형

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        List<Number> numbers = new ArrayList<>(
                List.of(
                        new Number(5),
                        new Number(2),
                        new Number(12),
                        new Number(22),
                        new Number(9),
                        new Number(17),
                        new Number(6)
                )
        );
        
        Collections.sort(numbers, new NumberComparator());
        
//        위 로직의 람다식
//        Collections.sort(numbers, (num1,num2)-> num2.getValue() - num1.getValue());

        Iterator<Number> it = numbers.iterator();
        while(it.hasNext()){
            Number num = it.next();
            System.out.println(num);
        }
        
//        위 로직의 람다식
//        numbers.forEach(num -> System.out.println(num.getValue()));
    }
}

class NumberComparator implements Comparator<Number> {
    @Override
    public int compare(Number number1, Number number2) {
        return number1.getValue() - number2.getValue();
    }
}

class Number{
    int value;

    public Number(int value) {
        this.value = value;
    }

    public int getValue() {
        return value;
    }
}

```
