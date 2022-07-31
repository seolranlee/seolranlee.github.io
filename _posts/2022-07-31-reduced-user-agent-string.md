---
layout: post
title: Reduced User-Agent string
---

내가 현재 속해 있는 프론트엔드 챕터에서는 매주 한 명씩 돌아가면서 챕터원 모두와 함께 나눌만한 주제로 기술 세미나를 진행하고 있다. 마침 얼마 전 나의 차례가 되어서 Reduced User-Agent string라는 주제로 발표를 하였고 나의 개인 블로그에 해당 발표자료를 한 차례 더 정리하는 과정을 거쳐 공유해 보고자 한다.

최근 크롬 100버전이 릴리즈 되면서 (작성일 기준 버전은 103.0.5060.114) 크롬 블로그에 아래와 같은 내용이 기제되었다.

## [Chrome100 - 축소되지 않은 UA 문자열을 지원하는 마지막 버전](https://developer.chrome.com/blog/new-in-chrome-100/?fbclid=IwAR1bnxqGSjwymXCsP_P9MeWL9c8KYaMV9u882kh3-c9A46D0JZ0xn4cpKVQ#reduced-ua)


### Reduced User-Agent string

> Speaking of the user agent, Chrome 100 will be the last version to support an unreduced User-Agent string by default. This is part of a strategy to replace use of the User-Agent string, with the new User-Agent Client Hints API.<br/><br/>Starting in Chrome 101, the user agent will be gradually reduced.<br/><br/>Check out User Agent Reduction Origin Trial and Dates on the [Chromium blog][crblog], to learn more about what will be removed, and when.

> 사용자 에이전트에 대해 말하자면, Chrome 100은 기본적으로 축소되지 않은 사용자 에이전트 문자열을 지원하는 마지막 버전이 될 것이다. 이는 사용자-에이전트 문자열 사용을 새로운 사용자-에이전트 클라이언트 힌트 API로 대체하기 위한 전략의 일부입니다.<br/><br/>크롬101부터는 사용자 에이전트가 점차 축소된다.<br/><br/>제거 대상과 시기에 대한 자세한 내용은 [Chromium 블로그][crblog]에서 User Agent Reduction Origin Trial and Dates를 참조하십시오.


