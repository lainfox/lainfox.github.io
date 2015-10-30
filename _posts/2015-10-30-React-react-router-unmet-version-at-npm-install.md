---
layout:     post
title:      React, react-router, history npm 설치시 버전 충돌 
date:       2015-10-30
summary:    react-router가 react와 버전이 맞지 않을 때 해결
categories: dev
---

package.json의 dependencies에 현재 react 버전은

    ...
    "react": "^0.14.0",
    "react-dom": "^0.14.0",
    ...

react-router 를 사용하기 위해 가볍게 react-router를 인스톨.
  
    npm install --save react-router
    
    npm WARN EPEERINVALID react-router@0.13.4 requires a peer of react@0.13.x but none was installed.

에러 발생. react-router 0.13.4 버전이 설치되려고 했는데 react 0.13.x 버전이 필요함. 근데 내 환경에는 0.14.x 버전이 설치되어있어서 발생하는 오류.  

일단 npm 을 업데이트.
  
    sudo npm install -g npm
    npm install --save react-router
    
이번에도 에러 발생.
  
    react-todo@1.4.0 /Users/lainfox/Workspace/node/react-todo
    ├─┬ hapi@11.0.2
    │ └─┬ subtext@2.0.0
    │   └── qs@4.0.0 
    ├── UNMET PEER DEPENDENCY react@0.14.1
    └── react-router@0.13.4 
    
    npm WARN EPEERINVALID react-router@0.13.4 requires a peer of react@0.13.x but none was installed.
    
react-router 최신 버전으로 설치 및 분리된 history 도 설치
  
    npm install --save history react-router@latest
    
    react-todo@1.4.0 /Users/lainfox/Workspace/node/react-todo
    ├─┬ hapi@11.0.2
    │ └─┬ subtext@2.0.0
    │   └── qs@4.0.0 
    ├─┬ history@1.13.0 
    │ ├── deep-equal@1.0.1 
    │ ├─┬ invariant@2.1.2 
    │ │ └─┬ loose-envify@1.1.0 
    │ │   └── js-tokens@1.0.2 
    │ ├── qs@4.0.0 
    │ └── warning@2.1.0 
    ├─┬ react@0.14.1
    │ └─┬ fbjs@0.3.2
    │   └─┬ loose-envify@1.1.0 
    │     └── js-tokens@1.0.2 
    └─┬ react-router@1.0.0-rc3 
      └── history@1.12.3 
      
    
npm 업데이트 하다가 npm 이 날아가 버리는 바람에  
별것 아닌 일에 20여분의 삽질; 정신차리자.

https://github.com/rackt/react-router
