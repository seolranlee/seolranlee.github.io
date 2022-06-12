---
layout: post
title: github readme 페이지에 유의미한 데이터 노출하기
---

## 필요하게 된 이유

나의 전 직장에서 팀장님으로 계셨던 분께서는 감사하게도 종종 개발 관련 좋은 정보가 있으면 메신저를 통해 공유를 해주시곤 하신다.
최근에도 여러 정보를 공유해주시다가 github readme페이지를 꾸미는 몇가지 방법들을 전달해주셨는데 readme 페이지를 별도로 구현하는 필요성에 대해 팀장님과 짤막하게 이야기를 나눈 기회가 있었다. 나는 개인적으로 github readme페이지를 별도로 구현해놓지는 않고 있던 상태였다. (딱히 필요성을 느끼지 못했다고 해야 맞을 것이다.)

그런데 막상 이 필요성에 대해 생각해보니, 나의 github 페이지 메인에 나의 코딩 활동에 대한 아무런 정보가 없고 단지 저장소들만 나열되어있는 상태라면, 타인이 나의 github 페이지 엔트리를 통해 내가 어떤 활동을 하고 있는지, 어떤 언어를 주력으로 개발하고 있는 개발자인지 단번에 알기 힘들겠다는 생각이 들었다.

그리하여 이번 기회를 통해 팀장님이 보내주신 자료와 몇가지 서치를 통해 나만의 github readme를 구현하게 되었다.

## 제한된 영역 내에서 최대한 유의미한 데이터 노출하기

