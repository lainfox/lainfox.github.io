---
layout:     post
title:      기존에 익숙하게 사용하던 ES5 Javascript 와 ES6 비교 
date:       2015-10-28
summary:    Front-end 에서 많이 사용하는 Javascript의 ES6 버전을 아직 많이 사용하고 있지 않다. 비교를 해보자.
categories: dev
---
*이 문서는 지속적으로 업데이트 될 예정입니다.*

2015년 6월 ES6 최종 스팩이 승인되었습니다.  
크롬과 파이어폭스에서는 쉽게 ES6 을 사용할 수 있지만 다른 브라우저는 
[babel][1], [traceur][2], [es6-shim][3]등을 이용해야 합니다. (추후 업데이트)



## 변수 선언 var vs let

ES6 에서 사용하는 변수 선언. ES5 에서 사용하던 var 외에 const나 let 

    var a = 5;
    var b = 10;

    if (a === 5) {
      let a = 4; // The scope is inside the if-block
      var b = 1; // The scope is inside the function

      console.log(a);  // 4
      console.log(b);  // 1
    }

    console.log(a); // 5
    console.log(b); // 1

## module import

    // ES5
    var React = require('react');
    var Router = require('react-router');

    var Route = Router.Route;
    var DefaultRoute = Router.DefaultRoute;
    var NotFoundRoute = Router.NotFoundRoute;

    // ES6
    import React from 'react';
    import {Route, DefaultRoute, NotFoundRoute} from 'react-router';
    
모듈에 대한 자세한 이야기는 (ECMAScript 6 modules: the final syntax)[http://www.2ality.com/2014/09/es6-modules-final.html] 
에서 확인해 보자.

## 클래스

    // ES5
    function Box(length, width) {
      this.length = length;
      this.width = width;
    }
    
    Box.prototype.calculateArea = function() {
      return this.length * this.width;
    }
    
    var box = new Box(2, 2);
    
    box.calculateArea(); // 4


    // ES6
    class Box {
      constructor(length, width) {
        this.length = length;
        this.width = width;
      }
      calculateArea() {
        return this.length * this.width;
      }
    }

    let box = new Box(2, 2);
    
    box.calculateArea(); // 4
    
익숙한 클래스 형태.  
  
  
## 클래스 상속(확장)

    // ES5
    var MyComponent = React.createClass({
      // React 컴포넌트 메소드를 사용할 수 있음
      // componentDidMount, render 등등
    })


    // ES6
    class MyComponent extends React.Component {
      // React 컴포넌트 메소드를 사용할 수 있음
      // componentDidMount, render 등등
    }

또한 익숙한 클래스 상속 형태.


## Map ( Fat arrow function )

    // ES6
    [1, 2, 3].map(n => n * 2); // [2, 4, 6]
    
    // ES5
    [1, 2, 3].map(function(n) { return n * 2; }); // [2, 4, 6]

백엔드 언어에서 사용하던 것과 유사하네요.


----
Ref.  

- http://sahatyalkabov.com/create-a-character-voting-app-using-react-nodejs-mongodb-and-socketio/


Resource.

- https://babeljs.io
- https://github.com/google/traceur-compiler
- https://github.com/paulmillr/es6-shim

  [1]: https://babeljs.io
  [2]: https://github.com/google/traceur-compiler
  [3]: https://github.com/paulmillr/es6-shim
