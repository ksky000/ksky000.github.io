---
title:  "Google Promises - 2"
excerpt: "Google에서 개발하여 오픈한 Promises Framework에 대해 알아봅시다."

categories:
  - Swift
tags:
  - Swift
  - Promise Pattern
  - Google Promises
last_modified_at: 2020-06-14T020:22:00
---

### Promises 소개

---

본 포스트는 Google Promises 프레임워크에 대해 설명하는 포스트이며 해당 프레임워크의 공식 사이트는 다음과 같습니다.

<https://github.com/google/promises>

그리고, 본 포스트는 Goolge Promises 프레임워크의 확장 오퍼레이터에 대한 설명을 담고 있으며 Google Promise의 기본 개념에 대해 알고 있어야 이해가 가능합니다.

기본 개념에 대한 설명은 다음 링크를 참조하세요.

<https://ksky000.github.io/swift/promises1/>

### Promise의 확장 오퍼레이터(Extensions)

---

Google Promise 클래스는 기본 오퍼레이터인 async, do, then, catch 외에 다양한 메서드를 제공하고 있습니다.

이중 인스턴스 메서드(Instance method)들은 Promise의 상태가 fulfilled, rejected로 변경됐을 때에 대한 처리를 담당하는 메서드들이고 클래스 메서드(Class Method)는 Promise의 객체 인스턴스를 생성 후 어떠한 조건에 의해 Promise의 상태가 fulfilled, rejected 상태로 변경될지에 대한 부분을 정의하도록 합니다.

다음은 Promises Framework에서 제공하는 확장 오퍼레이터 들입니다.

* 클래스 메서드(Class Methods)
    - all
    - any
    - race
    - await
    - retry
    - wrap
* 인스턴스 메서드(Instance Method)
    - always
    - delay
    - recover
    - reduce
    - timeout
    - validate

종류가 매우 다양하지요? 이제 위 오퍼레이터 들에 대해 자세히 알아보겠습니다.

### 클래스 메서드(Class Methods)

---

Promise 내에서 클래스 메서드는 대부분 Promise의 객체 인스턴스를 생성하고 생성된 Promise가 어떠한 조건에 의해 fulfilled 또는 rejected 상태로 변경될지를 정의합니다.

#### All

all 클래스 메서드는 파라미터로 Promise 객체 인스턴스로 이루어진 Array를 받습니다.

다음 예제를 봅시다.

```swift
enum SomeError : Error {
    case invalidNumber
}

// 1. startNum부터 endNum까지의 합을 구하여 반환하는 메서드
func sum(startNum: Int, endNum: Int) throws -> Int {
    guard startNum >= 0 && endNum >= 0 else {
        throw SomeError.invalidNumber
    }

    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

// 2. Promise의 객체 인스턴스를 생성하여 1부터 10000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise1 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 1, endNum: 10_000_000)
}

// 3. Promise의 객체 인스턴스를 생성하여 10000001부터 20000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise2 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 10_000_001, endNum: 20_000_000)
}

// 4. Promise의 객체 인스턴스를 생성하여 20000001부터 30000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise3 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 20_000_001, endNum: 30_000_000)
}

// 5. promise1, promise2, promise3이 모두 fulfilled 상태가 되면 fulfilled 상태가 되는 Promise 객체 인스턴스 생성
let allPromise = all([promise1, promise2, promise3])

// 6. allPromise의 then 오퍼레이터에 전체 결과값을 더하여 출력하는 클로저 전달
allPromise.then { results in
    print("called then")
    print(results.reduce(0, +))
}

// 6. allPromise의 catch 오퍼레이터에 error를 출력하는 클로저 전달
allPromise.catch { error in
    print("called catch")
    print(error)
}
```

위 코드는 1에서 3000만까지의 합을 구하여 출력하는 코드입니다.

