---
layout: post
title: github rest api를 이용해 배포 트리거하기
---

## 필요하게 된 이유

최근 새로운 회사로 이직을 하게 되면서 온보딩 과제로 사내 블로그 구축 업무를 맡게 되었다. 
완전히 새로운 구축은 아니었고, 사용자에게 보여질 블로그 자체의 기본적인 기능은 개발이 완료된 상태였다.  

여기에 현재 회사의 톤앤매너에 맞게 새로 디자인 작업이 된 시안을 펍하는 작업 + 블로그의 관리 기능을 구현하는 것이 나의 과제였다. 여기서 CMS 구현의 주요한 이슈는 블로그 프레임워크(JAM Stack기반의)가 제공하는 CMS를 이용하는 것이 아니라, 회사 자체 백오피스에 붙여야 한다는 요구사항에 있었다. (이와 관련된 여러 이유들이 있었다. 그런 이유들이 이 포스팅의 주제는 아니니 생략한다)

## 구현 전 블로그의 구조
내가 온보딩 과제를 마주한 시점의 블로그는 다음과 같은 구조를 지니고 있었다.
![블로그의 구조](../assets/posts/2022-04-17-01.jpeg)

블로그의 작성자, 포스트는 aws s3에서 개별의 파일로 관리되고 있었고 사용자에게 보여지는 블로그에서는 이 s3의 파일들을 aws s3 api를 이용해 GET정도만 해와서 뿌려주는 형태였다. 또한 s3 파일에 변경이 일어난다고 해도 blog repo의 deploy는 로컬에서 blog repo의 master에 푸쉬를 해주거나 manual 배포 등의 액션만으로 트리거 되고 있는 상태였기 때문에 s3파일의 변경과는 무관한 일이었다. 즉 실질적으로 s3 파일의 변경과 blog 서비스 사이에는 아무런 연결고리가 없는 상태였다. (누군가 수동으로 액션을 해주지 않는 이상)

이 시점에서 우리의 목적에 맞는 블로그의 관리를 위해 필요한 것은 백오피스에 작성자 및 포스트의 관리(aws s3와의 CRUD)구현과 백오피스에서 작성자 및 포스트의 변경점이 생기면 blog repo에 <b>지금 백오피스에서 작성자나 포스트가 변경되었으니 배포가 필요해</b>라고 알려주는 것이었다.

(해당 사내 블로그는 기술 블로그의 성격에 국한되지 않고 사내의 모든 직종의 직원분들이 다양한 주제와 토픽으로 포스팅을 할 공간이었기 때문에 개발자 친화적인 관리는 지양되었고, 사내 직원 누구나 접근 가능한 백오피스에서 포스트 및 작성자의 관리가 가능해야 했다.)


## 구현해야 할 방향성

이 즈음에서 고민이 된 부분은 백오피스에서 어떻게 블로그 레포에 <b>‘지금 작성자나 포스트가 변경되었으니 배포가 필요해!’</b>라고 알려줄 것인가 였다. 이 부분은 이전에 사내 블로그 구축을 담당했던 다른 팀원분의 도움을 받았는데, github에서 제공해주는 rest api가 있었다. (이런게 있는줄은 처음알았다.. 없는 게 없는 github..)

github rest api의 공식 문서를 보니, ****Create a repository dispatch event**** 라는 단락이 있었는데 이것이 내가 원하던 구현 방향성에 대한 스펙이었다.

[https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event](https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event)

이를 활용하면 외부에서 일어나는 활동(나의 경우에는 백오피스에서의 작성자 및 포스트의 CRUD일 것이다)이 특정 깃헙 레포의 액션(***.github.io의 배포를)을 트리거 할 수 있었다. 이렇게 되면 s3 파일에서 authors와 posts의 변경점이 생길때 마다 ***.github.io의 배포가 트리거 되기 때문에 사용자와의 접점에 있는 블로그 서비스가 변경사항을 즉각적으로 반영할 수 있게 된다.

