## 동작 파라미터화 코드

### 우리가 엔지니어링할 소프트웨어는 항상 변화한다.

### 소프트웨어의 변화는 사용자의 변화하는 요구사항이고 손쉽게 반영해야한다.

### 요구사항이란 기존 요구사항의 변화 혹은 그 양의 증가라 할 수 있다.

### 사용자의 요구사항은 예측할 수 없지만, **대비**할 수는 있다.

### 그때 효과적으로 사용할 수 있는 것이 동작 파라미터화(behavior parameterization)

## 상황을 가정해보자

우리는 학생정보 시스템을 만들고 있다.

학생정보 시스템의 사용자는 학생, 교수 혹은 직원이 될 수 있고 이들의 요구사항에 맞춰 소프트웨어를 설계하고 확장가능하며 유지보수가 쉽도록 해야한다.

```java
class Student{
    // 이름
    private String name;
    // 학번
    private int id;
    // 학점
    private int gpa;
    // 전공
    private String major;
}
```

간단한 학생 엔티티를 생성했다.

## 첫 시도 : 컴퓨터공학 전공 필터링

```java
public class Main {
    public static void main(String[] args) {
        Student s1 = new Student("song",2020158020,3.3,"cs");
        Student s2 = new Student("park",2020158230,3.1,"cs");
        Student s3 = new Student("kim",2018203029,4.3,"me");
        Student s4 = new Student("lee",2019203029,2.3,"me");
        Student s5 = new Student("choi",2015203029,3.9,"ee");

        List<Student> students = new ArrayList<>(Arrays.asList(s1,s2,s3,s4,s5));
        // 호출
        filterComputerScience(students);
    }
    public static List<Student> filterComputerScience(List<Student> students){
        List<Student> result = new ArrayList<>();
        // 컴퓨터공학 필터링
        for(Student student:students){
            if(student.getMajor().equals("cs")){
                result.add(student);
            }
        }
        return result;
    }
}
```

가장 쉽게 생각할 수 있는 방법이다.

메서드를 정의하고 적절한 구현을 통해 서비스를 제공할 수 있다.

### 그런데 요구사항이 컴공이아닌 전자공학이라면 메서드가 하나 더 늘어나야한다.

### 하지만 세상에 전공이 컴공 전자공학 둘 뿐이진 않다.

### 전공이 1000개면 메서드도 1000개 생성된다.

### 1000개 중에서 한 전공의 이름이 바뀌면? 1000개를 다 뒤져야한다.

### 즉 요구사항의 변화에 유연하게 대처하지도 못하고 유지보수에도 좋지 않다.

## 두 번째 시도 : 전공을 파라미터화하자

filterComputerScience 와 같이 하나의 메서드를 하나의 전공 찾기로 구속하지말고 전공은 파라미터로 받아보자.

꽤 괜찮은 방법이다.

```java
// Main에서 호출
filterStudentByMajor(students,"cs");

public static List<Student> filterStudentByMajor(List<Student> students, String major){
        List<Student> result = new ArrayList<>();
        // 컴퓨터공학 필터링
        for(Student student:students){
            if(student.getMajor().equals(major)){
                result.add(student);
            }
        }
        return result;
    }
```

이제 전공이 1000개여도 메서드는 1000개 생성되지않아도 된다.

**동작**을 파라미터화 한 것은 아니지만, 코드의 중복을 해결했다.

그런데 갑자기 교수님께서 학점이 4.0이상인 학생까지 알고싶다고 하신다...

## 세 번째 시도 : 모든 조건을 필터링하자

전공만 조회하는 메서드, 학점만 조회하는 메서드를 따로 만들기엔 또 다시 중복된다.

OK 그럼 또 파라미터화를 해보자.

하지만 그러기 위해서는 기준이 필요하다.

파라미터로 0을 넘기면 전공 조회, 1을 넘기면 학점조회로 구현해보자.

```java
// Main에서 호출
// 학점조회
filterStudentByMajor(students,null,4.0,1);
// 전공조회
filterStudentByMajor(students,"ce",0,0);

public static List<Student> filterStudentByMajor(List<Student> students, String major, int gpa, int flag){
        List<Student> result = new ArrayList<>();
        // 컴퓨터공학 필터링
        for(Student student:students){
            if((student.getMajor().equals(major) && flag == 0)||(flag==1 && (student.getGpa() > gpa))){
                result.add(student);
            }
        }
        return result;
    }
```

항상 코드는 **이해하기 쉬워야 유지보수가 쉽다.**