요약하자면 과도하게 많아진 문자열 종류들, 개인 식별 정보가 너무 많다는 이유 등등([UA가 이렇게 복잡해진 다양한 사정들이 잘 설명되어 있는 블로그 포스팅: 브라우저의 사용자 에이전트는 왜 이렇게 복잡하게 생겼을까?](https://wormwlrm.github.io/2021/10/11/Why-User-Agent-string-is-so-complex.html))으로 폐기하고 클라이언트 힌트란 형태로 소프트 체인지 중이며 크롬 100버전은 축소되지 않는 UA 문자열을 지원하는 마지막 버전이 될것이다. 라는 내용이다.

## 그러면 개발자들은 왜 UA 문자열을 사용할까

User-Agent 문자열은 많은 합법적인 [사용 사례](https://github.com/WICG/ua-client-hints/blob/main/README.md#use-cases)를 가능하게 하며 개발자와 사이트 소유자에게 중요한 목적을 제공한다.

개발자의 관점에서 UA 문자열에서 사용할 수 있는 세부 정보는 가치가 있으며 삭제에 대해 합리적으로 반대합니다. Safari의 UA 문자열 고정에 대한 피드백은 다음과 같이 [4가지 광범위한 사용 범주](https://github.com/WICG/ua-client-hints/blob/main/README.md#challenges)를 언급했다.

해당 문서에 따르면 이 네가지의 광범위한 사용 범주는 아래와 같다.

1. 버전에 따른 동작 차이
브라우저의 브랜드 및 버전 정보(예: "Chrome 69")를 사용하면 웹사이트에서 감지할 수 없는 특정 릴리스의 알려진 버그를 해결할 수 있다.

2. 사용자 에이전트에 따라 보여줄 콘텐츠 협상
개발자는 종종 사용자 에이전트와 플랫폼을 기반으로 보낼 콘텐츠를 협상한다. 예를 들어 일부 애플리케이션 프레임워크는 각 플랫폼의 미학과 디자인 패턴을 일치시키기 위해 iOS의 애플리케이션을 Android의 동일한 애플리케이션과 다르게 스타일을 지정한다.

3. 특정 버전의 버그
1과 유사하게, OS 개정판과 아키텍처는 웹사이트의 코드에서 해결할 수 있는 특정 버그를 담당할 수 있으며, 다운로드를 위해 적절한 실행 파일을 선택하는 것과 같이 좁은 방식으로써 유용할 수 있다.

4. OS의 동작 차이
정교한 개발자는 모델/제조업체를 사용하여 사이트를 장치의 기능(예: [Facebook Year Class](https://engineering.fb.com/2014/11/06/android/year-class-a-classification-system-for-android/))에 맞게 조정하고 때때로 모델/제조업체에 특정한 성능 버그 및 회귀를 찾아낼 수 있다.

크게 4가지의 광범위한 사용 범주에 UA문자열이 담고 있는 정보들은 개발자들이 유저에게 적절한 서비스 경험을 제공해주기 위해 합리적인 관점에서 필요하다.

## 그렇다면 필요한데 왜 축소시킬까

그렇다면 UA는 개발자의 관점에서 합리적인 이유로 필요한 스펙인데 구글에서 축소시키려는 시도를 하는 이유는 무엇일까? 아래의 유투브 영상에 이에 대한 답을 잘 설명해주고 있다.

[🎥 Just the data you need](https://www.youtube.com/watch?v=f0YY0o2OAKA&t=34s)

요는, 개발자는 ***모든 데이터를 수집하기만 하면 나중에 가치가 있을 수 있을 것*** 이라는 생각에 빠질 유혹이 있으나 그것은 올바른 태도가 아니며, 해당 데이터가 유출될 경우 더 많은 규정을 준수해야 하며 더 많은 위험과 더 많은 해야할 일거리들을 낳을 수 있다.

또한 축소되기 이전의 User-Agent string에는 많은 **high entropy**`(여기서 말하는 엔트로피는 정보의 엔트로피를 뜻하며, 높은 엔트로피란 OS 관련 정보나 Browser 상세 정보 등 유저를 특정 지을 수 있는 정보들을 의미한다.)`들이 담겨 있고, 이로 인해 개인정보 침해 문제가 있을 수 있다. 구글은 크롬은 개인 정보 보호를 위한 다양한 일감들을 진행하고 있는데 이를 [샌드박스 프로젝트](https://www.chromium.org/Home/chromium-privacy/privacy-sandbox/)로 명칭하고 있으며 이 Reduced User-Agent string도 이 프로젝트 일환이다.

또한 현재의 UA 문자열은 구조화된 데이터가 아니므로(문자열의 나열일뿐) 이를 구문 분석하기 위한 불필요한 복잡성도 늘어난다.(지나치게 길고 복잡한 UA 문자열에서 우리들이 원하는 정보만을 얻기 위해서는 정규식 등의 코드를 구현해야 한다.)

## 축소된 UA

### 축소된 navigator.userAgent

Chrome 93에서 `about://flags/#reduce-user-agent` 플래그를 활성화하여 축소된 UA 버전을 테스트할 수 있다.(참고: 이 플래그는 Chrome 84 -92 버전에서 `about://flags/#freeze-user-agent`로 명명됨)

그러나 Chrome 100버전 이상에서는 별도로 flags를 조정하지 않아도 이미 점진적으로 축소가 진행중에 있다. (현재 해당 포스팅 작성일 기준 크롬 버전 `103.0.5060.114` 을 사용하고 있는데, 이미 크롬 major버전을 제외한 버전 정보는 UA문자열에서 표현되고 있지 않다.)

1. Chrome 브라우저의 버전이 메이저 버전만 제공

|기존|변경 후|
|------|------|
|Chrome/`<majorVersion>.<minorVersion>`<br/>ex) Chrome/84.`0.4076.0`|Chrome/`<majorVersion>.0.0.0`<br/>ex) Chrome/84.`0.0.0`|

2. Chrome 데스크탑에서 고정된 플랫폼 문자열만 제공

제공되는 플랫폼
- Windows NT 10.0; Win64; x64
- Macintosh; Intel Mac OS X 10_15_7
- X11; Linux x86_64
- X11; CrOS x86_64

|기존|변경 후|
|------|------|
|`<platform>; <oscpu>`<br>ex) Windows NT `5.0`; Win64; x64<br/>ex) Windows NT `6.3`; Win64; x64|`<unifiedPlatform>`<br/>ex)Windows NT `10.0`; Win64; x64<br/>ex) Windows NT `10.0`; Win64; x64|

3. Chrome 모바일 및 테블릿에서 안드로이드 버전 및 디바이스 정보가 고정

|기존|변경 후|
|------|------|
|Android `<androidVersion>; <deviceModel>`<br/>ex) Android `9; SM-A205U`|Android `10; K`<br/>ex) Android `10; K`|