## 404 Bad credentials
그런데 어쩐 이유에서인지 해당 문서의 cURL에 따라
```shell
curl \
  -X POST \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"on-demand-test","client_payload":{"unit":false,"integration":true}}'
```
백오피스에서 api를 콜하니, 자꾸 Bad credentials 404 error가 떨어졌다. 

```json
{
  “message”: “Bad credentials”,
  “documentation_url”: “https://docs.github.com/rest”
}
```

권한 관련 문제라고 생각하고 로컬에 내 github에 대한 정보를 (username, personal access token)등을 주입하는 등의 시도를 해도 계속 404가 떨어졌다. 이리저리 찾아보니 해당 POST 요청 header에 `Authorization`값이 필요했다. 헤더에 이를 함께 보내주지 않아 권한 에러가 떨어졌던 것..

`Authorization: token ${GITHUB_PERSONAL_ACCESS_TOKEN`

여기서 `GITHUB_PERSONAL_ACCESS_TOKEN`은 생성시 `admin:repo_hook, repo, workflow` 권한이 있어야 하고, 배포 트리거를 걸 레포(나의 경우는 ***.github.io의 레포)에 초대된 계정의 access token이어야 한다. 나의 경우에는 개인 블로그가 아니라 사내 블로그였기 때문에, 플랫폼 챕터의 도움을 받아 개발팀이 공용으로 관리할 목적으로 access token을 발급받아 이를 사용하였다.

그리하여 최종적인 백오피스에서의 POST요청은 이렇게 구현하였다.
```shell
// 백오피스에서의 POST콜 
curl \
  -X POST \
  -H "Authorization: token ${GITHUB_ACCESS_TOEKN}" \
  -H "Accept: application/vnd.github.v3+json" \
  https://api.github.com/repos/OWNER/REPO/dispatches \
  -d '{"event_type":"webhook"}'
```

event_type 네이밍은 간단히 ‘webhook’으로 하였고, 이를 배포 트리거가 일어날 blog 레포에서 트리거 가능하도록, 깃헙 액션을 구성하는 yml파일에 아래와 같이 추가해주면 성공적으로 구현이 완료된다. (백오피스에서 보내주는 POST요청의 event_type의 value와 blog 레포에서 repository_dispatch로 받는 types의 네이밍이 일치해야 한다.) 

```yaml
// blog 레포
name: CI

on:
  repository_dispatch:
    types:
      - webhook
```

## 최종 구현 흐름
![최종 구현 흐름](../assets/posts/2022-04-17-02.jpeg)

결론적으로 구현한 내용은 이렇다.

1. 백오피스에서 블로그의 작성자나, 포스팅을 생성, 삭제, 편집한다.
2. aws s3 API를 이용해 s3의 파일을 CRUD한다. 동시에 github rest API를 이용해 github blog 레포에 배포를 트리거한다.
3. 배포가 완료되면 변경사항이 사용자와의 접점에 있는 블로그 서비스 페이지에 정상적으로 반영된다.

## 끝으로
이러한 흐름으로 블로그를 구현하는 경우는 일반적인 케이스는 아니라고 생각한다. 이런 흐름에는 나름의 여러가지 사정들이 있었으나 결론적으로 방법을 찾게 되었고, 이전엔 활용해보지 못한 새로운 방법론을 알게 되어 어찌보면 유익한 경험이라는 생각이 든다.

나와 같은 케이스가 아니라더라도 외부에서의 활동을 감지하여 특정 레포의 배포 트리거를 일으키는 것은 다양한 니즈에 맞는 해결책으로써 활용이 가능할 듯 하다.

## 참고목록
- [https://docs.github.com/en/rest](https://docs.github.com/en/rest)
- [https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event](https://docs.github.com/en/rest/reference/repos#create-a-repository-dispatch-event)
- [https://velog.io/@dahunyoo/Github-Action의-Triggering을-이용한-간단한-CI-구성해보기](https://velog.io/@dahunyoo/Github-Action%EC%9D%98-Triggering%EC%9D%84-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-CI-%EA%B5%AC%EC%84%B1%ED%95%B4%EB%B3%B4%EA%B8%B0)