하지만 딱봐도 코드는 복잡해졌다.

호출 부는 더 복잡하다.

```java
// 학점조회
filterStudentByMajor(students,null,4.0,1);
// 전공조회
filterStudentByMajor(students,"ce",0,0);
```

여기서 도대체 null과 0,1들이 무엇을 의미하는지 직접 메서드에 가서 이해하거나, 혹은 해당 메서드를 작성한 사람에게 물어보거나, 해당 메서드의 문서를 찾아야한다.

주석을 쓰는 습관은 매우 좋지만, 정말 이상적으로 주석을 보지않아도 이해되는 코드를 작성하려 노력해야한다.

## 네 번째 시도 : 동작 파라미터화

### 이제 객체지향의 핵심을 적용해보자.

### 객체지향 다형성의 핵심은 변화하는 부분을 추상화 시키고 캡슐화시킨다.

### 그렇다면 변화하는 부분은 어디고 변화하지 않는 부분은 어디인가?

### 자세히 보면 for문 안쪽만 변화하고 for문 바깥쪽은 변화하지 않는다.

### 저 부분을 다형성을 이용해 추상화시키자.

```java
interface StudentFilter{
    boolean filter(Student student);
}
```

변화하는 부분을 메서드화 시킬 준비를 했다.

```java
class StudentMajorFilter implements StudentFilter{
    @Override
    public boolean filter(Student student) {
        return student.getMajor().equals("ce");
    }
}
```

변화 부분을 추출해서 추상화 시키고 내부 구현은 캡슐화하여 숨겼다.

```java
// 호출
StudentFilter sf = new StudentMajorFilter()
filterStudentByMajor(students,sf);

public static List<Student> filterStudentByMajor(List<Student> students, StudentFilter studentFilter){
        List<Student> result = new ArrayList<>();
        for(Student student:students){
            if(studentFilter.filter(student)){
                result.add(student);
            }
        }
        return result;
    }
```

이제 호출 부는 세부적인 구현은 알지 못하지만 파라미터로 객체를 받아 호출부가 모든 결정을 할 수 있도록 유연하게 변경됐다.

### 그런데 학점을 조회하기위해선 인터페이스를 구현하는 클래스를 또 만들어야하고, 이름이나 학번 등 여러 조건이 추가되면 클래스를 그만큼 또 만들어야한다.

### 모던 자바는 그러기가 싫다.

## 다섯 번째 시도 : 익명클래스를 사용해서 따로 클래스를 증가시키지 않을 수 있다.

### 익명 클래스는 선언과 구현을 동시에하고 단 한번만 사용하는 일회용 클래스다.

익명 클래스를 사용하면 호출부에서 사용이 가능하다.

```java
// 호출
filterStudentByMajor(students,new StudentFilter(){
    public boolean filter(Student student){
        return student.getMajor().equals("ce");
    }
})
```

### 이젠 클래스도 증가하지 않는다.

### 동작 파라미터화란 이렇게 특정 동작(메서드) 자체를 파라미터로 넘기는 것을 뜻한다.

### 동작을 파라미터화하면 리스트를 순회하는 일과 리스트의 엘리먼트들에 적용할 동작을 분리시켜 메서드를 더욱더 유연하게 만든다.

### 하지만 모던 자바는 멈추지 않는다.

### 더 가볍고 더 간단하게

## 여섯 번째 시도 : 람다를 사용하자

```java
// 호출
filterStudentByMajor(students,new StudentFilter(){
    public boolean filter(Student student){
        return student.getMajor().equals("ce");
    }
})
```

바로 위 코드는 아래와 같이 사용 가능하다.

```java
// 호출
filterStudentByMajor(students, (student) -> student.getMajor().equals("ce"))
```

### 아름답다.

## 일곱 번째 시도 : 제네릭을 이용해 외계인도 조회하자

```java
public static List<Student> filterStudentByMajor(List<Student> students, StudentFilter studentFilter){
        List<Student> result = new ArrayList<>();
        for(Student student:students){
            if(studentFilter.filter(student)){
                result.add(student);
            }
        }
        return result;
    }
```

### 이 코드를 아래와 같이

```java
public static List<T> filterStudentByMajor(List<T> something, StudentFilter<T> studentFilter){
        List<T> result = new ArrayList<>();
        for(T something:Somethings){
            if(studentFilter.filter(student)){
                result.add(student);
            }
        }
        return result;
    }
```

### 더 아름답다.
