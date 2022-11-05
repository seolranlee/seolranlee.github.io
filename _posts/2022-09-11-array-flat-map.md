---
layout: post
title: flatMap으로 배열을 좀 더 쉽게 다루기
desc: 코드 리뷰를 통해 얻은 경험 중 flatMap이라는 메소드를 소개합니다.
---

숨고의 프론트엔드 챕터 팀은 (개인적으로) 코드리뷰 문화가 굉장히 건강하게 정착된 편이라고 생각한다. 11명의 적지 않은 팀원들간의 코드 리뷰가 원활하게 돌아가게 하기 위해 내부적으로 이전부터 지금까지 다양한 시도와 노력들이 이루어졌을 거라고 생각한다. 

그 중 하나가 github 프론트엔드 레포와 슬랙의 연동이 매우 잘 되어 있다는 점인데, 매일 5시에 나에게 할당된 PR에 대한 리마인드가 이루어진다거나, 프론트엔드 챕터 전용 PR 채널이 있어서 그 곳에서 업데이트 되는 코멘트나 PR의 내용들도 간략하게 오버뷰 형식으로 확인할 수 있다. 

(모든 팀원들이 그러할 것 같은데) 나의 경우는 PR채널방에 심심할때마다 들어가서 내가 올린 PR뿐만이 아니라 다른 사람들의 PR, 그리고 흥미로운 코멘트들을 염탐하곤 하다. 최근에도 어김없이 해당 채널의 새롭게 업데이트되는 내용들을 읽고 있다가 다른 팀원이 제안해준 유용한 배열 내장 메소드를 알게 되었다.

## 개꿀 메소드 **Array.prototype.flatMap()**

![Pull Request](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbWuWUu%2FbtrLLVs33KB%2FklaWO2BioGEQcqEc9i9hM1%2Fimg.png)

*(저도 개꿀 내장 메소드 한번 사용해보겠습니다)*

프론트엔드 개발을 하다보면 공통적으로 느낄것이라 생각하지만 배열이나 객체를 다루는 일이 굉장히 잦다. 그래서 이와 관련된 코멘트들도 각자의 스타일에 맞게 각양각색으로 달리곤 하는데, 최근에 다른 팀원분의 PR 코멘트를 염탐하면서 새롭게 알게 된 Array 내장 메소드인 `flatMap`를 소개해 보려 한다.

우선 MDN 문서의 설명을 보자