1에서 3000만까지 루프를 돌면서 구하고자 하는데 3000만번의 루프를 다 수행하려면 시간이 많이 소요되기 때문에 1에서 1000만, 1000만1에서 2000만, 2000만1에서 3000만까지 각각 나눠서 더하도록 하였습니다.

그리고, 메인스레드에서 이 작업을 수행하면 UI가 멈추는 현상이 발생할 수 있으므로 글로벌 스레드에서 각각의 작업을 수행했습니다.

물론 메인 스레드 큐는 시리얼 큐이므로 3개의 작업이 동시에 진행되지 않기 때문에 만약 메인 스레드에서 수행하고자 하면 3개의 작업으로 나눠서 하는 것이 의미가 없습니다.

위에 코드 6번 주석 아래에 Promise 객체의 all 클래스 메서드를 이용하여 새로운 Promise 객체 인스턴스를 생성하였습니다.

all 메서드는 파라미터로 전달받은 Array에 포함된 promise들이 모두 fulfilled 상태가 되면 fulfilled 상태로 상태가 변경되는 Promise 객체 인스턴스를 생성하여 반환합니다.

즉, 위 코드에서는 promise1, promise2, promise3가 모두 fulfilled 상태가 되면 allPromise도 fulfilled 상태가 되는 것이죠.

위 코드의 결과는 다음과 같습니다.

```text
called then
450000015000000
```

모든 Promise의 클로저들이 정상적으로 수행되어 결과적으로 allPromise의 then 오퍼레이터의 클로저가 실행되었죠?

자, 그럼 위 코드를 살짝만 수정해 볼까요? 위 코드에서 3번 주석 아래에 있는 Promise 생성 구문을 다음과 같이 수정해 봅시다.

```swift
// 3. Promise의 객체 인스턴스를 생성하여 -1부터 20000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise2 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 20_000_000)
}
```

클로저 내에서 sum 메서드를 호출할 때 파라미터 중 startNum을 -1로 변경하였습니다.

그렇게 하면 sum 함수 내에서 예외처리에 의해 에러가 던져질 것입니다. 그러면 최종적으로 promise1과 promise3는 fulfilled 상태가 되고 promise2는 rejected 상태가 되겠죠.

그렇게 해서 실행해 보니 다음과 같은 결과가 나왔습니다.

```text
called catch
invalidNumber
```

예상하셨듯이 all 오퍼레이터에 전달된 3개의 Promise(promise1, promise2, promise3) 중 proimse2가 rejected 상태가 되어 그 결과로, allPromise도 rejected 상태가 되었습니다. 따라서, catch 오퍼레이터에 전달된 클로저가 호출되었습니다.

자, all 오퍼레이터에 대해서 이해가 잘 되시나요?

그럼 위 코드를 짧게 축약해 볼까요?

```swift
enum SomeError : Error {
    case invalidNumber
}

func sum(startNum: Int, endNum: Int) throws -> Int {
    guard startNum >= 0 && endNum >= 0 else {
        throw SomeError.invalidNumber
    }

    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

all([
    Promise<Int>(on: .global()) { () -> Int in
        return try sum(startNum: 1, endNum: 10_000_000)
    },
    Promise<Int>(on: .global()) { () -> Int in
        return try sum(startNum: 10_000_001, endNum: 20_000_000)
    },
    Promise<Int>(on: .global()) { () -> Int in
        return try sum(startNum: 20_000_001, endNum: 30_000_000)
    }
]).then { results in
    print(results.reduce(0, +))
}.catch { error in
    print("called catch")
    print(error)
}
```

#### Any

any는 all과 비슷한 오퍼레이터입니다. 파라미터로 Promise 객체 인스턴스의 배열을 전달합니다. 그렇다면 다른점은 무엇일까요?

all을 이용하여 생성한 Promise 파라미터로 전달한 모든 Promise가 fulfilled 상태가 되어야만 fulfilled 상태가 됩니다.

하지만, any는 하나만이라도 fulfilled 상태가 되면 fulfilled 상태가 됩니다.

