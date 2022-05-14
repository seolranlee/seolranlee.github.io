---
layout: post
title: 비동기 요청 취소하기
---


## 필요하게 된 이유

필요하게 된 이유를 적어야할지에 대한 고민이 있었다. 그 이유는 슬프게도 이 포스팅의 동기가 된 실무에서의 장애 현상이 있었는데, 결국 그 장애를 해결하진 못했기 때문이다. 하지만 해당 주제에 대해 생각하게 된 계기가 되었기 때문에 그 의의가 있다고 생각하여 서술을 해보려고 한다.

 내가 속한 서비스 회사인 숨고에서는 카카오 픽셀이라는 트래커를 활용하고 있다. 어느날 카카오 픽셀 내부적으로 장애가 발생하였고, 이것이 숨고 서비스에까지 영향을 미치게 되었다. 카카오 픽셀에서는 내부적으로 `pageView()`라는 메서드가 구현되어 있는데, 이를 호출하면 해당 트래커를 심은 서비스 페이지의 전환율을 볼 수 있다. 때문에 라우터 전환시에 항상 호출되는 메소드로 구현되어있는데, 이 메소드가 호출되면 카카오 픽셀 내부적으로 또 다른 네트워크 요청이 발생된다. 그날의 카카오픽셀 장애는 이 네트워크 요청이 `pending`상태로 계속 유지되는 현상이었다. (정상적인 때에는 200으로 잘 떨어진다.)

 숨고 서비스에서의 문제상황의 재현 flow는 이러했다.

1. 숨고 서비스 메인 접속 (tracker가 init되면서 `kakaoPixel.pageView()` 호출=>`pending`상태 지속)
2. 다른 페이지로의 전환 (라우터 전환시 `kakaoPixel.pageView()` 호출=>`pending`상태 지속)
3. 브라우저 백버튼으로 메인으로 되돌아가려는 시도를 했을 때 2번 페이지에서 메인으로 넘어가지 않고 무한 로딩되는 현상 발생.

## 해결하려는 시도

숨고 서비스 내부적으로 트래커는 `nonblocking call`의 코드로 구현이 되어 있다. 그래서 트래커에 장애가 발생한다고 해도 서비스에 영향을 받을 일은 사실상 없었다. (**코드상**으로는 그렇다.) 하지만 이렇게 외부 서드파티를 쓰게 되면, 내부적인 코드까지 우리가 컨트롤을 해줄 수 없기 때문에 이런 성격의 장애는 해결하기 힘들어진다.

처음에는 `kakaoPixel.pageView()`메서드가 비동기 요청을 내부적으로 보내고 있으므로, 숨고 서비스 메인 진입 후 라우터 전환시에 해당 비동기 요청을 cancel하는 방향으로 구현을 하려고 했었다. 하지만...

`kakaoPixel.pageView()` 메서드는 카카오 픽셀 내부적으로 구현된 메소드이고 이 메소드 내부적으로 어떤식으로 비동기 요청을 콜 하고 있는지 나로서는 알 수 있는 방법이 없었다. 그렇기 때문에 장애가 발생한 비동기 요청을 `cancel`하는 것도 숨고 서비스 코드 레벨에서는 구현할 수 있는 방법이 마땅치 않다는 결론이 내려졌다.

## 그리하여 실패

개인적인 차원에서의 분석 및 결론은 이러하고 다음주에 이를 챕터차원에서 공유하고 도움을 요청할 계획인데, 포스팅의 본론에 오기까지의 컨텍스트는 이러하다. 장애에 대한 명쾌한 해결은 실패로 끝났으나 비동기 요청의 명시적인 취소에 대해서 생각해 볼 수 있는 계기가 되었다. (사실 그 이전에는 한번도 생각해본 적 없는 주제였다.)

## 비동기 요청 취소를 언제할까?

그동안 원할한 속도로 200이 떨어지는 네트워크 요청들만 경험해서 그런것일까. 이 주제에 대해 왜 이제야 생각을 해보게 되었나 싶을정도로 비동기 요청이 취소되어야 할 때는 많아 보였다. 바로 최근 카카오 픽셀 장애와 같은 상황도 마찬가지인데, 카카오 픽셀의 비동기 요청을 내부적으로 접근해서 수정가능했다면, 메인 첫 진입시 `Tracker.init()`으로 인해 호출된 `kakaoPixel.pageView()`의 비동기 요청을 라우터 전환시(페이지 전환시)에는 취소시켜주는 방법 등으로 해결할 수 있을것이다.

이런 비슷한 상황을 재현해보는 예제를 구현해보았다.


## 예제

