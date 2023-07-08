Exception Performance Cost
=

예외가 성능에 미치는 영향을 정리한 글이다.

https://www.baeldung.com/java-exceptions-performance


## 결론부터

> Since throwing and handling exceptions is expensive, we shouldn't use it for normal program flows. Instead, as its name implies, exceptions should only be used for exceptional cases.

예외를 new로 생성하고 catch로 잡아서 trace를 추적하는 것은 비용이 매우 비싸다.

예외는 정말 그 의미 자체로 예외적인 상황에서만 사용해야한다.

## 얼마나 차이날까?

    ExceptionBenchmark.createExceptionWithoutThrowingIt       avgt   10   16.605 ± 0.988  ms/op
    ExceptionBenchmark.doNotThrowException                    avgt   10    0.047 ± 0.006  ms/op
    ExceptionBenchmark.throwAndCatchException                 avgt   10   16.449 ± 0.304  ms/op
    ExceptionBenchmark.throwExceptionAndUnwindStackTrace      avgt   10  326.560 ± 4.991  ms/op
    ExceptionBenchmark.throwExceptionWithoutAddingStackTrace  avgt   10    1.185 ± 0.015  ms/op

1. 예외를 new로 생성만하고 던진 메서드

2. 예외에대해 전혀 고려하지않은 메서드 (비교대상)

3. 예외를 던지고 잡은 메서드

4. 예외를 잡긴했지만 trace를 추적한 메서드

5. 예외를 던지고 잡고 trace는 추적하지않은 메서드

4번 메서드가 비용이 가장 비싸다. 약 10000배 비싸다.

## 잘 생각해서 사용하자.

비싸다는 사실을 인지했다면 예외를 던질 때 한 번쯤은 생각하게될 것같다. 

이 경우가 정말 예외적인지 그렇지 않은지를

