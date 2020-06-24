---
layout: post
title: "[React] react-router에서 Private Router 구현하기(권한별 routing)"
description: "[React] react-router에서 Private Router 구현하기, 권한별 routing 메뉴 구현"
author: kimchanjung
date: 2020-06-24 18:00:00 +0900
categories: react
published: true
---

# react-router 에서 Private Router 구현하기(권한별 routing)
실무에서 페이지를 개발할 때 인증 및 **부여된 권한 등급에 따라서 메뉴의 노출 및 접근이 제한** 되도록 구성하는것이 일반적입니다. **react-route**r 라이브러리는 **routing**을 구현하는 라이브러리인데 뭔가 라이브러리 차원에서 **권한별 라우팅을 쌈박한 방법**으로 지원해 줄 것이라는 기대와 달리 뭔가 **복잡하게 구현하는 예제만 제공**되고 있었습니다. 본 포스팅에서 **권한을 체크하여 routing 이 가능**하도록 하고 권한이 없으면 **권한이 없다는 페이지로 redirect**하는 방법을 알아보고자 합니다.


## 권한을 체크하는 Router Component 생성
```react
import { Route, Redirect } from 'react-router';

const PrivateRoute = ({component: Component, ...parentProps}) => {
  return (
    <Route
      {...parentProps}
      render={props => (
        checkAuth() ? (
         <Component {...props} parentMenu={this.props.menu} />
        ) : (
         <Redirect to={{ pathname: '/403' }} />
        )
      )}
    />
  );
}
```
> Route 컴포넌트가 호출되기 전에 권한 체크를 한다음 권한이 있으면 지정된 컴포넌트(페이지)를 호출 하고 권한이 없다면 권한없음을 안내하는 페이지로 redirect 시킵니다.  

#### parentProps의 설명
**부모 컴포넌트로 부터 넘겨 받은 props**에서 Component를 제외한 **나머지 props를 parentProps에 모두 담습니다**.   
**parentProps**는 **exact**, path="/users" 같은 **react-router**의 **Route 컴포넌트**를 사용할때 제공하는 설정 값들을 넘겨받아서 **{...parentProps}** 와 같이 스프레드 문법으로 **그대로 설정해주기 위함**입니다.  
```html
  // parentProps로 받아온 나머지 props를 스프레드 문법으로 풀어주면
  <Route {...parentProps} />
  // 사실상 아래 처럼 되는 것과 같습니다.
  <Route exact path="/users" .... />
```

## 권한을 체크하는 커스텀 Private Router를 사용
```react
import PrivateRoute from './PrivateRoute';

class Routers extends React.Component {
  constructor(props) {
    super(props);
  }

  render() {
    return (
      <React.Fragment>
        // Route 컴포넌트의 사용방식과 동일하게 사용하면 된다.
        <PrivateRoute exact path="/rider-management" component={RiderManagement}/>
        <PrivateRoute exact path="/common-management" component={CommonManagement} />
      </React.Fragment>
    );
  }
}

export default Routers;
```
> 권한 체크 기능이 추가된 react-router Route 컴포넌트를 래핑한 PrivateRouter를 사용하도록 하여 router를 구성한다. 

## 마무리
권한 체크기능이 추가된 **Private Router**는 **react-router**의 **Router**에 **컴포넌트를 호출할 때 간단한 분기처리**가 추가된 방법으로 간단하게 구현해 보았습니다.