> **`flatMap()`** 메서드는 먼저 매핑함수를 사용해 각 엘리먼트에 대해 map 수행 후, 결과를 새로운 배열로 평탄화합니다. 이는 깊이 1의 [flat](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) 이 뒤따르는 [map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 과 동일하지만, `flatMap` 은 아주 유용하며 둘을 하나의 메소드로 병합할 때 조금 더 효율적입니다.
> 

대충 메소드 이름만 봤을 때는 `flat과` `map`의 기능이 같이 있는 녀석인거 같은데 일단 유즈 케이스를 살펴보자

```jsx
let arr1 = [1, 2, 3, 4];

arr1.map(x => [x * 2]);
// [[2], [4], [6], [8]]

arr1.flatMap(x => [x * 2]);
// [2, 4, 6, 8]

// 한 레벨만 평탄화됨
arr1.flatMap(x => [[x * 2]]);
// [[2], [4], [6], [8]]

// flatMap의 유즈케이스를 더 잘보여주는 사례
// 문장의 리스트로부터 단어의 리스트를 생성

let arr1 = ["it's Sunny in", "", "California"];

arr1.map(x=>x.split(" "));
// [["it's","Sunny","in"],[""],["California"]]

arr1.flatMap(x => x.split(" "));
// ["it's","Sunny","in","California"]
```

flatMap은 `map`의 역할을 함과 동시에 `map`을 돌면서 반환받는 요소들의 깊이를 한 레벨씩 평탄화 하는 `flat`의 역할까지 동시에 수행하고 있음을 알 수 있다.

이러한 용도로 쓰는 것도 당연히 (개 꿀) 편리하지만 `flatMap`의 가장 편리한 점은 하기에 서술할 특징이 아닐까 싶다.

## 아이템의 추가나 제거가 가능

사실 저 코멘트를 달아주신 팀원분이 링크를 걸어주신 섹션도 해당 기능에 대한 섹션이었다. 

MDN의 설명을 먼저 보자.

> `flatMap`은 `map`을 하는 과정에서 아이템을 추가하거나 제거하여 **아이템 개수를 바꿀 수 있습니다.** 다른 말로는 각각의 아이템에 대하여 **1:1대응관계 뿐만 아니라 다대다 대응도 가능하다는 것**입니다. 이러한 측면에서 [filter](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)가 하는 역할의 반대역할을 한다고 볼 수 있습니다. 단순히 아무런 변화를 주고 싶지 않을때에는 원소 하나를 가진 배열을 반환할 수도, 아이템을 추가하고 싶을 때는 다-원소 배열을 반환할 수도, 아이템을 제거하고 싶을 때에는 빈 배열을 반환할 수도 있습니다.
> 

`map`메소드의 경우에는 아이템의 개수를 변경할 수 없다. 해당 아이템 각각에 대하여 주어진 함수에 대한 반환값으로 새로운 배열(이전의 배열과 같은 `length`를 가진)을 반환하는 메소드인 것. 

팀원분이 해당 메소드를 제안하게 된 배경이 된 코드를 살펴보자.

```jsx
get selectedProBucketIds() {
    return this.proBucketList.filter((item) => !!item.selected).map((item) => item.id);
}
```

`selectedProBucketIds`은 `proBucketList`배열의 아이템들 중 `selected` 필드의 값이 `true`인 것을 먼저 `filter` 메소드(주어진 함수의 테스트를 통과하는 모든 요소를 모아 새로운 배열로 반환)로 골라내고, 이 반환된 아이템 배열에 다시 `map`메서드를 사용하여 아이템들의 `id`값만 추출한 새로운 배열을 반환하는 `getter`이다. 

나도 이런 결과값이 필요할 땐 항상 `filter`와 `map`의 조합을 사용했던 것 같은데, `flatMap`을 사용하면 아래와 같이 한번에 처리가 가능해진다.

```jsx
get selectedProBucketIds() {
    return this.proBucketList.flatMap((item) => (item.selected ? [item.id] : []));
}
```
*개이득*

위와 같이 조건에 맞지 않으면 제거(`[]`로 리턴)하는 것 말고도, 아이템을 추가할 수도 있다. MDN예제를 보자.

```jsx
// 다음은 음수는 제거하고 홀수는 1과 짝수로 분리하는 예시입니다.
const a = [5, 4, -3, 20, 17, -33, -4, 18]

a.flatMap((n) =>
    // 요소의 값이 음수이면 제거
    (n < 0) ?      [] :
    // 요소의 값이 양수이면서 짝수이면 그대로 return, 홀수이면 값에서 1을 빼주고 그 뒤에 1을 추가
    (n % 2 == 0) ? [n] : [n-1, 1]
)

// expected output: [4, 1, 4, 20, 16, 1, 18]
```

기존 배열에서 아이템을 제거하고 추가하는 것을 모두 하는 과정을 보여준 예시라 할 수 있겠다.

코드리뷰는 많은 것을 얻어 갈 수 있는 과정이다. 건강하게 정착되어 있다면 나의 PR을 통해서도, 혹은 다른 사람의 PR을 통해서도 실무에서 필요한 다양한 스킬이나 새로운 관점들을 얻어갈 수 있다. (기술 면접 때 기억에 남는 코드 리뷰 코멘트가 있냐는 질문을 받은 기억도 있다.)

이번에 소개한 케이스가 복잡하고 깊은 이해가 필요한 리뷰 코멘트는 아니지만,심플하면서도 팀원들의 코드 생산성을 올려줄 수 있는 코멘트라고 생각한다. 개인적으로 이렇게 바로 도움이 되는 코멘트의 경우 내 PR에 달린게 아니더라도 다음 내 코드에 바로바로 적용해보려 하는 편이다. 

역시 개발자에게 코드리뷰란 참 중요한 과정이라는 걸 다시한번 깨달으며 이번 포스팅을 마치고자 한다.

## 참고자료

- [MDN Array.prototype.flatMap() 명세](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap)
