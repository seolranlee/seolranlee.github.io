---
layout: post
title: React hooks는 실제로 어떻게 동작할까? - 1.왜 hooks는 호출순서에 의존할까?
desc: React hook의 규칙 중 hook이 호출 순서에 의존하는 이유를 알아봅니다.
---

해당 포스트는 [React.js Deep Dive 시리즈 중의 3편인 How Do React Hooks Actually Work?](https://youtu.be/1VVfMVQabx0)을 기반으로 작성하였습니다.

리액트에서 hooks를 사용해 개발을 해보았다면 모두가 알고 있는 규칙이 있다.
바로 최상위에서만 hook을 호출해야 한다는 것이다. (조건문 혹은 중첩된 함수 안에서 hook을 호출해서는 안된다.)  hook의 실행 순서가 언제나 동일하게 보장되어야 하기 때문이다.

[리액트 공식문서에 기재되어 있는 Hook의 규칙](https://ko.reactjs.org/docs/hooks-rules.html#gatsby-focus-wrapper)

공식문서에 기재되어 있으니 이 규칙에 따라 관성적으로 hook을 사용하고 있다곤 했어도, 그 이유에 대해 깊이 생각해본적이 없었는데, 현재 스터디하고 있는 React deep dive영상에서 관련 내용도 다루고 있어서 이 기회에 이 주제로 deep dive를 해보기로 하였다.

# React hooks를 둘러싼 다양한 질문들

- 왜 반복문이나 조건문 안에서 hooks를 호출할 수 없을까?(=왜 hooks는 호출 순서에 의존하는가)
- 왜 이것에 대한 hooks이 없나요? 왜 저것에 대한 hooks가 없나요? (이런 기능을 하는 hooks을 만들 수 있을것 같은데 왜 없냐는 질문)

이번 포스트에서는 상기의 두가지의 중요한 질문들 중 첫번째 질문에 대한 답을 다루어보려고 한다.

# 왜 hooks는 호출 순서에 의존할까

React공식 문서에 의하면 hook은 항상 동일한 순서로 호출되는 것이 보장되어야한다.
반복문이나 조건문 안에서 hooks를 호출할 수 없는 이유는 모두 이 때문이다. (반복문이나 조건문 안에서 hooks를 사용하면 동일한 호출 순서가 보장되지 않기 때문이다.)

## hooks는 배열과 관련되어 있다.

useState hooks가 리버스 엔지니어링을 통해 구현되는 방법을 이해해보자. 우리만의 React 버전을 빠르게 구현해보자.

## useState구현

```jsx
const ReactX = (() => {
  const useState = (initialValue) => {
    let state = initialValue;

    const setterFunction = (newValue) => {
      state = newValue;
    };

    return [state, setterFunction];
  };
  return {
    useState,
  };
})();

const { useState } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);
};

// 렌더러처럼 사용
Component(); // 1
```

초기 값이 1인 counter와 이 상태를 변경하는 setter함수를 반환하는 간단한 hook을 만든다.  그리고 Component에서 이 hook을 사용하여 단순히 카운터 값을 출력하는 console.log를 추가한다.(렌더러처럼 사용)

Component를 호출하여 카운터 값을 출력해본다. 예상하는 대로 1이 출력된다.

자 이제 상태를 변경해보자.

```jsx
const ReactX = (() => {
  const useState = (initialValue) => {
    let state = initialValue;

    const setterFunction = (newValue) => {
      state = newValue;
    };

    return [state, setterFunction];
  };
  return {
    useState,
  };
})();

const { useState } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component(); // 1
// React에서 상태를 변경하면 리렌더가 발생한다는 것을 알고 있으므로 여기선 간단히 Component를 다시 호출해준다.
Component(); // 2를 기대하지만 그렇지 않다. // 1이 출력된다.
```

무엇이 문제이며 어떻게 고쳐야 할까?

문제는 useState가 호출될때 마다 서로 다른 함수라는 점이다. 카운터 값은 실제로 2로 변경되지만, 첫 번째 함수 내부에서만 변경된다.

이것을 어떻게 고쳐야 할까?
hook 스코프 밖에서 state를 정의해주자.

```jsx
const ReactX = (() => {
  let state;
  const useState = (initialValue) => {
    state = initialValue;

    const setterFunction = (newValue) => {
      state = newValue;
    };

    return [state, setterFunction];
  };
  return {
    useState,
  };
})();

...

Component(); // 1
Component(); // 1
```

여전히 제대로 작동하고 있지 않고(1이 출력되고 있고) 그 이유는 고칠 것이 하나 더 있기 때문이다.

```jsx
const ReactX = (() => {
  let state;
  const useState = (initialValue) => {
    // 이 line은 state를 1로 설정한다. 이것은 우리가 원하는 것이나 hook이 호출될 때마다 설정된다. 이것은 우리가 원하는 것이 아니다.
    state = initialValue;

    ...
```

우리가 원하는 것은 hook이 처음 실행 될 때만 초기값으로 세팅되는 것이다. 

```jsx
const ReactX = (() => {
  let state;
  const useState = (initialValue) => {
    if (state === undefined) {
      state = initialValue;
    }
...
Component(); // 1
Component(); // 2
```

기본적으로 상태 값은 정의되지 않으므로 정의되지 않았는지 확인하고 그럴 경우에만 초기 상태로 세팅한다. 이제 콘솔에서 1과 2를 차례로 볼 수 있다. 꽤 멋지지만 한 가지 작지 않은 문제가 더 남아있다.

현재 우리는 하나의 상태 변수만을 가지고 있기 때문에, 우리의 어플리케이션에서는 하나의 상태밖에 존재할 수 없다. (그러나 실제 리액트를 사용해보면 알 수 있듯이) 우리는 무제한의 state에 접근할 수 있어야 한다.

배열로 바꾸면 이것이 가능해진다.

```jsx
const ReactX = (() => {
	// state를 배열로 관리.
  let state = [];

  // 배열을 사용하기 위해서는 index가 필요하다.
  let index = 0;
  const useState = (initialValue) => {
    const localIndex = index;
    // 다음 hook를 위해 index를 증가시켜 둠
    index++;
    if (state[localIndex] === undefined) {
      state[localIndex] = initialValue;
    }

    const setterFunction = (newValue) => {
      state[localIndex] = newValue;
    };

    return [state[localIndex], setterFunction];
  };
  return {
    useState,
  };
})();

const { useState } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component(); // 1
Component(); // 1
```

테스트를 해보면 또 1이 두번 출력된다. 우리가 고쳐야 할 것이 하나 있기 때문이고 이번에는 꽤 명백하다. React 내부의 index는 증가만 하고 있을 뿐 절대 0으로 재설정되지 않는다.

```jsx
const ReactX = (() => {
  let state = [];

	 ...

    return [state[localIndex], setterFunction];
  };

  // index를 0으로 리셋해주는 간단한 함수를 만들어 같이 내보내준다.
  const resetIndex = () => {
    index = 0;
  };
  return {
    useState,
    resetIndex,
  };
})();

const { useState, resetIndex } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component(); // 1
// 렌더러 사이에 리셋함수를 호출한다.
resetIndex();
Component(); // 2
```

이제 useState를 사용할 수 있다.  이제 (렌더링 사이에서 index를 리셋하는 것만 기억하면) 자유롭게 여러개의 state를 생성하여 사용할 수 있다. 

물론, React는 다른 많은 일들을 뒤에서 하기 때문에 실제의 useState대신 사용하기에는 불가능할 것이다. (예를 들어 React는 state가 변화하면 자체적으로 리렌더를 발생시킨다.) 그러나 해당 코드들은 현실에 매우 근접한 코드라 할 수 있다.

해당 코드에서 왜 localIndex라는 추가 변수를 만들까? 우리가 추가 변수를 만들지 않고 index를 대신 사용했다면 세터 함수는 hook이 호출된 후에 실행되기 때문에 잘못된 index를 사용했을 것이다.

이제는 hooks의 순서가 보장되어야 하는 이유를 알 수 있을 것이다. state는 배열이며, useState는 index를 사용하여 이 배열에 접근한다. 반복문 또는 조건문 내부에서 useState를 호출한 경우 각 렌더마다 index가 다를 수 있으며 이는 각 hook이 잘못된 state를 사용하여 우리의 어플리케이션에 버그를 발생할 수 있게 한다.

 

useState를 다루었으므로 useEffect의 구현도 다루어보도록 하자. useState에 이어 두번째로 중요한 hook이기도 하고, 구현도 그리 어렵지 않기 때문이다.

## useEffect구현

```jsx
const ReactX = (() => {
  ...

  // 종속성이 없으면 콜백이 항상 호출되고 종속성이 있으면 종속성이 변경된 경우에만 콜백이 호출된다.
  const useEffect = (callback, dependencyArray) => {
    // 그래서 내부에서 hasChanged라는 변수를 생성하고 기본값은 true로 둔다.
    // 종속성이 있는 경우, hasChanged를 계산한다.
    // 이러한 비교를 위해 우리는 이전 종속성에 접근할 수 있어야 한다.
    // 배열을 사용해 상태를 저장하던 것과 동일한 방식으로 종속성을 저장한다.
    let hasChanged = true;
    if (hasChanged) {
      callback();
    }
  };

  const resetIndex = () => {
    index = 0;
  };
  return {
    useState,
    useEffect,
    resetIndex,
  };
})();
```

```jsx
const ReactX = (() => {
  // effect도 같이 관리하고 있으므로, hooks로 이름을 바꿔준다.
  let hooks = [];
  let index = 0;
  const useState = (initialValue) => {
    const localIndex = index;
    index++;
    if (hooks[localIndex] === undefined) {
      hooks[localIndex] = initialValue;
    }

    const setterFunction = (newValue) => {
      hooks[localIndex] = newValue;
    };

    return [hooks[localIndex], setterFunction];
  };

  // 종속성이 없으면 콜백이 항상 호출되고 종속성이 있으면 종속성이 변경된 경우에만 콜백이 호출된다.
  const useEffect = (callback, dependencyArray) => {
    let hasChanged = true;

    const oldDependencies = hooks[index];

    if (oldDependencies) {
      hasChanged = false;

      // 현재의 의존성 배열을 반복문을 돌려 이전의 의존관계와 다른지 확인한다.
      dependencyArray.forEach((dependency, index) => {
        const oldDependency = oldDependencies[index];
        const areTheSame = Object.is(dependency, oldDependency);
        // 둘이 같지 않으면 종속셩의 변경이 발생한 것
        if (!areTheSame) {
          hasChanged = true;
        }
      });
    }

    if (hasChanged) {
      callback();
    }

    hooks[index] = dependencyArray;
    index++;
  };

  const resetIndex = () => {
    index = 0;
  };
  return {
    useState,
    useEffect,
    resetIndex,
  };
})();

const { useState, useEffect, resetIndex } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);

  console.log(counterValue);

  // 빈 종속성 배열을 넣어 한번만 실행되도록 한다.
  useEffect(() => {
    console.log("useEffect");
  }, []);

  if (counterValue !== 2) {
    setCounterValue(2);
  }
};

Component();
resetIndex();
Component();

// 출력결과
1
useEffect  // 실제로 한번만 호출된다.
2
```

조금 더 테스트를 해보기 위해 name이라는 다른 state를 만들고 Thomas라고 set해주자.

```jsx
const ReactX = (() => {
  let hooks = [];
  let index = 0;
  const useState = (initialValue) => {
    const localIndex = index;
    index++;
    if (hooks[localIndex] === undefined) {
      hooks[localIndex] = initialValue;
    }

    const setterFunction = (newValue) => {
      hooks[localIndex] = newValue;
    };

    return [hooks[localIndex], setterFunction];
  };

  const useEffect = (callback, dependencyArray) => {
    let hasChanged = true;

    const oldDependencies = hooks[index];

    if (oldDependencies) {
      hasChanged = false;

      dependencyArray.forEach((dependency, index) => {
        const oldDependency = oldDependencies[index];
        const areTheSame = Object.is(dependency, oldDependency);
        if (!areTheSame) {
          hasChanged = true;
        }
      });
    }

    if (hasChanged) {
      callback();
    }

    hooks[index] = dependencyArray;
    index++;
  };

  const resetIndex = () => {
    index = 0;
  };
  return {
    useState,
    useEffect,
    resetIndex,
  };
})();

const { useState, useEffect, resetIndex } = ReactX;

const Component = () => {
  const [counterValue, setCounterValue] = useState(1);
  const [name, setName] = useState("Thomas");

  console.log(counterValue);
  console.log(name);

  // 의존성 배열에 name을 추가하여 name이 변화될 때만 실행하도록 한다.
  useEffect(() => {
    console.log("name changed");
  }, [name]);

  if (counterValue !== 2) {
    setCounterValue(2);
  }

  // counterValue조건을 추가하여 세번째 렌더링 때 Jack으로 바뀔 수 있도록 특정한다.
  if (name !== "Jack" && counterValue === 2) {
    setName("Jack");
  }
};

Component();
resetIndex();
Component();
// 이제 resetIndex와 렌더러를 다시 호출해서 결과를 확인한다.
resetIndex();
Component();

// 결과
1
Thomas
name changed
2
Thomas
2
Jack
name changed
```

기대한 대로 결과가 잘 출력되는 것을 볼 수 있다.

이러한 과정을 통해 이제는 꽤 명백하게 왜 hooks의 호출 순서가 언제나 동일하게 보장되어야 하는지 알수 있을 것이다. 

# 참고자료

- [https://youtu.be/1VVfMVQabx0](https://youtu.be/1VVfMVQabx0)
- [https://ko.reactjs.org/docs/hooks-rules.html#gatsby-focus-wrapper](https://ko.reactjs.org/docs/hooks-rules.html#gatsby-focus-wrapper)