위에 예제로 사용한 코드를 다시 가져와서 약간 수정해 보겠습니다.

```swift
enum SomeError : Error {
    case invalidNumber
}

// 1. startNum부터 endNum까지의 합을 구하여 반환하는 메서드
func sum(startNum: Int, endNum: Int) throws -> Int {
    guard startNum >= 0 && endNum >= 0 else {
        throw SomeError.invalidNumber
    }

    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

// 2. Promise의 객체 인스턴스를 생성하여 1부터 10000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise1 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 1, endNum: 10_000_000)
}

// 3. Promise의 객체 인스턴스를 생성하여 10000001부터 20000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise2 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 10_000_001, endNum: 20_000_000)
}

// 4. Promise의 객체 인스턴스를 생성하여 20000001부터 30000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise3 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 20_000_001, endNum: 30_000_000)
}

// 5. promise1, promise2, promise3이 하나 이상의 Promise가 fulfilled 상태가 되면 fulfilled 상태가 되는 Promise 객체 인스턴스 생성
let anyPromise = any([promise1, promise2, promise3])

// 6. allPromise의 then 오퍼레이터에 전체 결과값을 출력하는 클로저 전달
anyPromise.then { results in
    print("called then")
    for result in results {
        if let value = result.value {
            print(value)
        } else if let error = result.error {
            print(error)
        }
    }
}

// 6. allPromise의 catch 오퍼레이터에 error를 출력하는 클로저 전달
anyPromise.catch { error in
    print("called catch")
    print(error)
}
```

위 코드를 실행하면 promise1, promise2, promise3이 모두 fulfilled 상태가 되고 최종 anyPromise도 fulfilled 상태가 되어 아래와 같이 출력될 것입니다.

```text
called then
50000005000000
150000005000000
250000005000000
```

then 오퍼레이터에 전달하는 클로저의 파라미터는 Array 타입이며 Maybe라는 enum 타입의 값들을 가지고 있습니다.

Maybe는 .value와 .error 중 하나의 값을 가지고 있습니다.

이제 하나의 promise1, promise2, promise3 중 하나의 proimse를 rejected 상태로 만들어 보겠습니다.

위 코드에서 3번 주석 아래의 Promise 객체 인스턴스 생성 코드를 아래와 같이 수정해 보겠습니다.

```swift
// 3. Promise의 객체 인스턴스를 생성하여 -1부터 20000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise2 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 20_000_000)
}
```

클로저 내 sum 메서드에 전달하는 파라미터 중 startNum 파라미터의 값을 -1로 수정하여 sum 메서드 내에서 에러를 던지도록 수정하였습니다.

자 결과는 어떻게 될까요?

```text
called then
50000005000000
invalidNumber
250000005000000
```

promise2의 상태가 rejected가 되었지만 그 외의 Promise 들의 상태가 fulfilled 이므로 최종 anyPromise의 then 오퍼레티어에 전달한 클로저가 수행되었습니다.

이제 다음과 같이 모든 Promise를 rejected 상태로 만들어 볼까요?

```swift
// 2. Promise의 객체 인스턴스를 생성하여 1부터 10000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise1 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 10_000_000)
}

// 3. Promise의 객체 인스턴스를 생성하여 10000001부터 20000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise2 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 20_000_000)
}

// 4. Promise의 객체 인스턴스를 생성하여 20000001부터 30000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise3 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 30_000_000)
}
```

모든 Promise 생성 구문의 클로저 내에서 sum 메서드를 호출 시 파라미터 startNum의 값을 -1로 전달하여 세개의 Promise가 모두 rejected 상태가 되도록 하였습니다.

그 결과는 예상하셨듯이 아래와 같습니다.

```text
called catch
invalidNumber
```

anyPromise에 전달된 모든 Promise가 rejected 상태가 되어 anyPromise도 rejected 상태가 되었고 최종적으로 anyPromise의 catch 오퍼레이터에 전달된 클로저가 수행 되었습니다.

