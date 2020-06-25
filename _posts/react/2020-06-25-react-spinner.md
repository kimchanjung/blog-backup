---
layout: post
title: "[React] 로딩처리용 spinner를 효과적이고 깔끔하게 적용하는 방법"
description: "[React] 로딩처리용 spinner를 효과적이고 깔끔하게 적용하는 방법"
author: kimchanjung
date: 2020-06-25 13:00:00 +0900
categories: react
published: true
---
# React에서 로딩처리용 spinner를 효과적이고 깔끔하게 적용하는 방법
**API호출**시 보통 **로딩을 처리하는 동안 Spinner를 표시**해서 로딩처리를 사용자에게 알리는 용도로 많이 사용됩니다. 
**NPM에서 React용 Spinner**를 찾아보면 **수많은 종류**의 로딩처리 Spinner 모듈들이 존재합니다. **기능이나 디자인도 수없이 다양**합니다. NPM **대부분의 Spinner 모듈이 다양한 방식**으로 사용하고자 할때 **몇가지 문제점**이 있었습니다.
이번 포스팅은 **Spinner 모듈을 가져와 사용할 때 발생하는 문제점을 해결**하고 **깔끔한 방식으로 사용**할 수 있는 방법을 알아봅니다.

## 사용 모듈 버전정보
- **react-loader v2.4.5**
- **react-portal v4.1.5**

## React에서 NPM Spinner 모듈들을 사용하는 방식의 문제점
대부분의 **NPM의 Spinner 모듈**을 사용하려고 할때 다음과 같은 몇가지 **벽에 부디치게 됩니다**.  

### Modal이나 Dialog 사용시 Spinner의 위치 설정문제
#### 대부분의 모듈들의 사용방법을 보면 아래와 같습니다.
```javascript
class UserPage extends React.Component {
  render() {
    return (
      <>  
        <Loader loaded={this.state.loaded} />
        <UserPageTemplate model={this.state.profile} />
      </>
    );
  }
}            
```
- 렌더링하는 컴포넌트에 Loader 컴포넌트를 가져와서 사용하는 방식입니다.
- 이런 경우 일일히 모든 페이지에 Loader 가져와서 사용해야합니다.

#### 그러나 한곳에만 Loader를 포함시켜놓고 코드 중복없이 사용하고 싶습니다. 
#### App 컴포넌트 
```javascript
// Loader 컴포넌트를 포함시킨다.
@inject('store')
@observer
class App extends React.Component {
  render() {
    return (
      <>
        <Loader loaded={this.props.store.isLoadingSpinner} />
        <UserPage />
        <OrderPage />
        <AccountPage />
      </>
    );
  }
}  
```  
#### User 페이지 컴포넌트 
```javascript
// 개별페이지들은 Store의 값을 컨트롤하여 Spinner를 조작한다.
// 개별페이지들은 일일히 Spinner 모듈을 가져와 사용할 필요 없어져서 중복이 제거 되었다.
@inject('store')
@observer
class UserPage extends React.Component {
  componentDidMount() {
    this.props.store.visibleSpinner(true);
  }
  render() {
    return (
      <>  
        <UserPageTemplate model={this.state.profile} />
      </>
    );
  }
}
``` 
#### Store
```javascript
// 스토어는 Mobx를 예로 들었습니다.
class Store {
  @observable  
  isLoadingSpinner = false;

  @action
  visibleSpinner(value) {
    this.isLoadingSpinner = value;
  }
} 
```
> 일일이 개별페이지에서 가져다 사용하는 것이 아니라 상단 컴포넌트에 한 곳에만 Loader를 지정해두고 각각의 페이지는 state관리 라이브러리를 사용하여 store에 선언된 state를 조작하여 Spinner 모듈을 컨트롤하여 사용하도록 합니다.

#### Modal이나 Dialog는 최상의 엘리먼트 밖에 생성됩니다
#### App 컴포넌트
```javascript
// App.js
@inject('store')
@observer
class App extends React.Component {
  render() {
    return (
      <>
        <Loader loaded={this.props.store.isLoadingSpinner} />
        <UserPage />
        <OrderPage />
        <AccountPage />
      </>
    );
  }
}  

// index.js
ReactDOM.render(<App />, document.getElementById('root'));
```
#### 모달이 생성되는 위치
```html
<body>
  <div id="root">
    // 실제 페이지를 구성하는 html등이 위치한다.
    <div id="spinner" style="......">
        // 로딩 스피너는 여기에 포함된다.
    </div>
  </div>
  <div id="modal">
    // 모달을 구성하는 html등이 위치한다.
  </div>
</body>
```
- **React App**은 id="root" 엘리먼트 하위에 생성됩니다.
- **Modal은 App**밖에 생성됩니다.
- 이런경우 **공통으로 사용하기 위한 Spinner**는 **모달에서 사용할 때** 모달엘리먼트 **상위나 하위에 포함된 것이 아니라서** 위치나 포지션 설정시 **원하는 위치에 설정하기 불가능한 상황이 발생합**니다.

