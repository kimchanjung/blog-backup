---
layout: post
title: "[React] Mobx 와 react-router 사용시 history를 이용한 url 이동이 동작하지 않는 경우 해결방법"
description: "[React] Mobx 와 react-router 사용시 history를 이용한 url 이동이 동작하지 않는 경우 해결방법"
author: kimchanjung
date: 2020-06-22 12:00:00 +0900
categories: react
published: true
---

# Mobx 와 react-router 사용시 history를 이용한 url 이동이 동작하지 않는 경우 해결방법

## 일반적으로 React에서 react-router 사용시 링크를 설정하는 경우
```react
<Link to="/about">About</Link>
// <a href="/about">About</a> 와 같습니다
```
> React에서 react-router와 같이 사용하는 경우 링크설정시 아래와 같이 사용합니다.

## javascript에서 history를 이용하여 동적으로 URL 이동하는 경우
```javascript
document.location.href = "/about";
document.location.hash = "#about";
window.history.back()
window.history.pushState(null, null, "/about")
```
> 일반적인 상황에서는 a태그 대신 javascript를 사용해서 위 예제 처럼 동적으로 페이지 이동이 필요한 경우도 있습니다.  


## React에서 react-router를 사용하는 경우 동적으로 페이지 이동 하는 방법
### Router 설정
```javascript
import { createBrowserHistory } from "history";

const history = createBrowserHistory();

ReactDOM.render(
  <Router history={history}>
    <App />
  </Router>,
  node
);
```
> React에서 react-router를 사용하는 경우 보통 createBrowserHistory() 사용하여 history props에 선언합니다  
> 그리고 각각의 컴포넌트에 history, location, match 객체를 props로 제공하여 사용할 수 있도록 합니다.

### 컴포넌트에서 react-router의 history 이용하여 페이지 이동하는 예
```javascript
class Comp extends React.Component {
  movePage() {
   this.props.history.push("/about")
  }
}
```
> react-router가 제공하는 props 중에 history를 사용하여 페이지 이동이 동적으로 가능합니다

### 컴포넌트 이외(store나 다른 함수등에서) react-router의 history를 이용하여 페이지 이동하는 예
```javascript
 import { useHistory } from "react-router-dom";

class AboutStore() {
  history = useHistory();

  handleClick() {
    this.history.push("/home");
  }
}
```
> 컴포넌트는 react-router 설정시 props에 history를 제공하기 때문에 사용가능하지만 그외 곳 store등에서 사용시에는 react-router-dom의 useHistory() 함수를 통하여 history를 가져와서 사용이 가능합니다. 

## mobx를 적용한 프로젝트에서 react-router의 동적 페이지 이동이 동작하지 않는 문제 해결
### Router 설정
```javascript
import { createBrowserHistory } from "history";
// mobx-react-router 모듈을 추가합니다.
import { syncHistoryWithStore, RouterStore } from 'mobx-react-router';

const browserHistory = createBrowserHistory();
const routerStore = new RouterStore();
// mobx-react-router 의 syncHistoryWithStore 함수로 한번더 설정을 추가합니다.
// 설정을 추가한 history를 props에 전달합니다.
const history = syncHistoryWithStore(browserHistory, routerStore);

ReactDOM.render(
  <Router history={history}>
    <App />
  </Router>,
  node
);
```
- mobx-react-router 라는 라이브러리를 설치합니다.
- mobx-react-router 의 syncHistoryWithStore, RouterStore 를 이용하여 기존 Router의 history 설정에 추가적인 설정이 포함되도록 합니다.

### 컴포넌트에서 페이지 이동하는 방법
기존 방법과 동일합니다. 컴포넌트의 props에 history를 기존대로 사용하면 됩니다.

### 컴포넌트 이외의 곳(store)에서 history를 사용하여 페이지 이동하는 방법
```javascript
import { RouterStore } from 'mobx-react-router';

class AboutStore() {
  routerStore = new RouterStore();

  handleClick() {
    this.routerStore.history.push("/home");
  }
}
```
> mobx-react-router의 RouterStore 객체를 생성하여 history를 가져온 다음 동적으로 페이지 이동이 가능합니다.

## 마무리
react-router 사용할때 동적으로 페이지를 이동하는 방법에서 mobx를 사용할때 동작하지 않는 부분을 mobx-react-router 라이브러리를 이용하여 설정을 추가한 후 해결하는 방법을 알아 보았습니다.  

React로 개발하다 보면 뭔가 매끄럽거나 깔끔하지 못한 부분이 많은데 아무래도 React는 rendering만 제공하는 프레임워크가 아닌 라이브러리이기 때문에 나머지 기능은 서드파티 라이브러리를 사용한다는 점에서 매끄럽지 못한 것 같다.  

깔끔한 방법을 아시는 분은 코멘트 남겨 주시면 감사하겠습니다.