자 이제 all과 any의 차이를 잘 아시겠죠?

#### Race

race 오퍼레이터는 all, any와 비슷한 또 하나의 오퍼레이터입니다.

all과 any에 의해 생성된 Promise는 파라미터로 전달된 모든 promise의 작업이 끝나야만 상태가 변경됩니다.

race는 그 중, 첫번째로 작업이 끝나는 proimse의 상태와 동일한 상태가 됩니다.

```swift
enum SomeError : Error {
    case invalidNumber
}

func sum(startNum: Int, endNum: Int) throws -> Int {
    guard startNum >= 0 && endNum >= 0 else {
        throw SomeError.invalidNumber
    }

    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

race([
    Promise<Int>(on: .global()) { () -> Int in
        return try sum(startNum: 1, endNum: 10_000_000)
    },
    Promise<Int>(on: .global()) { () -> Int in
        return try sum(startNum: 10_000_001, endNum: 20_000_000)
    },
    Promise<Int>(on: .global()) { () -> Int in
        return try sum(startNum: 20_000_001, endNum: 30_000_000)
    }
]).then { result in
    print("called then")
    print(result)
}.catch { error in
    print("called catch")
    print(error)
}
```

위 코드에서는 all, any 오퍼레이터 대신 race 오퍼레이터를 이용하여 Promise 객체 인스턴스를 생성했습니다.

위 결과는 어떻게 나올까요?

```text
called then
250000005000000
```

제가 테스트 했을 때는 위와 같이 나왔습니다. 하지만, 매번 위와 같이 결과가 나오지 않습니다. 3개의 Promise내 작업이 동시에 진행되고 가장 먼저 끝난 작업의 결과 값을 출력하게 될 것이므로 출력되는 결과는 일정하지 않을 것입니다.

all, any의 케이스와 동일하게 promise 중 하나를 rejected 상태로 만들어 봅시다.

```swift
Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 20_000_000)
}
```

fulfilled로 변경되는 것은 덧셈을 1000만번 하기 때문에 느리지만 rejected로 변경되는 것은 예외처리에 의해 금방 끝납니다. 따라서 모든 Promise내 작업 중 rejected로 변경되는 Promise의 작업이 가장 먼저 끝나게 되어 다음과 같은 결과가 나오게 됩니다.

```text
called catch
invalidNumber
```

#### Await

await는 다른 언어의 async, await와 유사한 기능을 하는 오퍼레이터입니다.

await를 사용하면 비동기 코드를 동기 코드처럼 보여지게 코드를 구성할 수 있어 매우 편리한 기능입니다.

다시 1부터 3000만까지 더해 볼까요?

```swift
enum SomeError : Error {
    case invalidNumber
}

// 1. startNum부터 endNum까지의 합을 구하여 반환하는 메서드
func sum(startNum: Int, endNum: Int) throws -> Int {
    guard startNum >= 0 && endNum >= 0 else {
        throw SomeError.invalidNumber
    }

    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

// 2. Promise의 객체 인스턴스를 생성하여 1부터 10000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise1 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 1, endNum: 10_000_000)
}

// 3. Promise의 객체 인스턴스를 생성하여 10000001부터 20000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise2 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 10_000_001, endNum: 20_000_000)
}

// 4. Promise의 객체 인스턴스를 생성하여 20000001부터 30000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
let promise3 = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 20_000_001, endNum: 30_000_000)
}

// 5. 1부터 3000만까지를 더하여 출력
Promise<Int>(on: .global()) { () -> Int in
    let result1 = try await(promise1)
    let result2 = try await(promise2)
    let result3 = try await(promise3)
    return result1 + result2 + result3
}.then { result in
    print(result)
}.catch { error in
    print(error)
}
```

