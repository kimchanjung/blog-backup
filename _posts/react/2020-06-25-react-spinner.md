---
layout: post
title: "[React] 로딩처리용 spinner를 효과적이고 깔끔하게 적용하는 방법"
description: "react loading loader spinner api 적용 깔끔"
author: kimchanjung
date: 2020-06-26 14:00:00 +0900
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

## Modal이나 Dialog 사용시 Spinner의 위치 설정문제
대부분의 NPM에서 React용 모듈을 사용하는 방법은 모듈을 import하고 컴포넌트를 삽입하는 방법입니다. 

## React 컴포넌트의 부모자식 구조
**컴포넌트를 삽입**하면 실제 렌더링된 **DOM의 형태도 마찬가지로 해당 부모자식 DOM 엘리먼트 구조**로 렌더링 됩니다.
**NPM React용 Spinner모듈들이 대부분 컴포넌트를 삽입하는 방식**입니다(제가 찾은 모듈은 대부분) 결국은 **Spinner 모듈도 id="app"인 div 하위 어딘가에 포홤**된다는 이야기가 됩니다. 하지만 **Modal의 경우**는 보통 **전체화면**의 메인이 되는 **영역밖에 생성**되는 것이 보통합니다.
```javascript
class App extends React.Component {
  render() {
    return (
      <div id="app">  
        <Loader />
        <Search  />
        <UserPages />
      </div>
    );
  }
} 

class Search extends React.Component {
  render() {
    return (
      <form>  
        <input id="address" name="address" .... />
        <input id="phone" name="phone" .... />
        ...
      </form>
    );
  }
} 

class UserPages extends React.Component {
  render() {
    return (
      <div>  
        <UserList />
        <Pager/>
        ...
      </div>
    );
  }
}
```

## React 컴포넌트가 랜더링된 결과 
```html
<div id="app">
  <div id="spinner">
    로딩 스피너 관련 엘리먼트들이 위치하는 곳
  </div>
  <form>
    <input id="address" name="address" .... />
    <input id="phone" name="phone" .... />
    ...
  </form>
  <div>  
    <table>
     ~~~~
    </table>
    <div>
      <span>1</span>
      <span>2</span>
      <span>3</span>
      <span>next</span>
    <div>
  </div>
</div>
```  

## React에서 Modal의 사용과 렌더링된 결과
**Modal 컴포넌트**를 분명 **UserPage 컴포넌트에 삽입**하여 사용했는데도 불구하고 실제 **렌더링은 root 엘리먼트 밖**에 생성이 되는 결과를 가져왔습니다. **포지션이나 스크롤 기타 여러 style설정**과 관련하여 전체 **화면밖에 생성**되는 것이 일반적입니다.
```javascript
class UserPage extends React.Component {
  render() {
    return (
      <div>  
        <UserList />
        <Pager/>
        <Modal /> 
      </div>
    );
  }
}

<div id="app">
  <div id="spinner">
    로딩 스피너 관련 엘리먼트들이 위치하는 곳
  </div>
  <form>
    <input id="address" name="address" .... />
    <input id="phone" name="phone" .... />
  </form>
  <div>  
    <table>
      사용자리스트
    </table>
    <div>
      <span>1</span>
      <span>2</span>
      <span>3</span>
      <span>next</span>
    <div>
    // 이자리에 modal이 렌더링되어야할 위치 같지만 실제는 밖에 생성됩니다.
  </div>
</div>

// 대부분 modal의 경우 포지션이나 설정이나 기타 그런 것 들 때문에 
// 전체 페이지 영역밖에 생성되는 경우가 일반적입니다.
<div id="modal">
  <div>
    모달이 생성되는 곳
  </div>
</div>
```
> modal에서 spinner사용할때 **spinner가 modal 내부에 포함어 있지않아**서 **position 관련 style을 제대로 조작하기 어렵게** 됩니다.  
> **modal 중앙에 표시**되어야 하는데 전**체 화면 중심에 표시**된다거나 **스크롤이 되더라도 spinner는 고정**되어야 하는데 같이 **스크롤된다거나** 하는 설정이 제대로 동작하지 못하게 되는 것이 가장 큰 문제점입니다.  

## React용 Spinner 모듈을 사용하는 간단한 예
```javascript
import Loader from 'react-loader';

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
> - 렌더링하는 컴포넌트에 Loader 컴포넌트를 가져와서 사용하는 방식입니다.
> - 이런 경우 일일히 모든 페이지에 Loader 가져와서 사용해야합니다.

## 그러나 한곳에만 Loader를 포함시켜놓고 코드 중복없이 사용하고 싶습니다. 
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
#### Store를 통한 Spinner 사용
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

## Modal이나 Dialog는 최상의 엘리먼트 밖에 생성됩니다
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
> - **React App**은 id="root" 엘리먼트 하위에 생성됩니다.
> - **Modal은 App**밖에 생성됩니다.
> - 이런경우 **공통으로 사용하기 위한 Spinner**는 **모달에서 사용할 때** 모달엘리먼트 **상위나 하위에 포함된 것이 아니라서** 위치나 포지션 설정시 **원하는 위치에 설정하기 불가능한 상황이 발생합**니다.

## React Potal을 이용하여 Spinner 모듈 컴포넌트를 App밖에 생성
### Potal을 이용하여 특정 엘리먼트에 Spinner를 동적으로 위치시키는 컴포넌트
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
> - 여기서는 R**eact Portal**을 간단하게 사용하게 해주는 **NPM모듈을 사용**하였습니다.  
> - **Spinner(Loader) 모듈**은 이제 **특정 클래스명을 가진 엘리먼트에 동적으로 추가**되록 구성되었습니다.  
> - 결국은 **$('.app-root').append('<div id='spinner' style='....'>...</div>');** 와 같은 기능입니다.

#### App 컴포넌트에 추가
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

#### Store를 통한 Spinner 사용
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

#### 개별 컴포넌트들은 상황에 맞게 Spinner를 조작
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
> - 각각의 컴포넌트나 모달은 Spinner를 사용할 때 **Spinner를 위치시킬 엘리먼트의 class명을 함께 지정**합니다.
> - **Spinner**는 생성될때 **지정된 class 하위에 생성**됩니다. 
> - **App밖에 생성되는 Modal하위에도 Spinner**는 생성됩니다.
> - **Spinner**의 **position** 즉 **absolute, fixed** 설정이 **Modal을 기준으로 위치**할 수 있게 됩니다.
> - 애초에 **App 내부**에 포함하여 사용하는 방식이면 **Modal에서 position 설정의 기준이 다르기** 때문에 **맞지않게 되는 문제**가 발생하지만 **Potal을 이용하여 동적으로 Spinner를 위치** 시키므로써 가능해 졌습니다.

### API호출시에 자동으로 Spinner가 로딩 되도록 하는 팁
```javascript
import axios from 'axios';
import { store } from 'stores';

class AxiosConfig {
  initInterceptor() {
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