### 축소된 navigator.platform

고정된 문자열 제공 예정

|Platform|Reduced value|
|------|------|
|macOS|MacIntel|
|Windows|Win32|
|Chrome OS|Linux x86_64|
|Linux|Linux x86_64|
|Android|Linux armv81|

💡 `navigator.platform`은 삭제 예정 속성으로, `navigator.userAgentData.platform`으로 대체 필요.

## 그렇다면 어떤 방식으로 전환될까

### [User-Agent Client Hints](https://web.dev/user-agent-client-hints/#javascript-api)

앞서 말한것과 같이 축소되기 이전의 UA문자열은 유저의 개인정보를 침해할 가능성이 있고 현재의 형태가 바람직하지 못하다 할지라도, 개발자들은 합리적인 관점에서 축소되기 이전의 UA문자열이 제공해주는 정보들을 활용해야 할 일이 생길 수 있다. 

그렇다면 무작정 축소를 진행할 것이 아니라 축소되면서 표현이 생략된 정보들을 알 수 있는 현재보다 더 나은 대체 방안이 제시되어야 할 텐데 그 방법으로 구글 크롬이 내놓은 방법이 바로 User-Agent Client Hints이다. 

> 클라이언트 힌트를 사용하여 개발자는 사용자 에이전트(UA) 문자열에서 구문 분석할 필요 없이 사용자의 장치 또는 조건에 대한 구조화된 정보를 능동적으로 요청할 수 있습니다. 이 대체 경로를 제공하는 것은 결국 사용자 에이전트의 문자열에서 유저를 특정 지을 수 있는 정보를 줄이는 첫 번째 단계입니다.

이를 내가 이해한 내용으로 풀어서 전달해보자면, 크롬은 축소된 UA문자열을 통해 사용자를 특정 지을 확률이 높은 정보들은 점진적으로 생략해 나갈 것이며, 개발자가 부득이하게 생략된 정보들 중 필요로 하는 정보가 있다면 명시적으로 해당 정보를 요청할 수 있다는 것이다. 

이전에는 비교적 `일반적인 정보 + 사용자를 특정할 수 있는 사용자들간의 유사성이 낮은 정보`들이 모두 합쳐져 UA문자열에 포함되어 내려왔다고 한다면 축소된 UA문자열에는 `일반적이고 사용자들간의 유사성이 높은 정보들만` 내려주고, 그 외의 생략된 정보들이 필요하면 그때 능동적으로 요청해라. 라는 방향으로 바뀌었다고 할 수 있겠다.

### User-Agent Data