자 위 코드에서 5번 주석 아래를 보시면 await라는 오퍼레이터를 사용하고 있는 것이 보이시죠.

await는 파라미터로 전달한 Promise의 상태가 fulfiled 또는 rejected 상태가 되기 전까지 리턴하지 않고 있다가 상태가 변경되면 리턴합니다.

즉, 위 코드 중 다음의 구문을 살펴 보면

```swift
let result1 = try await(promise1)
```

위 코드는 await 오퍼레이터 proimse1 파라미터를 전달하며 호출하고 있습니다. promise1은 1부터 1000만까지는 더하는 작업이 끝나면 fulfilled로 바뀌겠죠?

proimse1이 await는 1부터 1000만까지 더하는 작업이 끝나기 전까지 리턴하지 않고 있다가 해당 작업이 끝나고 promise1의 상태가 fulfilled로 바뀌면 리턴합니다.

리턴값은 promise1에 저장된 값이며 위에서는 1부터 1000만까지 더한 결과값이겠죠.

위와 같이 await는 비동기로 동작하는 코드를 동기적으로 동작하는 것처럼 코드를 구성할 수 있게 해줍니다.

위의 5번 코드를 await를 쓰지 않고 코드를 작성하면 다음과 같이 될 것입니다.

```swift
// 5. 1부터 3000만까지를 더하여 출력
Promise<Int>(on: .global()) { () -> Promise<Int> in
    var total = 0
    return promise1.then { result -> Promise<Int> in
        total += result
        return promise2
    }.then { result -> Promise<Int> in
        total += result
        return promise3
    }.then { result -> Int in
        total += result
        return total
    }
}.then { result in
    print(result)
}.catch { error in
    print(error)
}
```

자 await를 쓴 것이 더 코드가 훨씬 깔끔하죠?

마지막으로 await 오퍼레이터를 쓰면 1부터 3000만까지 더할 때 1000만 단위로 나눠서 동시에 비동기 작업을 통해 더한 후 그 결과들을 다시 더하는 형태로 만드는 성능 향상의 이점을 볼 수는 없습니다.

하나의 Promise 내 작업이 끝나야 다음 작업이 호출되기 때문이죠.

#### wrap

wrap은 기존에 제공되고 있는 일반적인 콜백 패턴의 메서드를 Promise 패턴의 메서드로 변환해주도록 하기위해 제공되는 오퍼레이터입니다.

```swift
URLSession.shared.dataTask(with: URL(string: "https://www.google.com")!) { (data, urlReponse, error) in
    guard error == nil else {
        print(error.debugDescription)
        return
    }

    if let data = data {
        print(String(data: data, encoding: .utf8) ?? "")
    }
}.resume()
```

자 위 코드를 보시면 "https://www.google.com"에 http 요청을 하고 그에 대한 응답을 출력하는 코드입니다. 기존의 콜백 패턴으로 되어 있죠.

URLSession.shared.dataTask(_:_:) 메서드를 wrap 오퍼레이터를 이용하여 Promise 패턴으로 바꿔보겠습니다.

```swift
wrap { handler in
    URLSession.shared.dataTask(with: URL(string: "https://www.google.com")!, completionHandler: handler).resume()
}.then { result in
    print(result)
}.catch { error in
    print(error)
}
```

코드가 상당히 깔끔해졌죠? wrap 오퍼레이터가 호출되면 wrap 오퍼레이터는 클로저에 또 다른 콜백(위 코드에서는 handler)를 넘겨줍니다.

그것을 그대로 래핑하고자 하는 메서드의 콜백 파라미터로 전달해 주는 것만으로 래핑 작업이 완료됩니다.

#### Retry

retry는 오류가 발생했을 때 바로 catch 오퍼레이터의 클로저를 호출하지 않고 재시도를 할 수 있도록 하는 기능을 제공해주는 오퍼레이터입니다.