예제코드는 빠르게 CRA로 만들어보았고, api는 [jsonplaceholder](https://jsonplaceholder.typicode.com/), 그리고 임의로 네트워크의 장애 상황을 구현하기 위해 크롬 익스텐션인 [URL Throttler](https://chrome.google.com/webstore/detail/url-throttler/kpkeghonflnkockcnaegmphgdldfnden?hl=en)를 사용하였다.


```jsx
import axios from "axios";
import { useEffect, useState } from "react";
const App = () => {
  const [todo, setTodo] = useState(null)
  const getTodo = async () => {
    try {
      const response = await axios.get('https://jsonplaceholder.typicode.com/todos/1');
      setTodo(response.data)
      console.log(response.data);
    } catch (error) {
      console.error(error);
    }
  }
  
  useEffect(() => {
    console.log('todo change')
  }, [todo])
  
  return (
    <div>
      <button onClick={getTodo}>getTodo</button>
      {
        todo && (
          <div>
            <p>{todo.userId}</p>
            <p>{todo.id}</p>
            <p>{todo.title}</p>
            <p>{todo.completed ? '완료' : '미완료'}</p>
          </div>
        )
      }
    </div>
  );
}

export default App;
```

해당 코드는 https://jsonplaceholder.typicode.com/todos/1 api를 호출하여 todo 하나를 가져오는 정말로 아주아주 간단한 코드이다. 여기에 임의로 https://jsonplaceholder.typicode.com/* 에 대응되는 모든 api를 Throttler 걸어준다. (네트워크 요청을 설정해준 시간동안 모두 pending상태로 해준다.)

<p align="center">
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FwOhBj%2FbtrB8P1FVCZ%2FWQhnUCRz5oNuFRH4zQS6K1%2Fimg.png" alt="URL Throttler">
</p>

pending 되는 시간은 2초정도로 설정해두었다.


[ ![network pending](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fz0PZV%2FbtrB6uqXEcI%2F0tIH9kReQVPzPtSQ7eqexk%2Fimg.png) ](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fz0PZV%2FbtrB6uqXEcI%2F0tIH9kReQVPzPtSQ7eqexk%2Fimg.png)


동일한 네트워크 요청에 대해서 쌓이고 있는 pending들을 볼 수 있다. 게다가 해당 요청들이 2초를 넘긴후 차례대로 응답값이 오게되면 불필요한 돔 리렌더링도 반복적으로 발생하게 된다. (`useEffect`의 `console.log('todo change')`가 반복적으로 찍히는 것을 보면 알 수 있다.) 

숨고의 서비스 장애 상황과 완전히 똑같은 예시는 아니지만, 버튼 클릭을 페이지 전환과 치환해서 생각해보면, 유사한 상황이라는 것을 알 수 있다. 동일한 네트워크 요청에 대해서 다음 버튼 클릭 행동시에는 이전 네트워크 요청을 취소하고 재요청한다면 이러한 불필요한 중복 네트워크 요청을 방지할 수 있을것이다.


## 비동기 요청 취소하기

### axios에서 비동기 요청 취소하기

보통 실무에서는 주로 axios를 사용하여 비동기 통신을 할 것이라고 생각된다. axios에서는 취소 토큰이란 걸 지원하고 있고, 이를 통해 비동기 요청을 취소할 수 있다. [axios에서의 요청취소](https://yamoo9.github.io/axios/guide/cancellation.html)

상기 예제코드에 이를 적용해 보면 이렇게 되겠다.

```jsx
import axios from "axios";
import { useState, useRef, useEffect } from "react";
const App = () => {
  const [todo, setTodo] = useState(null)
  const CancelToken = axios.CancelToken;
  const source = useRef(null);


  const getTodo = async () => {
    if(source.current) {
      source.current.cancel();
    }
    source.current = CancelToken.source();
    try {
      const response = await axios.get('https://jsonplaceholder.typicode.com/todos/1', {
        cancelToken: source.current.token
      });
      setTodo(response.data)
    } catch (error) {
      console.error(error);
    }
  }
  
  useEffect(() => {
    console.log('todo change')
  }, [todo])
  
  return (
    <div>
      <button onClick={getTodo}>getTodo</button>
      {
        todo && (
          <div>
            <p>{todo.userId}</p>
            <p>{todo.id}</p>
            <p>{todo.title}</p>
            <p>{todo.completed ? '완료' : '미완료'}</p>
          </div>
        )
      }
    </div>
  );
}

export default App;
```

### AbortController를 활용하여 비동기 작업취소하기

axios가 아닌 비동기가 api를 활용하여 비동기 작업을 했을 경우에도 비동기 작업을 취소하고 싶어지는 상황이 발생할 수 있다. (이번 포스팅에서는 대표적으로 Fetch api를 예를 들어 작성해보겠다.)

이를 위해 DOM에서 범용적으로 사용 가능한 [AbortController](https://developer.mozilla.org/ko/docs/Web/API/AbortController)를 활용하여 비동기 요청을 취소해보겠다. (MDN 문서를 확인하면 알수 있듯이 AbortController는 fetch요청을 비롯한 모든 DOM 요청을 취소할 수 있다.)

AbortController를 이전 예제 코드에 적용하면 이렇게 될 수 있겠다.

```jsx
import { useState, useEffect, useRef } from "react";
const App = () => {
  const [todo, setTodo] = useState(null)
  const abortController = useRef(null);

  const getTodo = () => {
    if(abortController.current) { // already exists request
      abortController.current.abort();
    }
    abortController.current = new AbortController();
    fetch("https://jsonplaceholder.typicode.com/posts/1", { signal: abortController.current.signal })
      .then((response) => response.json())
      .then((data) => {
        setTodo(data)
      });
  }
  
  
  useEffect(() => {
    console.log('todo change')
  }, [todo])
  
  return (
    <div>
      <button onClick={() => {
        getTodo();
      }}>getTodo</button>
      {
        todo && (
          <div>
            <p>{todo.userId}</p>
            <p>{todo.id}</p>
            <p>{todo.title}</p>
            <p>{todo.completed ? '완료' : '미완료'}</p>
          </div>
        )
      }
    </div>
  );
}

export default App;
```
이번 포스팅을 작성하며 알게 된 사실인데, axios의 CancelToken을 이용한 비동기 요청 취소 로직과 AbortController를 활용하여 비동기 요청을 취소하는 로직이 크게 다르지 않아 막연히 axios의 CancelToken이 내부적으로 AbortController 기반으로 구현되지 않았을까? 라고 생각했는데 axios의 CancelToken은 이미 deprecated된 스펙이었다. (https://axios-http.com/docs/cancellation 참고. v0.22.0 이후부터는 AbortController를 권장한다는 내용)

그리하여 axios에 CancelToken이 아닌 AbortController를 적용한다면 axios에 대한 예제 코드가 최종적으로 이렇게 변하게 되겠다.

```jsx
import axios from "axios";
import { useState, useRef, useEffect } from "react";
const App = () => {
  const [todo, setTodo] = useState(null)
  const abortController = useRef(null);


  const getTodo = async () => {
    if(abortController.current) { // already exists request
      abortController.current.abort();
    }
    abortController.current = new AbortController();
    try {
      const response = await axios.get('https://jsonplaceholder.typicode.com/todos/1', { signal: abortController.current.signal });
      setTodo(response.data)
      console.log(response.data);
    } catch (error) {
      console.error(error);
    }
  }
  
  useEffect(() => {
    console.log('todo change')
  }, [todo])
  
  return (
    <div>
      <button onClick={() => {
        getTodo();
      }}>getTodo</button>
      {
        todo && (
          <div>
            <p>{todo.userId}</p>
            <p>{todo.id}</p>
            <p>{todo.title}</p>
            <p>{todo.completed ? '완료' : '미완료'}</p>
          </div>
        )
      }
    </div>
  );
}

export default App;
```


### 다음 버튼 행동 시 이전 네트워크 요청 취소 로직 적용 후

[ ![after](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWvQeG%2FbtrB67hH5ZW%2FUwnsAtIzDpLOMTrfJHIRv1%2Fimg.png) ](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWvQeG%2FbtrB67hH5ZW%2FUwnsAtIzDpLOMTrfJHIRv1%2Fimg.png)


불필요한 이전 요청들이 모두 취소된 걸 볼 수 있다. 더불어 불필요한 돔 리렌더링도 일어나지 않는다. 

## 끝으로

이번 서드파티 장애를 통해 비동기는 쓸수록 어려운 개념이라는 생각을 다시금 하게 되었다. 프론트엔드 개발자로서 가장 많이 하는 작업인 동시에 이 때문에 깊은 이해 없이 코드레벨에서 별 생각없이 쓰기도 쉬운 개념이라고 생각하는데 이번 장애 상황을 통해 비동기에 대해 다시 한번 생각할 수 있는 계기가 되어 좋은 기회가 되었던 것 같다.

## 참고목록
- [MDN AbortController](https://developer.mozilla.org/ko/docs/Web/API/AbortController)
- [자바스크립트에서 AbortController 를 활용하여 비동기 작업 중단하기](https://genie-youn.github.io/journal/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EC%97%90%EC%84%9C_AbortController%EB%A5%BC_%ED%99%9C%EC%9A%A9%ED%95%98%EC%97%AC_%EB%B9%84%EB%8F%99%EA%B8%B0_%EC%9E%91%EC%97%85_%EC%A4%91%EB%8B%A8%ED%95%98%EA%B8%B0.html)
- [[React] CancelToken을 이용한 비동기 요청 취소](https://dghg.github.io/canceltoken/)