## React Potal을 이용하여 Spinner 모듈 컴포넌트를 App밖에 생성합니다.
#### Potal을 이용하여 특정 엘리먼트에 Spinner를 동적으로 위치시키는 컴포넌트
```javascript
// SpinJsLoader.js
import { Portal } from 'react-portal';

export default function SpinnerLoader(props) {
  // containerName은 Spinner를 추가하고 싶은 특정 엘리먼트의 클래스네임입니다.
  const { isLoading, containerName } = props;
  return (
    <Portal node={document.getElementsByClassName(containerName)[0]}>
      <Loader
        className="spinner"
        loaded={!isLoading}
      />
    </Portal>
  );
}
```
- 여기서는 R**eact Portal**을 간단하게 사용하게 해주는 **NPM모듈을 사용**하였습니다.  
- **Spinner(Loader) 모듈**은 이제 **특정 클래스명을 가진 엘리먼트에 동적으로 추가**되록 구성되었습니다.  
- 결국은 **$('.app-root').append('<div id='spinner' style='....'>...</div>');** 와 같은 기능입니다.

#### App 컴포넌트에 추가합니다.
```javascript
import { SpinJsLoader } from 'SpinJsLoader';

class App extends React.Component {
  render() {
    return (
      <>
        <SpinnerLoader 
          isLoading={this.props.store.isLoading}
          container={this.props.store.containerName}
        />
        <UserPage />
        <OrderPage />
        <Modal />
      </>
    );
  }
} 
```

#### Store
```javascript
// 스토어는 Mobx를 예로 들었습니다.
class Store {
  @observable  
  isLoading = false;

  @observable
  containerName;
  
  // spinner를 사용할 때는 표시여부와 spinner가 위치할 엘리먼트의 클래스명을 지정합니다.
  @action
  visibleSpinner(isLoading, containerName) {
    this.isLoading = isLoading;
    this.containerName = containerName;
  }
} 
```

#### 개별 컴포넌트들은 상황에 맞게 Spinner를 조작합니다.
```javascript
// 사용자 페이지 컴포넌트
@inject('store')
@observer
class UserPage extends React.Component {
  componentDidMount() {
    this.props.store.visibleSpinner(true, 'app-root');
  }
  render() {
    return (
      <>  
        <UserPageTemplate model={this.state.profile} />
      </>
    );
  }
}

// 검색용 모달 컴포넌트
@inject('store')
@observer
class Modal extends React.Component {
  componentDidMount() {
    this.props.store.visibleSpinner(true, 'search-modal');
  }
  render() {
    return (
      <>  
        <ModalTemplate model={this.state.profile} />
      </>
    );
  }
}
```
- 각각의 컴포넌트나 모달은 Spinner를 사용할 때 **Spinner를 위치시킬 엘리먼트의 class명을 함께 지정**합니다.
- **Spinner**는 생성될때 **지정된 class 하위에 생성**됩니다. **App밖에 생성되는 Modal하위에도 Spinner**는 생성됩니다.
- **Spinner**의 **position** 즉 **absolute, fixed** 설정이 **Modal을 기준으로 위치**할 수 있게 됩니다.
- 애초에 **App 내부**에 포함하여 사용하는 방식이면 **Modal에서 position 설정의 기준이 다르기** 때문에 **맞지않게 되는 문제**가 발생하지만 **Potal을 이용하여 동적으로 Spinner를 위치** 시키므로써 가능해 졌습니다.

### API호출시에 자동으로 Spinner가 로딩되도록 하는 팁
```javascript
import axios from 'axios';
import { store } from 'stores';

class AxiosConfig {
  initInterceptor(store, history) {
    const { contentRootStore } = rootStore;
    axios.interceptors.request.use(
      (config) => {
        // API 호출시 Spinner 표시 
        store.visibleSpinner(true);
        return config;
      },
      error => Promise.reject(error),
    );
    axios.interceptors.response.use(
      (response) => {
         // API 완료시 Spinner 제거  
        store.visibleSpinner(false);
        return response;
      },
      error => Promise.reject(error).catch((err) => {
         // API 오류시 Spinner 제거  
        store.visibleSpinner(false);
        return response;
      }),
    );
  }
}
```
> axios(http 호출 모듈)를 사용한다면 API 호출시 Spinner를 표시하고 완료시 제거하는 store의 메소드를 호출하는 코드를 넣어두면 개별 페이지에서 Spinner 호출 코드를 제거하여 중복을 줄일 수 있다. 

### 마무리
**Spinner모듈을 Modal**과 일반 페이지 둘다 **position 설정이 가능하도록 Portal**을 이용하여 **구현하는 방법**을 알아 보았습니다.
