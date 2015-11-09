---
layout:     post
title:      Isomorphic과 Universal javascript에 대한 단상
date:       2015-11-09
summary:    Isomorphic, Universal에 대해 들어보셨나요?
categories: dev
---

최근 react와 flux를 공부하면서 이전에는 몰랐었던 개념에 대해 알게 되었다.  
Isomorphic과 Universal.

Javascript로 백엔드에서 멋지게 그리고 빠르게 처리하는 node.js는 이미 다들 익숙히 들어봤을 것이다.  javascript를 사용하는 개발자에겐 따로 java, c계열, php, python, ruby 등이 아닌 익숙한 언어를 사용해 서버를 구축한다는 매력에 빠질 수 밖에 없는 언어. 하지만 node.js의 콜백 지옥에 빠진 경험은 정말 괴로웠었다.. 쿨럭

여하튼 최근 다시 node.js를 손대면서, 아니 정확히는 react.js를 접하고 찬양하기 시작하면서 알게 된 용어가 있다.  Isomorphic과 Universal. 도대체 이녀석은 무엇일까.

## Isomorphic

![isomorphic](https://cloud.githubusercontent.com/assets/92839/8012112/8fa3a59e-0b73-11e5-887e-98ae8dcb5705.png)  
*이서모픽*

server 와 client 에서 똑같이 실행.
좋은 예로는 jQuery를 들 수 있겠다. client side의 jQuery와 server side node.js에서 jQuery를 import 해 사용할때 동일한 API를 제공한다. 
오오~ 완전히 똑같진 않지만 백엔드와 프론트엔드에서 똑같은 API를 사용하고 같은 결과를 기대할 수 있다.  

React를 열심히 공부하고 있으면서도 가장 많이 느끼는 것이 이 부분이다.   
node.js에서 사용하는 react 컴포넌트는 프론트엔드에서 그대로 재사용을 할 수 있다는 것! 정말 멋진 부분이 아닐 수 없다.

## Universal 

![universal](https://cloud.githubusercontent.com/assets/92839/8012117/983bca24-0b73-11e5-8c8b-0487711d5112.png)  
*유니버설 ( 좀 더 정확히는 유너버설 )*

Javascript가 서버와 브라우저 뿐만 아니라 각종 디바이스 등에서도 동작하기 때문에 Isomorphic 보다는 Universal 이라고 부르자는 움직임도 있다.  (https://github.com/facebook/react/pull/4041)[https://github.com/facebook/react/pull/4041]


자, 그럼 좀 더 디테일 한 상황을 보자.

몇 년 전 모바일 전용 sencha 프레임웍을 할 때 가장 불만이었던 것은 Front-end 마크업 부분에 달랑 `<div id="app"></div>` 정도 밖에 남지 않는 것이었다.  
프론트엔드 개발로 밥벌이를 하고 있는 상황에서는 SEO를 전혀 맞출 수 없는 것이 여간 불만이 아닐 수 없었다.

최근 react를 node.js 에서 사용하면서도 처음엔 이런 불만이 있었다. 아니 정확히는 프론트에서도 react 컴포넌트를  사용할때에도 비어있는 HTML이 불만이었다. ( HTML을 사랑한다 )  

그러다가 isomorphic, universal에 대해 알게 되고 나서는 이런것들을 해결 할 수 있었다.

## SEO 구현

Javascript로 프론트와 서버에서 동시에 사용할 React 컴포넌트를 만들고  
서버에서 프론트로 던져주는 response 에 renderToString을 사용해 실제 HTML 마크업을 돌려준다. 
자 이제 서버에서 렌더링 된 react 컴포넌트는 프론트엔드 HTML마크업으로 출력되어 SEO를 구현할 수 있게 되었다.

그리고 조금 더 재밌는 작업을 하게 되는데 HTML5 history api를 이용한 react route 를 이용한다.

## 디테일한 상황 예

클라이언트에서 사용자가 페이지를 이동한다. 예를 들면 게시판 리스트에서 제목을 눌러 내용을 보러간다고 가정해 보자.  
React의 갖고 있는 개념대로 페이지 전체를 다시 로드하지 않고 해당 부분만을 다시 렌더링 한다. 
(URI는 초기 HTML5 history만 이용할때는 #이 붙은 history를 유지하겠지만 지금은 #을 없애는 많은 라이브러리가 있다.)

여기서 일반적인 MVC를 사용한 프로젝트라면 처음 게시판 리스트를 사용자가 접근했을때는 HTML 마크업이 정상적으로 게시판 리스트를 출력하고 있을 것이다.
하지만 제목을 클릭하여 본문을 보려고 할때 AJAX로 처리를 한다면?  

HTML5 history를 제대로 사용하고 있다고 하더라도 게시글을 보여주는 방식이 두 가지 이기 때문에 ( 리스트에서 AJAX로 보여주는 방식과 permalink로 바로 접근했을때 보여주는 방식 )
개발자는 여간 피곤하지 않을 수 없다.

react, react-route를 이용한다면 리스트에서 AJAX로 보여주는 방식에 대해서는 고민할 필요가 없어진다. 
즉, 서버단에서만 처리를 하면 되는 것이다.

    react-route : /bbs/list
                  /bbs/77
                   
이라고 가정하고 ( 실제로는 route는 `<Route path="/thread" component={ ThreadBox } />` 같은 형태로 정의한다 )

1. list 에서 77번 글을 사용자가 클릭해서 보려고 할때   
URI는 /bbs/list 에서 /bbs/77 로 history를 이용해 변경하고  
/bbs/77에 해당하는 react 컴포넌트를 서버에서 처리하고 마크업을 프론트로 보내주고 클라이언트의 react는 이를 AJAX로 처리해 해당 영역을 다시 렌더링 한다.

2. 사용자가 직접 /bbs/77로 접근할때  
/bbs/77에 해당하는 react 컴포넌트를 서버에서 처리하고 마크업을 프론트로 보내준다.

정말 아름다운 isomorphic, universal이 아닐 수 없구나!

일반적인 MVC를 작업할 때는 view를 위한 HTML과 JS를 따로 작업해야 했는데 이젠 이렇게 작업할 필요가 없어졌다는 것 또한 즐거운 일이다.