```swift
// 1. 파라미터로 전달받은 url 주소로 http 요청하는 Promise 반환
func fetch(_ url: URL) -> Promise<(Data?, URLResponse?)> {
    return wrap { URLSession.shared.dataTask(with: url, completionHandler: $0).resume() }
}

// 2. fetch에서 오류 발생 시 1초 후 재시도 진행. 재시도는 1번만 진행
let retryPromise = retry { () -> Promise<(Data?, URLResponse?)> in
    let fetchPromise = fetch(URL(string: "https://google.com")!)
    return fetchPromise
}

retryPromise.then {
    print($0)
}

retryPromise.catch {
    print($0)
}
```

위 코드를 보시면 fetch 메서드를 바로 호출하지 않고 retry 오퍼레이터 전달한 클로저 내에서 호출했습니다.

위와 같이 하면 fetch 메서드에서 반환한 fetchPromise가 rejected 상태가 되었을때 바로 오류 처리를 하지 않고 retry에 전달한 클로저를 한번 더 실행해 줍니다.

한번 더 실행한 후 또 rejected 상태가 되면 최종 catch 오퍼레이터의 클로저를 호출하고 재시도 시 성공하면 then 오퍼레이터의 클로저를 호출해 줍니다.

다음은 retry 오퍼레이터에 추가 옵션을 더 전달하여 재시도를 수정해 보겠습니다.

```swift
// 1. 파라미터로 전달받은 url 주소로 http 요청
func fetch(_ url: URL) -> Promise<(Data?, URLResponse?)> {
    return wrap { URLSession.shared.dataTask(with: url, completionHandler: $0).resume() }
}

let url = URL(string: "https://google.com")!

// 2. fetch에서 오류 발생 시 2초 후 재시도 진행. 재시도는 최대 5번까지 진행
let customQueue = DispatchQueue(label: "CustomQueue", qos: .userInitiated)
retry(
    on: customQueue,
    attempts: 5,
    delay: 2,
    condition: { remainingAttempts, error in
        (error as NSError).code == URLError.notConnectedToInternet.rawValue
}
) {
    fetch(url)
}.then { values in
    print(values)
}.catch { error in
    print(error)
}
```

위 코드에서 retry를 호출할 때 추가 파라미터를 더 전달했습니다. retry에 전달된 파라미터를 보실까요?

* on: 이미 알고 있드시 파라미터로 전달된 클로저를 실행할 큐입니다. 여기서는 main thread queue, global thread queue를 사용하지 않고 custom queue를 생성하였습니다.
* attempts: 재시도를 할 회수입니다. 여기서는 최대 5번까지 재시돌를 진행합니다.
* delay: 오류 발생 후 다시 재시도를 하기까지의 시간입니다. 여기서는 2초를 설정하였습니다.
* condition: 재시도를 할 조건입니다. true를 반환해야 재시도를 진행합니다. 여기서는 네트워크 연결이 되어 있지 않을 때 재시도를 하도록 하였습니다.

자 retry 이해가 되시나요?

이제 Promises의 클래스 메서드에 대한 설명을 마치고 다음은 인스턴스 메서드에 대해 설명하도록 하겠습니다.

### Instance Method

---

Google Promise의 인스턴스 메서드들은 Promise의 상태가 fulfilled, rejected로 변경되었을 때의 동작들을 정의합니다.

이전 포스트에서 설명했던 then, catch등이 그렇게 동작을 하죠?

#### Always

always 오퍼레이터는 Promise의 상태가 fulfilled, rejected로 변경됐을 때 그 상태에 관련없이 항상 호출됩니다.

다음 예제를 보시죠.

