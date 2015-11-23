---
layout:     post
title:      React transmit with flux store, dynamic data with state
date:       2015-11-10
summary:    Transmit 또는 Relay를 flux와 함께 사용하기
categories: dev
---

React를 사용하면서 flux 의 action > store > component 단방향 데이터 흐름에 매료되었는데 문제가 발생.
프론트엔드, SEO를 크게 신경 안쓴다면 전혀 문제가 아닐 수도..;;

일단 페이스북에서 나온 Relay는 graphQL을 사용하는 데이터 처리 라이브러리입니다. flux와 다른 데이터 처리를 하죠. 
relay에서 영감을 받아 나온 react-transmit은 graphQL을 사용하지 않고 promise 패턴으로 처리합니다. 구현 형태도 거의 유사하죠.
이 녀석들은 대략적으로 얘기하자면 component 에서 relay 를 통해 Async 데이터를 가져와 데이터를 처리하는 녀석인데 
문제는 상위 컨테이너를 생성하고 props를 해당 component에서 사용하게 만듭니다.

이게 뭐가 문제냐구요?



## Situation & Problem

일단 전제가 되는 상황은 server-side rendering 을 함께 처리하는 isomorphic/universal 을 하는 상황입니다.

1. 그냥 데이터를 fetch 해서 프론트에서 출력한다면 component의 `componentDidMount`정도에서 비동기 API를 호출하면 동적으로 DOM을 생성해서 마크업에 들어가겠죠.
server-side rendering 이 없기 때문에 실제 데이터들은 마크업의 소스코드에 노출되지 않습니다.   
현재까지 찾아본 isomorphic/universal react 들은 모두가 init 시의 동적생성 코드에 대한 해결책을 주지 않고 있더라구요.


2. flux에 매료된 저에게는 action > store > component 와 props 가 아닌 state 를 observer 하면서 다이나믹한 데이터 처리를 하는 게 너무 좋다는 것이죠.
relay나 transmit을 사용할때는 state 를 observer하지 않고 props를 보고 있기 때문에 데이터의 추가/제거 할때 다시 graphQL이나 promise fetch 처리를 해야 합니다. ( 일반적인 async 데이터 처리 )

react-transmit의 이슈 트래커에 이런 이슈들이 올라왔지만 아직 안되고 있죠. 아마도 안 될 가능성이 높을 거 같습니다. relay가 store를 사용하는 방향으로 가지 않는한..

그래서 해결점을 만들어 보았습니다. 의외로 간단히 처리가 되어버린;;;


## Solution

기존 Transmit 을 사용할때의 코드를 잠시 보자면

  class Thread extends React.Component {
  
    constructor(props) {
      super(props);
      ...
    }
    ....
    componentDidMount() {
      ThreadStore.listen(this.onPostChange);
      // Ajax API 호출을 하지 않음. Transmit을 이용해 init 데이터를 server-side에서 렌더링
    }
    ...
    render() {
      let { post, comment } = this.props; // Transmit으로 post 데이터는 보장됩니다.
      // ES5 라면 post = this.props.post, comment = this.props.comment
      // this.state 를 사용할 수 없습니다 ;ㅅ;
      ...
      }
    }
      
    export default Transmit.createContainer(Thread, {
      ....
      Promise.all ...
    })
  
이런 형태로 사용하게 되면 위의 `post`나 `comment` 는 Transmit 에서 promise 로 처리할 데이터를 받게 되고 server-side에서 렌더링 하게 됩니다. 
여기까지는 아주 좋습니다 :D

다만 react render에 있는 this.**props**.comment 는 this.**state**.comment 가 아니기 때문에 상태를 리스닝하고 있지 않기에 store의 dispatch가 불가능 합니다.  
그냥 다시 transmit의 fetch 를 통해도 되지만... flux store 를 꼭 쓰고 싶습니다!

이제 조금 변경해 보죠.

아이디어는 간단합니다. store의 dispatch로 state가 변경되면 `comment`에 state 의 값을 concat 해서 붙이는 것이죠 !
이것은 react의 render가 state change 마다 읽기 때문에 가능합니다


react render 부분을 약간 변경합니다.  comments를 그냥 props를 사용하지 않고 데이터처리 후 return을 받아 처리하는 거죠.


    render() {
      let { post } = this.props;
      let comments = this.getComments();
      
  
그리고 getComments를 구현합니다.

    getComments() {
      let comments = [];
        
      if( !_.isEmpty( this.state.comments ) ) { // 클라이언트에서는 초기 로드시 this.state.comments 는 {} 일 거에요. state는 store의 dispatch가 일어나면 추가되었겠죠.
        comments = this.props.post.comments.concat( this.state.comments ); // props의 comments 는 Transmit으로 인해 생성되어있죠. 거기에 state 내용을 추가합니다.
      }
      else {
        comments = this.props.post.comments; // 초기 props의 comments
      }
      
      // console.log( 'comments: ', comments.length )
      return comments;
    }

자 일단 완성. 그런데 문제가 발생합니다. 같은 key의 react component 오류가 나오게 되죠.   
이유는 위의 `comments = this.props.post.comments.concat( this.state.comments );` 부분입니다. 
이 react component는 서버와 프론트 양쪽에서 모두 실행되기 때문인데요.

- server-side : this.props.post.comments 있음 / this.state.comments 있음 
- cliend-side : this.props.post.comments 있음 / this.state.comments 없음

이렇게 되어있게 되는 거죠. 그렇기 때문에 server-side 에서 this.props.post.comments 에 this.state.comments 를 그냥 `conacat` 해 버리면 똑같은 배열이 두개 연결되는 결과를 보여줍니다.

- server-side : concat 한 후에 comments.length = 22
- client-side : concat 한 후에 comments.length = 11

그래서 중복된 배열값은 concat 하지 말아 보죠. 또한 서버와 클라이언트 렌더링을 분기해서 처리하면 불필요한 처리를 피할 수 있겠네요.

    
    getComments() {
      let comments = [];
      
      if (__SERVER__) { // global 변수로 __SERVER__ 를 만들어서 처리하고 있는데 그냥 if (typeof window === 'undefined') { // For server 정도를 사용할 수도 있습니다.
        // console.log("Hello server");
        comments = this.props.post.comments;
      }
      
      else if (__CLIENT__) { // if (typeof window !== 'undefined') { // For client
        // console.log("Hello client");
        if( !_.isEmpty( this.state.comments ) ) {
          // comments = this.props.post.comments.concat( this.state.comments ); // 이놈을 사용하지 않고
          comments = _.union( this.props.post.comments, this.state.comments); // underscore를 이용해 union으로 유니크한 배열만 합칩니다.
        }
        else {
          comments = this.props.post.comments;
        }
      }
      
      // console.log( 'comments: ', comments.length )
      return comments;
    }


자 이제 `comments`는 props의 기본 데이터에 state 가 변할때마다 state.comments 를 추가하게 되었네요. 

Yeahhhhhhh~~