이번에 개선을 진행한 [나의 github readme](https://github.com/seolranlee/seolranlee/blob/main/README.md)기반으로 게시글을 작성해 보도록 하겠다.

나는 github에는 애초에 readme라는 페이지 자체가 없었기 때문에 맨 처음으로 나의 github계정에 노출될 readme.md파일을 생성하는 과정이 필요했다.

### 1. 나의 github 계정명과 동일한 레포지토리 생성

<p align="center">
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fn9iPO%2FbtrEywftELI%2FSKONWrYhlpVI8VdelVbTYk%2Fimg.png" alt="create a new repository">
</p>

먼저, [나의 github](https://github.com/seolranlee) 계정명(seolranlee)과 동일한 이름의 레포지토리를 생성한다.

<p align="center">
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2Fdqzd8J%2FbtrEwjgVDVo%2Floof7zT0NiavSTvl4mMqX1%2Fimg.png" alt="check Add a README file">
</p>

생성시 레포의 초기 설정 중 ```Add a README file```에 체크 하여 레포 생성 시 README.md파일이 자동 생성될 수 있도록 해둔다. (물론 해당 레포를 클론 받은 후 README.md파일을 추가해 준 뒤에 push해줘도 되지만 귀찮으니까..)

<p align="center">
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2F7r98D%2FbtrEBrLinkP%2FtJco7JVT6aH1RBHBe2UL60%2Fimg.png" alt="check Add a README file">
</p>

***짜잔. 나의 github 메인에 노출될 README.md파일 생성이 완료되었습니다.***

이제 해당 파일의 내용은 나의 github 엔트리 화면에 노출될 화면이 된다.

### 2. GitHub Readme Stats

gtihub readme를 꾸미는 방법에는 여러가지가 있는데, 그 중 나는 내가 활동한 코딩 데이터와 실질적인 연관이 있는 GitHub Readme Stats라는 통계 데이터를 삽입하기로 결정하였다.

[GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats/blob/master/docs/readme_kr.md) 해당 통계 데이터의 사용법은 한국어로도 번역되어 있고 사용법 자체도 상당히 간단하여 이 문서만으로도 이해하는데 무리는 없을것이라 예상되지만, 나의 github README.md를 바탕으로 몇가지 소개를 하고자 한다.

#### GitHub 통계

##### 사용법

***엔드 포인트: ```api?username=${username}```***


- 코드

```![Seolranlee's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee)```

- 실제 표현되는 화면

![Seolranlee's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee)

##### 총 커밋 수에 비공개 기여도 (private contribs) 수 추가하기

```?count_private=true``` 속성을 추가하면, 모든 비공개 기여도까지 반영된다. (프라이빗 레포에서 한 활동이 모두 통계에 추가됨)

- 코드

```![Seolranlee's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee&count_private=true)```

- 실제 표현되는 화면

![Seolranlee's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee&count_private=true)

대부분의 개발자들이 그러할 것이라고 예상하는데, 대부분의 개발자들이 그러할 것이라고 생각하지만 나의 경우 나의 대부분의 코딩 활동은 회사 서비스를 타겟으로 이루어지는데, 거의 대부분의 회사 서비스 레포들은 보안상의 이유로 비공개 레포에 기반을 두고 있다. 이런 이유로 인해 나의 활동들은 거의 모두 프라이빗한 레포 위에서 이루어지고 있기에 해당 파라미터를 추가함으로써 Total Commits (2022)의 갯수가 42개에서 576개까지 오르게 되었다. 

당연히 비공개 레포 위에서 이루어지는 활동인지라 디테일한 통계까지 가져오진 못하지만, 커밋수 정도만 노출 시키는 것만으로도 그래도 꾸준히 코딩을 하고 있는 사람이라는 인상을 줄 수 있지 않을까(...사실 이 기회로 퍼블릭한 코딩 활동을 조금 더 적극적으로 해야겠다는 생각이 들었다.)

##### 아이콘 표시하기

아이콘 항목을 활성화 하기 위해선, 다음과 같이 ```show_icons=true``` 속성을 추가해주면 된다.

- 코드

```![Seolranlee's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee&show_icons=true)```

- 실제 표현되는 화면

![Seolranlee's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee&show_icons=true)


#### 언어 사용량 통계

##### 사용법

***엔드 포인트: ```api/top-langs?username=${username}```***

- 코드

```![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee)```


- 실제 표현되는 화면

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee)

##### 통계에서 제외할 저장소 지정하기

```?exclude_repo=${repo1},${repo2}``` 속성을 통해 특정 저장소를 제외할 수 있다. (```,```로 여러개의 저장소를 구분시켜준다.)

비교적 옛날 호랑이 담배피던 시절에 생성했던 저장소 몇개를 제외시켜보았다. ([board](https://github.com/seolranlee/board), [study](https://github.com/seolranlee/study), [lesson](https://github.com/seolranlee/lesson))

- 코드

```![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&exclude_repo=board,study,lesson)```

- 실제 표현되는 화면

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&exclude_repo=board,study,lesson)

옛날 옛적 저장소를 제외시키니 해당 저장소에서 사용한 언어들이 분량이 통계데이터에서 차감된 것을 볼 수 있다.(HTML 57.63% => 10.06%, PHP는 아예 제거..)

##### 통계에서 특정 언어 제외하기

```?hide=${language1},${language2}``` 속성을 통해 특정 언어를 제외할 수 있다.

- 코드

```![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&hide=html,php)```

- 실제 표현되는 화면

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&hide=html,php)

나의 경우에는 현재는 무용한 옛날 저장소들을 일일이 찾아내서 ```exclude_repo```의 값으로 집어넣는 것 보다 언어를 제외시키는 것이 더 효율적이라고 생각되어 ```html```, ```php```를 제외하는 방향으로 구현하였다.

##### 컴택트한 카드 레이아웃 설정하기

```&layout=compact``` 속성을 통해 카드의 디자인을 변경할 수 있다.


- 코드

```![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&layout=compact)```

- 실제 표현되는 화면

![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&layout=compact)


#### 테마 적용

내장 테마를 사용하면, 별도의 커스터마이징 없이 GitHub 통계 카드를 꾸밀 수 있다. ```?theme=${THEME_NAME}``` 속성을 이용하여 테마를 지정한다.
[사용 가능한 모든 테마](https://github.com/anuraghazra/github-readme-stats/blob/master/themes/README.md)

- 코드

```markdown
![Seolran's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee&count_private=true&show_icons=true&theme=onedark)
![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&theme=onedark)
```

- 실제 표현되는 화면

![Seolran's GitHub stats](https://github-readme-stats.vercel.app/api?username=seolranlee&theme=onedark)
![Top Langs](https://github-readme-stats.vercel.app/api/top-langs/?username=seolranlee&theme=onedark)

나의 경우는 ```onedark```테마를 선택하여 구현하였다.

#### hits counter 달기

https://hits.seeyoufarm.com/ 에서 카운터의 색상 및 스타일 등을 지정한 뒤 생성되는 코드를 README.md문서에 붙여넣으면 된다. 


## 완성

<p align="center">
  <img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FrypFp%2FbtrEE1Zub4X%2F03n3ziDmFk0HkoEiqBLs5k%2Fimg.png" alt="my README.md">
</p>

인사말 뒤에 간략하게 나에 대한 소개를 넣었고 (나의 github에 접근하는 대부분의 사람들은 한국어를 기반으로 하고 있을것이라 예상되기 때문에 굳이 영어로 작성하진 않았다.)
그 뒤에 [GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats/blob/master/docs/readme_kr.md)에서 제공하는 api를 활용하여 나의 Github 통계, 그리고 언어 사용량 통계를 노출하였다. 

해당 포스팅에 작성한 것들 이외에도 다양한 커스터마이징 방법이 공식 문서에 소개되어 있으니 조금 더 다양한 방식으로의 확장 및 변형을 원하시는 분들이 있다면 참고하면 좋을 듯 하다.
([커스터마이징](https://github.com/anuraghazra/github-readme-stats/blob/master/docs/readme_kr.md#%EC%BB%A4%EC%8A%A4%ED%84%B0%EB%A7%88%EC%9D%B4%EC%A7%95) 섹션에서 조금 더 자세히 확인이 가능하다.)

## 참고목록
- [GitHub Readme Stats](https://github.com/anuraghazra/github-readme-stats/blob/master/docs/readme_kr.md)
- [Github Readme 꾸미기 총정리](https://velog.io/@seondal/Github-Readme-%EA%BE%B8%EB%AF%B8%EA%B8%B0-%EC%B4%9D%EC%A0%95%EB%A6%AC)