```swift
enum SomeError : Error {
    case invalidError
}

// 1. 파라미터 success에 true를 넘기면 fulfilled
// false를 넘기면 rejected로 변경되는 Promise 객체 인스턴스를 생성하여 반환
func request(success: Bool) -> Promise<Bool> {
    return Promise<Bool> { fulfill, reject in
        if success {
            // 2. Promise의 상태를 fulfilled로 변경
            fulfill(true)
        } else {
            // 3. Promise의 상태를 rejected로 변경
            reject(SomeError.invalidError)
        }
    }
}

// 4. request 메서드를 호출하여 생성된 Promise에
// then, catch, always 오퍼레이터 세팅
request(success: true).then { result in
    print("called then")
}.catch { error in
    print("called catch")
}.always {
    print("called always")
}
```

위의 코드에서 request 메서드는 파라미터로 전달된 success의 값이 true라면 fulfilled 상태가 되고 false라면 rejected 상태가 되는 Promise의 객체 인스턴스를 생성하여 반환합니다.

그리고, 4번 주석 아래의 코드를 보시면 request 메서드를 호출하면 서 success 파라미터에 true를 전달하고 있죠.

자 위 코드의 실행 결과는 어떻게 될까요?

```text
called then
called always
```

자 Promise의 상태가 fulfilled로 변경되었으니 당연히 then 오퍼레이터에 전달된 클로저가 호출되었고 always 오퍼레이터에 전달된 클로저도 호출되었습니다.

위 코드에서 4번 주석 아래 request 메서드를 호출하는 코드의 success 파라미터 값을 아래와 같이 false로 변경해 봅시다.

```swift
// 4. request 메서드를 호출하여 생성된 Promise에
// then, catch, always 오퍼레이터 세팅
request(success: false).then { result in
    print("called then")
}.catch { error in
    print("called catch")
}.always {
    print("called always")
}
```

결과는 어떻게 될까요?

```text
called catch
called always
```

Promise의 상태가 rejected로 변경되었으니 당연히 catch 오퍼레이터에 전달된 클로저가 호출되었고 always 오퍼레이터에 전달된 클로저도 호출되었습니다.

위의 예에서 알 수 있듯이 always 오퍼레이터는 Promise의 상태 변경 시 변경된 상태가 어떤 상태이든 무조건 호출되도록 하는 오퍼레이터입니다.

#### Delay

delay 오퍼레이터는 Promise 객체 인스턴스 상태가 fulfilled로 변경되면 delay의 파라미터로 전달된 시간 후에 then 오퍼레이터에 전달된 클로저를 실행 시키는 기능을 가지고 있습니다.

반대로, 상태가 rejected가 되면 delay 시간을 무시하고 바로 catch에 전달된 클로저를 실행 시킵니다.

```swift
enum SomeError : Error {
    case invalidError
}

// 1. 파라미터 success에 true를 넘기면 fulfilled
// false를 넘기면 rejected로 변경되는 Promise 객체 인스턴스를 생성하여 반환
func request(success: Bool) -> Promise<Bool> {
    return Promise<Bool> { fulfill, reject in
        if success {
            // Promise의 상태를 fulfilled로 변경
            fulfill(true)
        } else {
            // Promise의 상태를 rejected로 변경
            reject(SomeError.invalidError)
        }
    }
}

request(success: true).delay(3).then { result in
    // 5. then 호출 시 요청 결과를 출력
    print("called then after 3 seconds")
}.catch { error in
    // 6. catch 호출 시 에러를 출력
    print("called catch after 3 seconds")
}
```

위 코드를 실행 시키면 request 메서드의 success 파라미터에 true를 전달하였으므로 then 오퍼레이터에 전달된 클로저가 호출이 될 것입니다.

단, request 함수에서 반환된 Promise객체 인스턴스에 then을 바로 호출하는 것이 아니라 delay를 먼저 호출했죠?

따라서, 위 코드를 실행 시키면 "called then after 3 secondes"라는 로그가 바로 출력되지 않고 3초 후에 출력될 것입니다.

#### Recover

recover는 catch와 유사하게 Promise가 rejected로 변경됐을 때 호출됩니다. 그러나 catch와 다른 점은 catch는 오류에 대한 처리 후 Promise 체인이 종료되지만 recover는 오류에 대한 수정 후 체인을 계속 이어나갈 수 있습니다.