그리고 구글 크롬은 문자열의 나열일 뿐인 현재의 User-Agent string을 세분화해 Object 형식으로 나타낸 User-Agent Data를 사용할 수 있도록 제공하고 있다. [인터페이스는 다음과 같다.](https://github.com/lukewarlow/user-agent-data-types/blob/master/index.d.ts)

```typescript
// WICG Spec: https://wicg.github.io/ua-client-hints

declare interface Navigator extends NavigatorUA {}
declare interface WorkerNavigator extends NavigatorUA {}

// https://wicg.github.io/ua-client-hints/#navigatorua
declare interface NavigatorUA {
    readonly userAgentData?: NavigatorUAData;
}

// https://wicg.github.io/ua-client-hints/#dictdef-navigatoruabrandversion
interface NavigatorUABrandVersion {
    readonly brand: string;
    readonly version: string;
}

// https://wicg.github.io/ua-client-hints/#dictdef-uadatavalues
interface UADataValues {
    readonly brands?: NavigatorUABrandVersion[];
    readonly mobile?: boolean;
    readonly platform?: string;
    readonly architecture?: string;
    readonly bitness?: string;
    readonly model?: string;
    readonly platformVersion?: string;
    /** @deprecated in favour of fullVersionList */
    readonly uaFullVersion?: string;
    readonly fullVersionList?: NavigatorUABrandVersion[];
    readonly wow64?: boolean;
}

// https://wicg.github.io/ua-client-hints/#dictdef-ualowentropyjson
interface UALowEntropyJSON {
    readonly brands: NavigatorUABrandVersion[];
    readonly mobile: boolean;
    readonly platform: string;
}

// https://wicg.github.io/ua-client-hints/#navigatoruadata
interface NavigatorUAData extends UALowEntropyJSON {
    getHighEntropyValues(hints: string[]): Promise<UADataValues>;
    toJSON(): UALowEntropyJSON;
}
```

```typescript
NavigatorUAData
{
    "brands": [
        {
            "brand": ".Not/A)Brand",
            "version": "99"
        },
        {
            "brand": "Google Chrome",
            "version": "103"
        },
        {
            "brand": "Chromium",
            "version": "103"
        }
    ],
    "mobile": false,
    "platform": "macOS"
		[[Prototype]]: NavigatorUAData
			getHighEntropyValues: ƒ getHighEntropyValues()
```

- `brands`는 사용자 에이전트의 브라우저의 이름과 메이저 버전이 담긴 배열 값이다.
- `mobile`은 사용자 에이전트의 기기가 모바일인지를 나타내는 값이다.
- `platform`은 `navigator.platform`을 대체할 속성이다.
- [`getHighEntropyValues`](https://developer.mozilla.org/en-US/docs/Web/API/NavigatorUAData/getHighEntropyValues) 메서드는 높은 엔트로피에 해당하는 값을 가져온다.

`brands`(브라우저 이름과 메이저 버전)와 `mobile`은 낮은 엔트로피로 분류되기 때문에 동기 방식으로 값을 가져올 수 있지만 그 외 정보는 높은 엔트로피로 분류되어 `getHighEntropyValues` 라는 비동기 메서드를 통해 가져와야 한다. 

또한 User-Agent string처럼 모든 정보를 가져오는 게 아니라 **자신에게 필요한 정보**만 가져올 수 있다.

```javascript
navigator.userAgentData.getHighEntropyValues([  
	"architecture",
	"bitness",
	"model",
	"platformVersion",
	"uaFullVersion",
	"fullVersionList"
]).then(info => {
	console.log(info);
});
```

### 동기방식으로 확인 가능한 정보(Low Entropy)

- `navigator.userAgentData.brands`
- `navigator.userAgentData.mobile`
- `navigator.userAgentData.platform`


### 비동기방식으로만 확인 가능한 정보(High Entropy)

- macOS, Windows, Android를 제외한 OS 정보
- 특정 OS의 풀 버전
- 브라우저의 풀 버전

### 마그레이션이 필요한 케이스

- 비동기 방식으로 전환된 정보를 동기방식으로 활용하고 있는 케이스
  + macOS, Windows, Android를 제외한 OS 정보
  + 특정 OS의 풀 버전
  + 브라우저의 풀 버전

## [이전까지의 릴리즈 사항들과 축소 계획](https://blog.chromium.org/2021/09/user-agent-reduction-origin-trial-and-dates.html)

### 축소 준비

### 1단계: Chrome 92 이후(2021년 7월 20일)

CTA: 사이트 사용을 감사하여 마이그레이션이 필요한 위치를 파악합니다.

M92부터는 DevTools에서 navigator.userAgent, navigator.appVersion 및 navigator.platform에 액세스하는 경우 경고합니다.

### 2단계: Chrome 95에서 Chrome 100으로

CTA: 사이트에 대한 오리진 평가판에 등록하고 Chrome 101이 출시될 때까지 피드백을 제공하십시오.

사이트에 대한 원본 평가판을 시작하여 테스트 및 피드백을 위해 최소 6개월 동안 최종 축소된 UA 문자열을 선택합니다.

원본 시험 파트너와 커뮤니티의 피드백을 평가 하고 이 피드백을 기반으로 계획의 3~7단계로 진행하여 생태계가 그들 사이에 적응할 수 있는 적절한 시간을 제공합니다. 그렇지 않으면 피드백에 따라 최선의 조치를 재고할 것입니다.

### 축소 롤아웃

### 3단계: Chrome 100

**CTA**: 필요한 경우 사이트에 대한 지원 중단 평가판 또는 엔터프라이즈 정책에 등록합니다.

사이트 마이그레이션에 더 많은 시간이 필요할 수 있는 경우 사용 중단 평가판 및 엔터프라이즈 정책을 시작합니다.

### 4단계: Chrome 101

**CTA**: 사이트가 축소된 Chrome 버전 번호와 호환되는지 확인하고 그렇지 않은 경우 UA 클라이언트 힌트로 마이그레이션합니다 .

축소된 Chrome MINOR.BUILD.PATCH 버전 번호('0.0.0')를 배송합니다. 일단 롤아웃되면 감소된 UA 문자열은 사용 중단 평가판을 선택하지 않은 사이트에 대한 데스크톱 및 모바일 운영 체제의 모든 페이지 로드에 적용됩니다.

### 5단계: Chrome 107

**CTA**: 귀하의 사이트가 축소된 Desktop UA 문자열 및 관련 JS API와 호환되는지 확인하고 그렇지 않은 경우 UA 클라이언트 힌트로 마이그레이션하십시오 .

축소된 Desktop UA 문자열 및 관련 JS API( navigator.userAgent , navigator.appVersion , navigator.platform )의 출시를 시작 합니다. 롤아웃되면 축소된 UA 문자열이 지원 중단 평가판을 선택하지 않은 사이트의 데스크톱 운영 체제 에서 모든 페이지 로드에 적용됩니다 .

### 6단계: Chrome 110

**CTA**: 귀하의 사이트가 축소된 모바일 UA 문자열 및 관련 JS API와 호환되는지 확인하고 그렇지 않은 경우 UA 클라이언트 힌트로 마이그레이션하십시오 .

축소된 Android 모바일(및 태블릿) UA 문자열 및 관련 JS API의 출시를 시작합니다. 롤아웃되면 축소된 UA 문자열이 지원 중단 평가판을 선택하지 않은 Android의 모든 페이지 로드에 적용됩니다 .

### 축소 완료

### 7단계: Chrome 113

지원 중단 평가판이 종료되고 모든 페이지 로드는 축소된 UA 문자열 및 관련 JS API를 수신합니다.

각 단계에서 자세한 내용과 예제 사용자 에이전트 문자열 은 함께 제공되는 축소 사용자 에이전트 문자열 업데이트 페이지 를 참조하세요. 이 페이지에 중대한 지연이나 변경 사항이 있을 경우에도 알려드립니다.

## 마치며

사실 UA 스트링은 레진에 있을때에도, 현재 숨고에서도 `isMobile`이나 `device type`을 detect하여 분기쳐서 적절한 로직을 실행해야 할때 자주 사용하곤 하였다. 그러나 사실 이미 기존 서비스 코드에 공통 util로 잘 만들어진 혹은 라이브러리로 설치된 것들을 위주로 가져다 썼기 때문에 UA string 자체가 실제로 어떻게 생겼고 그 문자열이 어떤 정보들을 담고 있으며, 그 정보들의 복잡도가 그렇게 높은 이유까지는 상세히 알지 못하였다. 이번 발표를 준비하면서 그런 일련의 이유들과 정보들을 알 수 있어서 좋은 기회가 되었던 것 같다.

또한 이런 발표가 주는 유익함은 나의 발표 후에 다른 동료들이 던져주는 유의미한 질문들인 것 같다. 어떤 동료분은 해당 발표를 들으시고 기존에 UA문자열이 내려오는 것이랑 `getHighEntropyValues`라는 메소드를 통해 호출하는 것과의 차이를 물어보셨는데 (어쨌든 원한다면 모든 정보를 기존과 다름 없이 다 얻을 수 있는것인데 뭐가 다를까?가 질문의 요점이셨던거 같다) 발표후에도 생각할만한 거리를 주신 좋은 질문이었던 것 같다.

내 나름대로 생각해보자면 크롬의 이런 시도의 주 목적은 대체하는 방법에 있다기보다 축소되는 UA문자열 그 자체라고 생각한다. 서비스를 이용하는 유저들이 굳이 개발자도 필요하지 않은 정보까지 담겨있는 UA문자열을 사이트의 모든 요청때마다 보내는 것이 아닌, 보안의 위협을 비교적 낮게 가질 수 있는 범용적인 정보들의 구성으로만 이루어진 문자열로써 축소 시키고 해당 서비스의 개발자가 정말로 굳이 그런 정보가 필요할 때만 요청하라. 그 때엔 그 정보들을 요청하면 받을 수 있도록 우리가 대안적인 스펙은 마련해주겠다. 의 취지가 아닐지 개인적으로 생각해본다.

## 레퍼런스

- [https://github.com/WICG/ua-client-hints/blob/main/README.md](https://github.com/WICG/ua-client-hints/blob/main/README.md)
- [https://dc2348.tistory.com/59?category=903868](https://dc2348.tistory.com/59?category=903868)
- [https://d2.naver.com/helloworld/6532276](https://d2.naver.com/helloworld/6532276)
- [https://blog.chromium.org/2021/09/user-agent-reduction-origin-trial-and-dates.html](https://blog.chromium.org/2021/09/user-agent-reduction-origin-trial-and-dates.html)