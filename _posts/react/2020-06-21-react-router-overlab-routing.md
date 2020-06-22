---
layout: post
title: "[react] react-router 에서 트리구조의 부모자식 URL 페이지를 구성하는 방법"
description: "[react] react-router 에서 트리구조의 부모자식 URL 페이지를 구성하는 방법"
author: kimchanjung
date: 2020-06-22 09:00:00 +0900
categories: react
published: true
---

# react-router 에서 트리구조의 부모자식 URL 페이지를 구성하는 방법
react-router를 이용해서 아래와 같은 url 구조가 있을 때 구현 하는 방법을 알아 봅니다

### URL 항목
URL 이렇게 존재한다고 했을 때
```bash
user/list
user/detail
order/list
order/detail
```

### URL의 구조
일반적으로는 이렇게 부모 자식으로 구성됩니다.
```bash
user - list
     - create

order - list
      - create
```

### react-router 에서 일반적인 routing 방식
```react
<React.Fragment>
    <Route exact path="/user/list" component={UserList} />
    <Route exact path="/user/create" component={UserCreate} />
    <Route exact path="/order/list" component={UserList} />
    <Route exact path="/order/create" component={UserCreate} />
</React.Fragment>
```
> user 와 order가 중복이 됩니다 URL이 많아 지면 복잡해 보이기도 합니다.

### react-router 에서 URL 매핑을 부모자식(트리구조)구조로 구현 하는 방법
```react
<React.Fragment>
    <Route
        path="/user"
        render={props => (
            <React.Fragment>
                <Route exact path={`${props.match.url}/list`} component={UserList} />
                <Route exact path={`${props.match.url}/create`} component={UserCreate} />
            </React.Fragment>
        )}
    />
    <Route
        path="/order"
        render={props => (
            <React.Fragment>
                <Route exact path={`${props.match.url}/list`} component={OrderList} />
                <Route exact path={`${props.match.url}/create`} component={OrderCreate} />
            </React.Fragment>
        )}
    />
</React.Fragment>
```

### react-router 에서 URL 매핑을 트리구조 구조의 컴포넌트로 분리하는 방법
부모 자식의 트리 구조로 컴포넌트를 분리하여 URL 매핑을 구성할 수 있습니다.  

#### Root Router (부모라우터)
```react
<React.Fragment>
    <Route path="/user" component={UserRouters} />
    <Route path="/order" component={OrderRouters} />
</React.Fragment>
```
> 상위 URL을 정의 해놓고 하위 URL을 매핑을 정의 해놓은 router 컴포넌트를 호출합니다.

#### User Routers (자식라우터)
```react
export default function UserRouters({ match }) {
  return (
    <React.Fragment>
        <Route
            path={match.url}
            render={props => (
                <React.Fragment>
                    <Route exact path={`${props.match.url}/list`} component={UserList} />
                    <Route exact path={`${props.match.url}/create`} component={UserCreate} />
                </React.Fragment>
            )}
        />
    </React.Fragment>
  )
}
```
> /user/~~~~ 시작하는 url은 모두 위 컴포넌트를 호출하게 됩니다.

#### OrderRouters (자식라우터)
```react
export default function OrderRouters({ match }) {
  return (
    <React.Fragment>
        <Route
            path={match.url}
            render={props => (
                <React.Fragment>
                    <Route exact path={`${props.match.url}/list`} component={UserList} />
                    <Route exact path={`${props.match.url}/create`} componet={UserCreate} />
                </React.Fragment>
            )}
        />
    </React.Fragment>
  )
}
```
> /order/~~~~ 시작하는 url은 모두 위 컴포넌트를 호출하게 됩니다.

### 마무리
URL이 많을 경우 모튼 URL 매핑을 나열하면 부모자식 구조가 잘 한눈에 보이지 않고 파악이 어렵지만 위 같은 방식으로 부모자식으로 패턴을 구분하여 구현 할 수 있습니다.  

개인적으로는 react-router의 부모자식 구조로 router를 구성하는 방식이 그리 깔끔한 것 같지는 않다는 생각이 듭니다.  
조금더 깔끔한 방법을 아시는 분은 댓글 부탁드립니다.