---
layout: post
title: React elements vs React components vs Component instances
---

프론트엔드 개발을 함에 있어서 리액트는 거의 필수적인 라이브러리로 자리잡은 듯 한다. (마치 자바스크립트도 잘 모르는 상태로 제이쿼리부터 배워서 웹 개발을 하던 때 처럼..?)

나 역시 회사에서 리액트를 사용하여 개발하고 있고, 그 이전 회사에서도 레거시 프로젝트의 마이그레이션을 위한 새로운 기술 스택으로 리액트를 선택하여 사용했었다. 아니 사실 그 이전부터도 실무에서가 아니더라도 리액트는 사용하고 있었다. 프론트엔드 개발자라면 모두가 리액트가 나온 그 시점부터 그 어떤 경로를 통해서라도(실무에서 리액트를 다루지 않는다면 사이드 프로젝트나 대외할동이나 무엇이든!) 한 번쯤은 사용해보지 않았을까? 나 역시 그러하였다. 그런데, 최근 리액트에 대해서 내가 얼마나 잘 아는가 하는 의구심이 들었다.

실무에서 사용하는 관성적인 차원의 앎 을 뜻하는 것이 아닌 리액트가 실제로 어떻게 동작하고 어떤 문제를 해결하고자 등장하였고 이 프레임워크가 지닌 철학같은 것들을 내가 잘 이해하고 쓰고 있는지 관점에서의 ‘앎’ 말이다.

이렇게 공개적으로 적기는 (매우매우) 부끄러운데, 최근 프론트엔드 개발을 시작하려고 공부하는 중인 지인의 코드를 봐주다가 한가지 트러블슈팅에 봉착하였다. 

```jsx
const [courseList, setCourseList] = useState([])

const editCourseList = async (newCourse) => {
  setCourseList([...courseList, newCourse])
  await editCourseListApi(courseList)
}
```

대충 이러한 맥락의 코드였는데 (어디까지나 의사코드다), 하고 싶었던 일은, ui를 통해 courseList의 상태가 변경 되면 (추가되거나, 수정되거나, 삭제될 때. 예시 코드는 추가되는 상황으로만 특정하였다.) setCourseList로 이를 반영해주고, 서버에서 수정된 courseList를 전달해주는 api통신을 하는 것이다. 꽤나 간단한 일이지 않는가? 그런데 이상하게 setCourseList이후 리렌더링이 발생하여 화면에 courseList의 갱신된 상태값들은 반영이 되었는데 그 이후 새로고침을 하면 그 이전 값으로 원복이 되어있는 현상이 있었다. 실제로 네트워크 탭을 열어 payload객체를 살펴보니, setCourseList를 통해 갱신된  courseList가 아닌 이전값이 담겨서 서버에 전송되고 있었다. 

왜 이러한 현상이 발생했을까? 아마 너무 기본적인 것이라 당황스러울 수도 있겠는데, 본인은 이러한 기본적이면서 중요한 개념에서 트러블 슈팅을 맞이하고 리액트를 쓰고 있는 프론트엔드 개발자로서의 회의감과 부끄러움과 현타가 밀려왔다(…)

