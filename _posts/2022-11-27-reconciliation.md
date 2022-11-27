---
layout: post
title: Reconciliation
desc: React의 Reconciliation과정에 대해 알아봅니다.
---

해당 포스트는 [React.js Deep Dive 시리즈 중의 1편인 How Does React Actually Work?의 Rconciliation 섹션](https://www.youtube.com/watch?t=334&v=7YhdqIR2Yzo&feature=youtu.be)을 기반으로 작성하였습니다.

- React는 엘리먼트들의 트리를 생성한다.
- 이 과정은 매우 빠르게 이루어지는데 왜냐하면 React element들은 단순 plain JavaScript 객체일 뿐이고 우리는 기본적으로 한순간에 수천 개의 객체를 만들 수 있기 때문이다.
- 이 모든 일은 (클래스 컴포넌트 기준) 우리가 render메서드를 호출할 때 일어난다.  
- 리액트는 맨 위에서부터 맨 아래까지 재귀적으로 이동함으로써 이 element들의 tree를 생성한다. 만약 component 타입을 만나면 그것을 호출하여 무슨 element를 뱉어내는지 알아내고 반환된 react element를 가지고 계속 하강한다.
- 리액트는 이 element 트리를 메모리에 계속 유지하고 있는데 이것을 virtual DOM이라고 부른다.
- 다음으로 해야 할 일은 virtual DOM을 실제 돔과 동기화 하는 작업이다.
초기 렌더에서 React는 전체 트리를 DOM에 삽입해야 한다. DOM에 그러한 종류의 조정을 하는 것은 매우 비용이 많이 들지만, 초기 렌더링에서는 그것을 피할 방법이 없다.

# But what if the tree changes?

state의 변화가 발생하여 다른 값과 element들이 반환되는 경우에는 어떻게 될까?
React는 다시한번 매우 빠르게 element들의 새로운 트리를 생성하고 이제 우리는 두가지의 (이전 것과 새로운 것) 트리를 가지게 된다. 

![two tree](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FDgURq%2FbtrR9M3YZsY%2FJ5Th1mwl7VjehnKbZ07KNk%2Fimg.png)

이제 새 트리를 포함하는 가상 DOM을 실제 DOM과 다시 동기화해야 한다.

트리 전체를 다시 렌더링하는 것은 매우 비효율적인 일이다. 실제 DOM을 변경하는 것은 매우 까다롭기 때문이다. 이것이 React가 이전 트리를 가져와서 새 트리로 변환할 연산의 가장 적은 수를 찾는 이유이다.(비용이 많이 드는 일이기 때문에 리액트는 가장 적은 연산의 방법을 찾으려고 한다.)

트리들을 구별하는 것이 목표이기 때문에 디핑 알고리즘이라고 불리는 알고리즘을 사용한다.

일반적인 상황에서 이것은 매우 어려울 것이다. 예를 들어, 만약 우리가 수천 개의 element들을 가지고 있다면, 그것은 10억 개의 연산을 필요로 할 수 있는데, 이것은 현실적이지 않다.(n개의 엘리먼트가 있는 트리에 대해 O(n^3)의 복잡도를 가진다.)

대신 React는 element들의 수보다 적은 수로 이 작업을 수행한다. 따라서 1,000개의 element의 경우 작업 수는 1,000개보다 적을 것이다. (O(n)의 복잡도) 이것은 믿을 수 없을 정도로 빠르고 오직 두 가지 매우 중요한 가정 때문에 가능하다. (React는 두 가지 가정을 기반하여 O(n) 복잡도의 휴리스틱 알고리즘을 구현하였다.)

- **Heuristic Algorithm**
    
    > **휴리스틱**(heuristics)이란 불충분한 시간이나 정보로 인하여 합리적인 판단을 할 수 없거나, 체계적이면서 합리적인 판단이 굳이 필요하지 않은 상황에서 사람들이 빠르게 사용할 수 있게 보다 용이하게 구성된 간편 추론의 방법이다.
    > 
    > 
    > 
    > 해결법이 정확히 해결되는가에 대한 문제를 배제하고, 경험과 직관을 통해, 일반적으로 좋은 해결법이나, 보다 간단한 해결법을 찾고자 하는 방법이다.
    > 
    

# The Diffing algorithm assumptions

1. 다른 타입의 두 element는 완전히 다른 트리를 생성할 것이다.
2. React는 하위 요소 목록이 있는 경우(변경한 하위 요소 목록이 있는 경우) 올바른 방법으로 키를 제공할 것이라고 가정한다.

예시를 보자.

```jsx
{
    items.map((item, index) => (
        // Not optimal
        <div key={index}>
            {item.name}
        </div>
    ))
}
```

```jsx
{
    items.map((item) => (
        // Optimal
        <div key={item.id}>
            {item.name}
        </div>
    ))
}
```

mapping된 item목록이 있고 고유한 키를 제공해야 한다고 가정해보자.

당신이 가지고 있는 첫 번째 아이디어는 index를 사용하여 키를 제공하는 것이지만, 이것은 우리 대부분이 언젠가는 발견하게 되는 악명 높은 버그 중 하나이다.

이는 예상치 못한 기능 동작으로 이어지며 전혀 놀랄일이 아니다. 우리는 아이템의 key가 항상 같게 유지되어야 한다는 것을 알고 있다. (key값을 index로 주게 되었을 때)우리가 목록의 시작에 새로운 아이템을 추가 하게 되면 하위 아이템의 key가 모두 1씩 증가하게 되는데 이는 잘못된 것이다.

이것이 우리가 id또는 단순 고유 속성을 사용하여 key를 제공하는 이유이다.

앞서, 키가 왜 props의 일부가 아닌지 궁금할 수 있다고 말하였는데 이제 우리는 왜 키가 다른 property와 다른지 이해할 수 있다. - React의 핵심인 Diffing algorithm의 중요한 부분이기 때문이다.

# Diffing algorithm operations

1. DOM node example

    ```jsx
    <div>
        <h1>Title1</h1>
    </div>
    ```

    ```jsx
    <div>
        <h2>Title2</h2>
    </div>
    ```

2. component example 

    ```jsx
    <div>
        <Component1 />
    </div>
    ```

    ```jsx
    <div>
        <Component2 />
    </div>
    ```

서로 다른 타입의 두 element가 같은 위치에 있다고 가정해보자. 이 경우, React는 첫번째 가정에 의해 처음부터 새로운 트리를 만든다. 즉, 이전 트리의 모든 component 인스턴스들이 현재 state와 함께 destoryed된다.

우리는 이것을 component들이 unmount된다고 한다.

만약 같은 위치에 같은 type(`div=div`, `Component=Component`)이지만 다른 attribute를 가진 두 element가 있다면 어떨까?

1. DOM node example

    ```jsx
    <div
        className="first"
    />
    ```

    ```jsx
    <div
        className="second"
    />
    ```

2. component example 

    ```jsx
    <Component 
        someProp="first"
    />
    ```

    ```jsx
    <Component 
        someProp="second"
    />
    ```

리액트는 심플하게 이 attribute를 업데이트하기만 하면 된다. 이 경우에는 인스턴스들은 동일하게 유지되고 state도 유지된다. children을 바꾸는 것으로 넘어가보자.

이름들의 목록을 상상해보자. 목록 끝에 이름을 추가하면 어떨까? React는 먼저 첫번째 item을 비교하고 두 item이 동일하다는 것을 인식한다. 그런 다음, 세번째 item이 새 item을 알아차리고 just 삽입한다. <심플-!>

```jsx
<ul>
    <li>John</li>
    <li>Martin</li>
</ul>
```
```jsx
<ul>
    <li>John</li>
    <li>Martin</li>
    <li>Thomas</li>
</ul>
```

하지만 목록의 시작 부분에 이름을 추가하면 어떨까?

```jsx
<ul>
    <li key="john">John</li>
    <li key="martin">Martin</li>
</ul>
```
```jsx
<ul>
    <li key="thomas">Thomas</li>
    <li key="john">John</li>
    <li key="martin">Martin</li>
</ul>
```

이 경우, React는 동일한 위치의 item을 비교하고 item이 모두 다르다는 것을 인식하므로 기본적으로 새 트리를 생성한다.

하지만 이것은 우리가 원하는 게 아니다.

만약 우리가 10,000개의 item을 가지고 있고, 새로운 item을 추가할 때 우리는 새로운 트리를 만들고 싶지 않다. 

우리가 할 수 있는 것을 key를 제공하는 것이다. key를 사용해 React에 어떤 item이 여전히 동일하고 어떤 item이 새것인지 알려준다. 다시 한번 말하지만, 이것이 우리가 index를 key로 사용할 수 없는 이유이다.(각 item이 지니고 있는 key는 언제나 동일하게 유지되어야 한다.)

# Fiber algorithm

React 16버전부터 도입.

위에서 보았듯, Diffing 알고리즘을 통해 Actual DOM 과 Virtual DOM을 비교하여 틀린 부분을 찾고 재랜더링할 때 **재귀**를 사용한다.

알고리즘에서 DFS를 통해 재귀를 해보았다면 Stack을 이용한다는 것을 알게 된다. 이는 동기 작업을 의미하게 된다. 리액트의 Element를 모두 순회할 때 까지 JS 엔진을 독차지 하게 된다는 문제점이 있다.

![stack](https://miro.medium.com/max/1100/1*PRfPGtWu1luNFAxRekJP7g.gif)

위의 이미지가 재귀적으로 이루어지는 알고리즘의 애니메이션 모양이 된다. 

![fiber](https://miro.medium.com/max/1100/1*hvfXLcMGNXQ4KITbG-EoNg.gif)

그에 반해 Fiber를 도입하게 되면 애니메이션이 자연스럽게 이어지는 효과를 볼 수 있다.

# 참고자료

- [https://youtu.be/7YhdqIR2Yzo](https://youtu.be/7YhdqIR2Yzo)
- [https://ko.reactjs.org/docs/reconciliation.html](https://ko.reactjs.org/docs/reconciliation.html)
- [https://velog.io/@naamoonoo/리액트는-어떻게-작동할까-Diffing-3주차-회고](https://velog.io/@naamoonoo/%EB%A6%AC%EC%95%A1%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%A0%EA%B9%8C-Diffing-3%EC%A3%BC%EC%B0%A8-%ED%9A%8C%EA%B3%A0)