```swift
enum SomeError : Error {
    case invalidNumber
}

// 1. startNum부터 endNum까지의 합을 구하여 반환하는 메서드
func sum(startNum: Int, endNum: Int) throws -> Int {
    guard startNum >= 0 && endNum >= 0 else {
        throw SomeError.invalidNumber
    }

    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

// 2. Promise의 객체 인스턴스를 생성하여 1부터 10000000까지 더하는 작업을 하는 클로저를 파라미터로 전달
Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: -1, endNum: 10_000_000)
}.recover { error -> Int in
    return 0
}.then { result in
    print(result)
}
```

위 예제를 보시면 첫번째 sum 메서드를 호출할 때 startNum에 -1을 전달하여 오류가 발생하고 있습니다. 이때, catch 대신 recover 오퍼레이터를 호출하면서 오류가 발생했을 때 결과값을 0으로 바꾸는 클로저를 전달했죠.

그러면 마지막 then의 클로저에 0이 전달되고 0이 출력될 것입니다.

#### Reduce

reduce는 swift 표준 라이브러리 중 reduce(_:_:)와 유사한 역할을 하는 오퍼레이터입니다. 즉, 배열과 클로저를 인자로 받아서 배열의 원소들을 순서대로 클로저의 파라미터로 하여 클로저을 배열의 개수만큼 호출해 줍니다.

```swift
let numbers = [1, 2, 3]
Promise("0").reduce(numbers) { partialString, nextNumber in
    Promise(partialString + ", " + String(nextNumber))
}.then { string in
    print("Final result = \(string)")
}
```

위 코드의 실행 결과는 아래와 같습니다.

```text
Final result = 0, 1, 2, 3
```

#### Timeout

timeout 오퍼레이터는 호출된 후 파라미터로 전달된 시간만큼 대기 후 시간 내에 Promise의 상태가 fulfilled 상태가 되지 않으면 rejected 상태로 변경됩니다.

이때 catch 오퍼레이터의 클로저에는 timeout 에러가 파라미터로 전달됩니다.

```swift
func sum(startNum: Int, endNum: Int) throws -> Int {
    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

let promise = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 1, endNum: 10_000_000)
}

// promise가 3초 내에 fulfilled로 변경되지 않으면 자동으로 rejected로 변경
promise.timeout(3).then { result in
    print(result)
}.catch { error in
    // timedOut error
    print(error)
}
```

위 코드는 1부터 1000만까지 더하되 더하는 작업이 3초 내에 끝나지 않으면 rejected 상태가 되는 코드입니다.

#### Validate

validate는 Promise에 저장된 결과값을 검증하는 오퍼레이터입니다.

```swift
func sum(startNum: Int, endNum: Int) throws -> Int {
    var result = 0
    for num in startNum...endNum {
        result += num
    }
    return result
}

let promise = Promise<Int>(on: .global()) { () -> Int in
    return try sum(startNum: 1, endNum: 100)
}

// validate에 전달한 클로저의 반환값이 true이면 fulfilled, false이면 rejected가 됨
promise.validate { result -> Bool in
    return result == 5050
}.then({ result in
    print(result)
}).catch { error in
    // timedOut error
    print(error)
}
}
```

위 코드는 1부터 100까지 더하여 그 결과를 출력하는 코드 입니다.

1부터 100까지의 합은 5050이므로 validate에 전달된 클로저의 반환값은 true가 되어 Promise의 상태는 fulfilled가 됩니다.

만약 validate에서 false를 반환한다면 Promise의 상태는 rejected가 될 것입니다.

### 마무리

---

여기까지 Google Promises의 오퍼레이터들을 알아봤습니다.

모든 오퍼레이터를 다 언급하다보니 매우 긴 글이 되었네요.

Promise에 대해 궁금해 하시는 분들에게 도움이 되었으면 좋겠습니다.