setState는 리액트 내부적으로 비동기로 작동하고 이말인즉슨 setState는 순서를 보장받을 수 없다. 더 자세한 이야기는 Dan Abramov의 블로그 [useEffect 완벽 가이드](https://overreacted.io/ko/a-complete-guide-to-useeffect/)에 자세히 나와있다. (이미 너무 유명한 글이지만 좋은 글이므로 읽어보기를 추천한다) 

그리하여 state의 변경은 effect로 관리하여야 한다. (저렇게 윗줄에 setState를 해줬다고 state가 바로 반영될 것이라고 대하고 냅다 api를 호출하는 것이 아니라..)

트러블 슈팅이라고 말하기에도 민망한 이 에피소드는 리액트에 대한 나의 지식 수준을 돌아보게 하였는데 그리하여 최근에 추천 받은 [React의 동작 원리와 개념들이 담겨있는 플레이 리스트](https://www.youtube.com/watch?v=7YhdqIR2Yzo&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ)를 차근 차근 공부해보기로 마음을 먹게 되었다. 적지 않은 내용이고 다루는 내용들의 깊이나 수준도 얕은 편이 아니라 앞으로 여러 차례에 걸쳐서 해당 주제에 대해 게시를 할 생각이다. (이 에피소드와 관련된 개념도 다루게 될 것이다.)

# React elements vs React components vs Component instances

해당 주제는 [React deep dive영상](https://www.youtube.com/watch?v=7YhdqIR2Yzo&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ)의 첫번째 주제이다. 영상의 설명만으로는 헷갈릴 수 있는 지점들이 있어서 여러 자료들을 참고 하였고 기본적인 골자는 React의 개발자인 Dan Abramov이 리액트 공식 홈페이지 블로그에 게시한 [React Components, Elements, and Instances](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html) 포스팅을 따른다.

## 배경

이들에 대해 알아보기 이전에 왜 리액트가 이러한 개념을 구현하게 되었는지 배경에 대해 알아보도록 하자. 배경에 대한 이해가 선행된다면 이러한 개념들이 해결하고자 하는 문제를 알 수 있기 때문에 더 깊이 있는 이해가 가능해 질 것이다.

### 전통적인 객체 지향 UI 프로그래밍에서의 UI 모델

```jsx
class Form extends TraditionalObjectOrientedView {
  render() {
    // Read some data passed to the view
    const { isSubmitted, buttonText } = this.attrs;

    if (!isSubmitted && !this.button) {
      // Form is not yet submitted. Create the button!
      this.button = new Button({
        children: buttonText,
        color: 'blue'
      });
      this.el.appendChild(this.button.el);
    }

    if (this.button) {
      // The button is visible. Update its text!
      this.button.attrs.children = buttonText;
      this.button.render();
    }

    if (isSubmitted && this.button) {
      // Form was submitted. Destroy the button!
      this.el.removeChild(this.button.el);
      this.button.destroy();
    }

    if (isSubmitted && !this.message) {
      // Form was submitted. Show the success message!
      this.message = new Message({ text: 'Success!' });
      this.el.appendChild(this.message.el);
    }
  }
}
```

- 자식 컴포넌트의 인스턴스를 관리(new연산자를 통해 인스턴스를 생성하거나 render, destory등을 통해 렌더하고 파괴하고 하는 행위들)는 책임을 부모 컴포넌트(코드 작성자)에게 돌림
- 부모 컴포넌트는 자식 컴포넌트 인스턴스 관리 뿐만 아니라 자신의 DOM node도 관리해야 함
- 관리해야 하는 상태가 늘어날 수록 코드 라인은 제곱으로 늘어남
- 부모가 자식 인스턴스를 직접 참조하고 있으므로 추후에 이들을 분리하기 어려워짐

결국 자식의 인스턴스 관리에 대한 책임을 부모에게 맡기면서 생기는 다양한 문제점들이라 할 수 있다.

## React는 이를 어떻게 해결했을까

> *In React, this is where the elements come to rescue.*
> 

### 1. Element는 Tree를 묘사한다.

React에서 element란 컴포넌트를 plain object로 표현한 것이다. (*이것은 인스턴스가 아니다*)

(*is not instance, just tell React what you want to see on the screen* ⇒ 인스턴스가 아니라 화면에 무엇을 그려야 할지 알려주는 요약본)

React에서 JSX는 

```jsx
const App = () => {
  return (
    <button className='button button-blue'>
      <b>
        OK!
      </b>
    </button>
  )
}

const App = () => {
  return /*#__PURE__*/React.createElement("button", {
    className: "button button-blue"
  }, /*#__PURE__*/React.createElement("b", null, "OK!"));
};

// React v17부터는 React.createElement로 하지 않는다
// Inserted by a compiler (don't import it yourself!)
// import {jsx as _jsx} from 'react/jsx-runtime';

// function App() {
//   return _jsx('h1', { children: 'Hello world' });
// }
```

[아래와 같은 (React.createElement 함수를 통해) React Element로 변환된다.](https://babeljs.io/repl)(react17이전)

```jsx
console.log(
  React.createElement("button", {
    className: "button button-blue"
  }, /*#__PURE__*/React.createElement("b", null, "OK!"))
)

// console.log(_jsx('h1', { children: 'Hello world' }))

// element: React.createElement()의 반환값
// DOM element
{
  type: 'button',  // type: (string | Class or Function)
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      children: 'OK!'
    }
  }
}
```

type이 string인 경우 type은 해당 컴포넌트가 어떤 HTML Tag인지를 표현하고, props는 해당 HTML Tag의 속성들을 명시한다.

element의 type이 Class나 Function이면 어떨까?

```jsx
// Component element
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

여기서 바로 리액트의 핵심 아이디어가 나온다.

***element(Dom element, Component element)들은 서로 중첩되고 섞일 수 있다.***

```jsx
const DeleteAccount = () => (
  <div>
    <p>Are you sure?</p>
    <DangerButton>Yep</DangerButton>
    <Button color='blue'>Cancel</Button>
  </div>
);
```

이 함수형 컴포넌트는 아래와 같은 React element로 해석된다.

```jsx
const DeleteAccount = () => (
  // React element
  {
    type: 'div',
    props: {
      children: [{
        type: 'p',
        props: {
          children: 'Are you sure?'
        }
      }, {
        type: DangerButton,
        props: {
          children: 'Yep'
        }
      }, {
        type: Button,
        props: {
          color: 'blue',
          children: 'Cancel'
        }
      }]
    }
  }
);

```

위 element를 보면 DeleteAccount컴포넌트는 div태그로 실제 렌더링이 되고 자식들로 p dom element, DangerButton component element, Button component element를 지니고 있다. 이 중첩되고 믹스된 엘리먼트 트리를 통해 이들간에 has-a, is-a관계를 파악할 수 있다.

- `Button` is a DOM `<button>` with specific properties.
- `DangerButton` is a `Button` with specific properties.
- `DeleteAccount` contains a `Button` and a `DangerButton` inside a `<div>`.

### 2. Component는 **Element Trees를 캡슐화한다**

Component는 Element에 돔 트리에 전달할 정보만 캡슐화되어 있다.

DeleteAccount컴포넌트는 DangerButton와 Button 컴포넌트에 대해 알고 있는 정보가 없다. (그들이 어떤 메소드를 지녔는지, 어떤 상태를 지녔는지 등…) 즉 DeleteAccount컴포넌트는 전통적인 oop 모델의 ui 프로그래밍같이 자식들의 인스턴스를 관리할 필요가 없게 되고(정보가 캡슐화 되어 있어서 코드 작성자 입장에서 관리를 할 수 있는 방법도 없다) 이들을 서로 분리하는 것도 가능해진다.

리액트가 함수나 클래스 타입(컴포넌트)의 element를 만나면

```jsx
// Component element
{
  type: Button
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

리액트는 Button이 무슨 element를 뱉어내는지 찾아서 다음과 같은 결과를 반환한다.

```jsx
// DOM element
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```

리액트는 페이지의 모든 element에 대한 기본 DOM element들을 알 때까지 이러한 과정을 계속 반복한다. (type이 string이 될 때 까지)

(리액트는 아이들이 세상의 모든 작은 것들(기본 DOM element)을 알아낼 때까지 우리가 설명하는 모든 **"X는 Y"**에 대해 *"Y"가 무엇이냐고* 묻는 것과 같다.)

모든 elements 가 지닌 DOM elements 를 알게 됨 → React가 해당 DOM elements 들을 적절한 때에 create, update, destroy해준다(리액트가 다른 곳에서 수행한다 우리가 해주지 않아도 됨.)

결론적으로, element들은 mix & match를 통해 서로간의 관계를 파악할 수 있는데, 컴포넌트는 엘리먼트 트리를 캡슐화 하고 있으므로 부모에서 컴포넌트 타입에 대해서는 알수 있는 정보들이 없다. 그러므로 이들은 서로 독립적인 관계를 유지해 분리가 가능해지며, 내부적으로 리액트는 엘리먼트 트리에서 컴포넌트 타입을 만나게 되면 이것이 실제로 dom에 렌더링 되는 dom element를 뱉어낼 때 까지 집요하게 물어본다. 이런 과정의 반복을 통해 트리의 모든 정보를 알아낸 리액트는 알아서 이들을 관리해줄 수 있게 된다. (이런 방식때문에 다른 컴포넌트의 내부 구조를 몰라도 서로 독립적으로 합쳐지고 섞일 수 있는 것이다.)

최종적으로 아까 본 Form을 React에서는 다음과 같이 작성할 수 있다.

```jsx
const Form = ({ isSubmitted, buttonText }) => {
  if (isSubmitted) {
    // Form submitted! Return a message element.
    return {
      type: Message,
      props: {
        text: 'Success!'
      }
    };
  }

  // Form is still visible! Return a button element.
  return {
    type: Button,
    props: {
      children: buttonText,
      color: 'blue'
    }
  }
};
```

우리는 리액트에게 Element(요약본)를 전달하기만 했고 실제로 컴포넌트를 생성하고 제거하고 업데이트하는건 리액트가 알아서 해준다.

### 3. Component instances

- **리액트에서** Instance는 위에서 설명한 Element와 Component에 비해 별로 중요하지 않다.
- 리액트의 클래스형 컴포넌트를 인스턴스화한걸 Instance라고 표현한다. (함수형 컴포넌트는 Instance가 없다.)
    - class component 에서의 this
- 엘리먼트의 type이 Class이면 리액트는 새로운 인스턴스를 생성하고 이것의 render메소드를 실행한다.
- 클래스 컴포넌트를 사용하더라도 리액트를 쓸 때 직접 instance를 관리하지 않는다. (인스턴스를 직접 생성하거나 파괴하거나 수정하거나 등등..). 리액트가 알아서 해준다.

# 참고자료

- [https://www.youtube.com/watch?v=7YhdqIR2Yzo&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=1](https://www.youtube.com/watch?v=7YhdqIR2Yzo&list=PLxRVWC-K96b0ktvhd16l3xA6gncuGP7gJ&index=1)
- [https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html](https://reactjs.org/blog/2015/12/18/react-components-elements-and-instances.html)s
- [https://hkc7180.medium.com/react-components-elements-and-instances-번역글-b5744930846b](https://hkc7180.medium.com/react-components-elements-and-instances-%EB%B2%88%EC%97%AD%EA%B8%80-b5744930846b)
- [https://www.youtube.com/watch?v=QSJUTS9PScY](https://www.youtube.com/watch?v=QSJUTS9PScY)
- [https://simsimjae.tistory.com/449](https://simsimjae.tistory.com/449)
- [https://kimchunsick.me/2022-07-16-how-to-work-react/](https://kimchunsick.me/2022-07-16-how-to-work-react/)
- [https://velog.io/@codns1223/Element와-Component의-차이](https://velog.io/@codns1223/Element%EC%99%80-Component%EC%9D%98-%EC%B0%A8%EC%9D%B4)