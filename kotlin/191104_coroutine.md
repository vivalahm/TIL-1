# 코틀린의 코루틴 

Kotlin은 언어로, 표준 라이브러리에서 최소한의 저수준 API을 제공 하여 다양한 다른 라이브러리에서 코루틴을 활용할 수 있도록 도와준다. 

`kotlinx.coroutines`는 JetBrains에서 개발한 코루틴을 위해 제공 되는 라이브러리이다. 여기에는 `launch`, `async`등을 포함하여 이 가이드에서 다루는 여러 고 수준의 코루틴 지원 요소들이 포함 되어 있다. 

이 가이드는 `kotlinx.coroutines`의 핵심 기능에 대한 가이드 이며 여러 주제로 나누어진 예제들과 함께 제공 된다. 코루틴을 사용 하고 싶다면 코루틴에 대한 의존을 추가 해야 한다. 예를 들어 안드로이드의 경우 `build.gradle`에 아래와 같이 추가 한다. 

```gradle
dependencies {
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.4.2'
}
```

- 공부 한 내용은 [Kotlin 공식 가이드 문서](https://kotlinlang.org/docs/reference/coroutines/basics.html)을 바탕으로 하였다. 

## 1. Asynchronous programming techniques

아래의 코드를 실행 하면 다음과 같은 출력 결과를 확인 할 수 있다. 

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch {    // 새로운 코루틴을 백그라운드 스레드로 실행 한다. 
        delay(1000L)        // non-blocking 방식으로 1초 동안 딜레이 시킨다. 
        println("World.")   // 딜레이 후 글자를 출력 한다. 
    }
    println("Hello,")       // 메인 스레드에서 코루틴 이 딜레이 걸린 동안(non-blocking) 출력 된다
    Thread.sleep(2000L)     // 메인 스레드 를 2초동안 딜레이 시켜준다. 
}

// 출력 결과 
Hello,
World.
```

본질적으로 코루틴은 경량화 된 스레드(light-weight thread) 이다. `CoroutineScope` 내 `launch` 코루틴 빌더를 통한 블록 내 코드를 `GlobalScope` 로 실행 시킨 것 이다. `GlobalScope` 의 lifetime 은 application 의 lifetime 에 의존하기 때문이다. 

> 안드로이드 에서는 Acvitiy 가 종료 되도 application 은 아직 살아 있기 때문에 코루틴 코드는 계속 실행 될 수 있다. 하지만 만약 application 이 강제 종료 되거나 하면 코루틴 의 블럭 내 코드도 종료 되어버릴 수 있다. 

위 코드에서 `GlobalScope.launch { ... }` 와 `thread { ... }`, 그리고 `delay { ... }` 와 `Thread.sleep(...)` 을 서로 바꾸어도 동일한 결과를 얻을 수 있다. 

만약 `GlobalScope.launch` 를 `thread` 로 변경 하면 컴파일러는 다음과 같은 에러를 알려준다. 

```
Error: Kotlin: Suspend functions are only allowed to be called from a coroutine or another suspend function
```

컴파일 오류가 발생 하는 이유는 `delay` 때문이다. 이 함수는 `suspending function` 으로서 non-blocking thread 이다. 

