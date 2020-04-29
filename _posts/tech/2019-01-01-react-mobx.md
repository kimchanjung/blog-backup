---
layout: post
title: "[Mobx] React 에서 Mobx 사용기"
description: React와 state 관리 라이브러리 mobx사용기 및 redux와 비교기
author: kimchanjung
date:   2019-01-01 16:16:01 +0900
categories: tech
---

# React에서 Mobx 경험기 (Redux와 비교기)

안녕하세요 딜리버리플랫폼팀 김찬정입니다.

## 이 글의 목적

**React와 함께 사용하는 State(상태)관리 라이브러리**중 가장 많이 사용되고 있는 **Redux**와 또 다른 라이브러리인 **Mobx**를 직접 사용하여 개발해 보고 느낀 **차이점과 Mobx만의 장점**, 그리고 **Java Spring Framework와 Mobx의 유사성**을 예제 코드와 함께 비교 해보려고 합니다.

> 많은 분들이 **React 자체 보다는 Redux를 적용** 하기 위해서 경험하는 **러닝커브가 생각보다 높아서 React가 어렵다고**들 느끼시는 것 같고 저도 공감이 되는 부분이긴 합니다. 하지만 초반 러닝커브를 극복하면 단순히 React만 사용 했을 때보다 **장점들이 분명**히 있기 때문에 State관리 라이브러리를 사용하는 것이 결론적으로 **더 낫다고 개인적**으로는 생각합니다. (이유는 이후에 설명하도록 하겠습니다.) 그런 맥락에서 **Mobx는 Redux에 비해서 눈에 띄는 강력한 장점**들이 있습니다. 용어 설명 이후에 바로 장점을 이야기 해보도록 하겠습니다.

> 이 글은 경험에 의한 지극히 주관적인 견해임을 미리 밝혀 둡니다.  
> 튜토리얼 개념으로 예제 코드를 작성한 것이 아닌, 부연설명을 위한 목적이므로 코드상 빠진 부분이 있음을 미리 밝힙니다.

## Overview

* Mobx의 장점과 특징(예제코드)
* Mobx와 Redux의 비교 (예제 코드)
* Mobx와 Java Spring의 유사성(예제 코드)
* React 개발시 효율적인 Directory 구조와 예시 (Atomic Design)
* React Component를 구분 하여 사용하는 예
* Redux와 일반적으로 같이 사용하는 라이브러리들
* React용으로 재구성된 UI 라이브러리들의 적용기 및 장단점(Material-UI, React-Bootstrap, Reactstrap)분석
* Redux Form Validation 라이브러리들의 적용기 및 장단점 분석
* Mobx Model 라이브러리들의 적용기 및 장단점 분석

## 용어 설명

### React

Javascript Web Front-End Rendering 라이브러리 중 하나

> 보통 Single Page Application Framework가 대부분의 기능을 포함 하고 있는 반면에 React는 대부분의 기능을 포함하고 있는. Framework가 아니라 View를 Rendering 하는 것이 주 기능이며 나머지 기타 기능들(router, ajax등등)은 서드파티 라이브러리를 추가적으로 사용해야 한다.

### Component

React에서 **데이터를 화면에 렌더링하는 가장 기본이 되는 단위** 라고 할 수 있겠습니다. **React.Component를 상속하는 클래스형태**의 Component와 **함수형태의 Component 두가지 형태**를 가지고 있으며 **목적에 따라 구분해서 사용**합니다. React는 작은 단위 부터 큰단위의 **Component의 조합으로 구성**되며 상단메뉴, 검색폼, 검색 데이터 그리드와 같이 Component를 분리하여 개발하고 **각각을 적절히 조합하여 하나의** Page를 구성하는 형태로 개발 합니다.

### State

React Component에서 변경가능한 데이터를 state라고 부릅니다.

> 본 포스팅에서 데이터와 State는 거의 같은 의미로 사용됩니다.

### Props

자식 컴포넌트가 부모 컴포넌트로 부터 Parameter로 받아 오는 값을 말하며 변경을 할 수 없습니다.

### Store

Global영역에서 애플리케이션의 State와 비즈니스로직을 가지고 있고 있는 주체를 Store라고 합니다.

> State를 Global한 영역에서 관리한다는 말은 즉 State관리 라이브러리 사용의 목적중 한가지 입니다.
>
> Redux에서는 State와 State를 핸들링하는 비즈니스로직을 가지고 있는 Reducer, Action등을 포함하는 의미 이기도 하지만, Mobx에서 Store는 명확히 State와 비즈니스로직을 포함하는 Class를 Store라고 부릅니다.

### Redux

Flux개념을 바탕으로한 React에서 현재 가장 많이 사용되는 State 관리 라이브러리 입니다.

### Mobx

Redux와 또 다른 **State관리 라이브러리**이며 이글을 작성하는 목적의 라이브러리입니다. 기본적으로 **객체지향 느낌**이 강하며 Component와 State를 연결하는**(Redux와 달리)** 번잡한 보일러플레이트 코드들을 데코레이터(애노테이션)제공으로 깔끔하게 해결합니다.

> 추후 예제 코드를 보면 아시 겠지만 데코레이터를 사용하는 장점이 얼마나 큰지 느끼실 수 있습니다.

### Observable

Mobx에서 Rerendering 대상이 되는 state(상태, 값)를 관찰 대상(observable value)라고 칭하며 **@observable** 데코레이터로 지정한 **State는 관찰대상**으로 지정되고 그 State는 값이 변경될 때 마다 **Rerendering**됩니다.

> 이것이 사실 Mobx가 동작하는 가장 기본 개념입니다.

### 불변성

React에서 렌더링을 할 때 판단 하는 방법은 State가 변경 되었을 때 인데 **변경전/변경후** State를 서로 비교 할 때 복잡도가 높은 객체의 경우 자식 Property까지 비교하는 것 보다 효율적인 방법으로 **State의 레퍼런스가 변경**되었을 때 변경된 것으로 간주 하고 렌더링을 합니다. 이경우 **기존 State 값을 직접 변경하는 것이 아니라**. 기존 State값을 바탕으로 변경되어 새로 생성된 객체의 레퍼런스를 **setState** 메소드를 통하여 변경하는데, 이것을 즉 **불변성**을 유지한다 라고 표현합니다.

## Mobx의 장점

### 객체지향적

보다 **객체지향적**입니다 ES6에서 추가된 Class를 이름뿐인 Class가 아니라 객체지향적으로 사용하고 개발하는 것을 권장하고 있습니다.

> 도메인모델로 분리됨으로 써 집중된 비즈니스 로직은 적절히 분산되고 도메인간의 상호작용은 message를 주고 받는 형태로 구현 할 수 있습니다.

### 서버개발자들에게 친숙한 아키텍쳐

**Java Spring Framework**와 유사한 아키텍쳐구조를 지향하고 있어 **서버개발자들에게 보다 친숙하고 낮은 러닝커브**를 제공, 장점을 그대로 적용할 수 있습니다. (흥분되는 부분 이기도 합니다)

### Decorator

데코레이터(java 애노테이션과 유사하다고 보면 된다)를 제공하기 때문에 Redux를 사용할 때 React Component와 state를 연결 하기위한 **mapStateToProps**, Redux action을 연결을 위한 **mapDispatchToProps** 그리고 **bindActionCreators**…. 등등의 **보일러플레이트 코드가 사라지고** 데코레이터가 처리하기 때문에 너무나도 깔끔한 코드가 생성됩니다.

> Redux로 개발 해보신 분이라면 느끼시겠지만 보일러플레이트 코드들의 양 만만하지 않고 또 그런 코드들을 작성하기 위해서는 어느정도 학습이 동반되어야 합니다.
>
> Redux가 어렵다가 아니라 React가 Vue보다 어렵다고 하는 이유에도 이부분도 한 몫하는 것 같습니다.

### 캡슐화

**Mobx Configuration 설정**으로 **State를 오직 메소드**를 통하여 변경할 수 있도록 **Private**하게 관리 할 수 있습니다.

> Javascript는 기본적으로 접근제어자를 제공하지 않아서 데이터 핸들링 비즈니스 로직이 펴져 버리고, 사이드 이펙트가 발생할 확률이 높고 또한 잘 관리하지 않으면 번잡스러운 코드가 생산되기 쉽습니다.
>
> 하지만 접근제어자가 없다고 해도 캡슐화를 구현할 수 있는 방법들이 있긴하지만 잘 활용되어 지지는 않습니다.
>
> Mobx는 Configuration에서 옵션 한줄로 state의 변경은 해당 클래스의 메소드를 통해서만 변경할 수 있도록 할 수 있고
>
> 도메인 모델간의 message를 통한 상호작용 코드 패턴을 유지해 나갈 수 있도록 해줍니다.

### 불변성 유지를 위한 노력이 불필요

**State의 불변성을 유지**하기 위해서 번잡스러운 코드나 **ImmutableJs**같은 라이브러리를 따로 사용할 필요가 없습니다. 이것이 왜 장점이 되냐 하면 **불변성을 유지하면 서 State를 변경하는 코드**는 Object가 Depth가 깊어지게 되면 **코드의 가독성이 매우 떨어**집니다. 그래서 **ImmutableJs** 라이브러리를 사용하게 되는데 **Redux와 같이 사용하게 될 경우** 여러가지 설정이 필요하고 추가적인 라이브러리도 필요할 뿐 만 아니라 추가적인 학습도 동반 되어야 합니다.

## State관리 라이브러리 사용 목적

### 다중 계증 컴포넌트에서 데이터와 메소드 접근의 복잡성 해결

**여러개의 Component 컴포넌트가 조합**되에 페이지가 구성된다고 할때 **Component간 상호작용** 즉 **데이터(State, Props)와 메소드**의 접근이 까다롭게 됩니다. SPA 개발이 없던 시절 서버렌더링 페이지에서 Jquery로 Dom을 조작하고 함수를 호출 할때는 **Global Scope에서 대부분 이루어져서** 크게 문제가 없었습니다. 그러나 **기능 단위의 Component**로 이루어진 최근의 SPA Framework에서는 **부모자식의 관계로 Scope**이루어져 있기 때문이 각 Component간 state와 method 접근이 복잡해 질수 있습니다. 이를 해결하기 위해서 State를 Global한 Store영역에서 관리하는 방법을 사용하여 state와 method의 접근이 용이 하게 됩니다.

### 컴포넌트에 집중된 비즈니스 로직의 분리

State관리 라이브러리 없이 **React Component로만 개발**하게 되면 거의 대부분의 **비즈니스로직이 Component에만 집중**되게 되고 코드는 점점더 스파게티화 되기 마련입니다. 하지만 State관리 라이브러리를 사용하게 되면 **Component는 Controller에 해당하는 역할**을 주로 하게 두고 나머지 로직은 적절히 분리하여 아키텍쳐를 구성할 수 있는 이점이 있습니다.

> 복잡한 페이지의 프로그램이 아니라면 사용할 필요가 없다는 의견도 있지만, 개인적으로는 실무에서 가장 기본적인 형태의 페이지라도 비즈니스로직을 분리하지 않고도 깔끔하게 코드를 유지할 정도으 규모는 보지 못해서 사용하는 편이 낫다고 생각이 됩니다. 어디까지나 개인적인 경험에서 오는 견해임을 밝혀 둡니다.

## Mobx에 앞서 Redux

> Redux에 비해 Mobx를 사용했을 때 장점을 이야기 하는 것 이지 State관리 라이브러리인 Redux를 사용하는 것 자체를 단점으로 이야기하는 것이 아닙니다. 다시 생각해도 State관리 라이브러리 없는 Component만 가지고 개발 하는 것 보다 다소 러닝커브가 있더라도 개인적으로는 State 관리 라이브러리를 사용 할 것 같습니다.

### Redux의 데이터 흐름

다음 이미지는 **Google로 'redux diagram' 키워드**로 검색했을 때 검색된 결과 들입니다. 리덕스의 개념을 설명하는데 **Data Flow Diagram**이 자주 등장 합니다. **Action, Reducer, Dispatcher, Store, View** 이런 개념들은 사실 State를 렌더링 하고 변경하기 위한 어떤 메소드 즉 서비스 같은 것을 가져다 그냥 사용하는 것 뿐인 데 개념을 장황하게 설명합니다. 그리고 그것들이 상호 작용하기 위해서 추가 해주는 보일러플레이트 코들이 매 Component마다 추가 해주어야합니다. 컴포넌트와 리덕스를 연결하기위해서 **mapStateToProps, mapDispatchToProps** 함수를 사용하고 **Action을 정의** 하고.. 등등 **Javascript의 높은 문법 자유**도 때문에 **예제 코드들의 선언 방식 또한 자유 분방**합니다. 바로 이런 것들이 React가 다소 어렵다라는 인식을 주게되는 하나의 요인 같댜는 생각이 개인적으로 들기도 합니다.

![redux-data-flow-diagram][1]

그림 1. redux diagram 키워드 검색한 결과

### Redux의 테크트리?

Redux를 사용하다보면 **redux-thunk, redux-saga, reselect** 등등 관련 라이브러리들이 등장합니다. 아래 링크는 어떤 개발자가 약간? 위트를 가미한 React + Redux를 개발하는 개발자가 겪게 되는 일련의 흐름을 포스팅 한 내용입니다. [리액트개발자가 겪게되는 길][2]

> Redux를 사용하기 위해 더 많은 라이브러리를 선택하고 사용해야 하는 고민에 빠지게 되는 점도 React가 어렵다(Redux가 어려운 것 인데.)라고 인식하게 되는 요인중 하나라로 생각됩니다.

## 본격적으로 Mobx

앞서 언급한 대로 **Mobx는 Redux와 비슷한 종류의 State관리 라이브러리**입니다. 위 언급한 Redux와는 다르게 너무나도 **간결하고 깔끔한 구조**를 가지고 있습니다.

### Mobx의 기본개념 및 특징

Mobx의 **State(데이터)의 흐름과 핵심 개념**을 간단하게 표현 해보았습니다. 사실 더 자세한 개념은 공식문서에 있지만 이해하는 데 방해가되는 개념은 제외하고 제가 이해한 부분을 간략하게 그려보았습니다. 배달 리스트를 가져오기 위해서 **DeliveryStore**(Spring의 서비스의 역할과 거의 비슷하다)의 **findAllDeliveries**를 호출하여 서버로 부터 가져온 데이터를 선언해둔 **deliveries** state에 할당 해주면 **DeliveryComponent**에서 **deliveries** 를 Rendering 하게 되는데 이것이 기본 동작 개념입니다. 기타 **Mobx Store**와 **React 컴포넌트를 연결**하는 방법은 Redux와 달리 **@inject** 데코레이더 한줄로 이루어 집니다. 물론 그외 여러가지 기능을 하는 **데코레이더 들이 제공**되고 있으며 설명은 이후 자세히 하겠습니다.

![mobx-data-flow-diagram][3]

그림 2. mobx 데이터 흐름

### Mobx의 아키텍쳐

Mobx는 렌더링 할 State를 관찰대상으로 지정, State를 변경하면 React Component Render 메소드에 의해서 Rerendering 되는 아키텍쳐를 기본 골격으로 합니다. 예제와 함께 Mobx의 작동 방식과 특징을 알아 보겠습니다. 마치 **Java Spring Framework유사한 Layer 아키텍쳐**를 가지고 있고 실제로 그런식으로 Layer를 분리하여 아키텍쳐를 구성하는 것을 권장하고 있습니다.

![mobx-spring-layer-table][4]  
표 1.spring과 mobx layer 비교

### Store = Service

**Java Spring Service와 비슷한 역할**을 합니다. 차이점 이라면 BackEnd Server의 Service(Spring)에서는 다수의 요청자에 의해서 요청 되기 때문에 **특별한 상황이 아니라면 상태를 가지고 있지 않는데** Mobx Store는 **observable한 state(상태)**를 가지고 있다는 점 입니다. Client는 사용자와 1:1 이기 때문에 서버측의 Service에서 상태와는 상황이 다릅니다. 하지만**Store는 싱글톤으로 유지 해야** 합니다. 만약 싱글톤이 아니라면 Component에 Inject된 Store는 매번 새로운 Instance가 되고 **Observable State**가 따로 생성되어 지게 됩니다. 이런 경우 상단 **메뉴 바의 크기를 변경하는 Store**를 각각 페이지에서 Inject하여 **changeMenuBarSize**라는 메소드를 통하여 **menuBarSize**라는 State를 변경한다고 하면 싱글톤이 아닐 때는 menuBarSize = '50px'라고 변경해도 menuBarSize는 실제 메뉴바의 사이즈를 가지고 실제 렌더링하는 state가 아닌 개별 Instance Store의 State일 것 입니다.
    
```javascript  
@Autobind
class RiderStore {
  @observable
  riderList = [];

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

  // 비동기인 경우 @action 대신 @asyncAction
  @asyncAction
  async *findAll(params) {
    const { data, status } = yield riderRepository.findAll(params);
    this.riderList = data.map(rider => new RiderModel(rider));
  }

  // 라이더 리스트에서 특정 요소를 제거 하는 메소드 인데 @observable로 지정된 property의
  // 변경은 store의 메소드에 의해서만 가능하다(@action decorator를 추가한)
  // 그렇게 때문에 비즈니스 로직이 여기저기 퍼져 나가는 것을 애초에 막을 수
  // 있어서 객체지향적인 코드를 유지 해 나갈 수 있다.
  @action
  removeRider(index) {
    this.riderList.splice(index, 1);
  }

  // 비즈니스 로직이 포함된 getter다 @computed 데코레이터는 반환하는 값이 변경되 었을 때
  // rerendering을 하는데 값이 변경 되었다 하더라도 변경되기 전과 같은 값이라면 불필요하게 
  // rerendering을 하지 않는다.
  @computed
  get activeRiders() {
    return this.riderList.filter(rider => rider.isActive);
  }
}

export default RiderStore;
```

코드 1.Mobx의 Store

### Repository = Repository

**Mobx Repository**는 **Ajax로 데이터를 가져오는 부분**입니다. 데이터를 가져오는 부분도 Layer를 나누어 구성하는 것을 권장하고 있습니다. **비즈니스 로직 분리**의 이점도 있지만 Test 코드 작성 시 **Mocking이 용이** 하다는 장점도 있습니다. (물론 ajax 자체를 mocking 할 수 있는 라이브러리도 있지만)
    
```javascript 
class RiderRepository {
  URL = "/v1/api/riders";

  constructor(url) {
    this.URL = url || this.URL;
  }

  findAll(params) {
    return axios.get(`${this.URL}`, { params });
  }

  findOne(riderAccountId) {
    return axios.get(`${this.URL}/${agencyId}`);
  }
}

// 싱글톤으로 리턴 (매번 새로운 객체를 생성 할 필요가 없다면 처음 부터 싱글톤으로 export)
export default new RiderRepository();
```
코드 2. Mobx Repositry Layer

### Model = Entity or Dto

**Spring의 Entity/Dto** 와 유사하다고 보면 됩니다. 도메인 로직을 Model Layer에 구성하는데 차이 점이라면 미리 필드 (property)들을 선언 하지 않아도 **Object.assign** 사용해서 **동적으로 추가**하면 되기 때문에 특별히 미리 선언할 필드가 없다면 아래 처럼 간단하게 선언 할 수 있습니다. **extendObservable**은 **Mobx가 제공하는 api**로 Object.assign 처럼 property와 값을 Target 오브젝트에 합쳐 주는데, 특징이라면 **관찰 가능한**(Rerendering 대상이 되는 값으로 만들어 추가해준다) **Property로 만들어 추가**합니다. 합치려는 객체의 Property가 **이미 선언 되어 있는 경우는 사용 할 수 없기 때문에** 그런경우는 **Mobx에서 제공하는 set api를 사용**하면 됩니다.

**미리 선언된 property가 없고 서버에서 받아온 JSON을 RiderModel로 생성하는 가장 심플한 예**
    
```javascript  
@Autobind
class RiderModel {
  constructor(data) {
    extendObservable(this, data);
  }

  // 라이더명과 지점명을 합친 getter
  // 모델 자신의 비즈니스로직을 가지고 있다. 모델 레이어가 없다면 아마도 아래 예제 처럼
  // 비즈니스 로직이 널리 퍼졌을 것이다.
  @computed
  get riderWithAgency() {
    return `${this.riderName}(${this.agencyName})`;
  }
  
  @action
  changeRiderName(riderName) {
    this.riderName = riderName;
  }
  
  // 렌더링 대상이 아니면 @computed는 필요없다.
  isActive() {
    return this.status === 'ACTIVE';
  }
}

export default RiderModel;
```

코드 3. Mobx Model Layer - extendObservable

#### 미리 선언된 Property가 있는 Model에 서버에서 가져온 JSON으로 객체를 생성하는 경우

```javascript
@Autobind
class RiderModel {
  @observable
  riderName;

  constructor(data) {
    set(this, data);
  }

  @computed
  get riderNameWithAgency() {
    return `${this.riderName}(${this.agencyName})`;
  }

  @action
  changeRiderName(riderName) {
    this.riderName = riderName;
  }
}

export default RiderModel;
```

코드 4. Mobx Model Layer - set

#### Model Layer가 없는 경우 비즈니스 로직의 집중

일반적으로 **Model Layer** 없이 JavaScript 개발을 해왔다면 아마도 이런 식으로 **Component에 비지니스 로직이 집중** 되었을 것입니다.
    
```javascript  
// 이런형태의 배열 데이터가 있다고 하자.
let riderList = [
  {
    name: '홀길동'
    age: 24,
    agencyName: '강남지점'
  },
  {
    name: '이순신'
    age: 34,
    agencyName: '송파지점'
  }
]

export default class SearchRider extends React.Component{

   componentDidMount() {
    // Component가 Mount될 때 서버에서 라이더 리스트를 가져오는 로직
    const riders = axios.get('http://www.rider.com/api/riders')
      .then(function(response) {
        response.data.map(rider => {
          rider.riderNameWithAgency = `${this.riderName}(${this.agencyName})`;
          return rider;
        })
      }
    );

    this.setState({riders});
  }
  // 특정 라이더의 이름을 변경하는 경우의 로직도 Component 메소드에..
  chanageRiderName = (riderName, riderId) =>{
    this.state.riders.map(rider => {
      if (rider.riderId === riderId) {
        rider.riderName = riderName;
      }
      return rider;
    })

    this.setState({
       riders:this.state.riders
    })
  }

  render(){
    .........
  }

}
```

코드 5. Component에 집중된 비즈니스 로직.

> 위 예제 처럼 도메인 로직이 해당 컴포넌트에 있었을 것이다 이런 것들이 한두개씩 늘어나면 Component는 그야말로 hell이다

### Observable

**Rendering 대상이 되는 State를 관찰 대상**이라고 칭하고 **@observable** 데코레이터로 **Observable State로 만들어** 줍니다. **Presentational Component**에서 값이 변경될 때 마다 값이 반영 되어 보여지게 됩니다.
    
```javascript  
@Autobind // javascript this bind를 자동으로 해주는 데코레이(arrow function 사용 없이)
export default class SearchRiderStore {
  // 라이더 리스트 state를 렌더링 할 것 이고 @observable 데코레이터를 추가하면 선언됩니다.
  @observable
  riderList = [];

  constructor(rootStore) {
    // rootStore를 통하여 다른 store(spring 서비스라고 생각하면 이해가 쉽다.)를 사용 할 수 있다.
    // rootStore.deliveryStore.findAll() <- 이런식으로
    this.rootStore = rootStore;
  }

  @asyncAction
  async *findAllRider(params) {
    const { data, status } = yield riderRepositiry.findAll(params);

    if (status === 200) {
      this.riderList = data.map(rider => new RiderModel(data));
    }
  }
}
```

코드 6. 라이더 리스트 데이터를 핸들링하는 역할을 하고 있는 Mobx Store

위에 선언된 **SearcRiderStore**에 **riderList**는 아래와 같이 렌더링 state로 사용됩니다.
    
```javascript  
// @inject 데코레이터 만으로 쉽게 riderStore를 inject한(redux와 비교하면 정말 간단하다)
@inject("searchRiderStore")
@observer // mobx observable state 를 rerendring 하기위에선언해준다
@Autobind // arrow function 없이 this를 자동으로 바인딩시켜준다.
export default class SearchRiderContainer extendsReact.Component {
  constructor(props) {
    super(props);
  }

  componentDidMount() {
    const { searchRiderStore } = this.props;
    // 컴포넌트가 마운트 되면 라이더를 가져온다
    searchRiderStore.findAllRider();
  }

  render() {
    const { riderList } = this.props.searchRiderStore;
    // 라이더 리스트를 데이블로 렌터링
    return ;
  }
}
```

코드 7. Component에서 RiderStore를 연결하고 RiderList를 렌더링

> Mobx의 Observable State가 작동하는 경우와 그렇지 않는 경우는 공식문서에 나와 있는 설명을 잘 살펴 보아야 합니다.분명히 @observable 데코레이터로 지정하고 값을 변경했는데 Rerendering이 되지 않는 경우의 케이스를 잘 파악해 놓아야 삽질을 피할 수 있습니다.

## Mobx와 Redux의 비교

이제 부터 Mobx와 Redux의 차이점을 예제 코드와 함께 비교해 보도록 하겠습니다.

### 같은 역할을 하는 두 라이브러리의 Layer(또는 라이브러리)

![mobx-redux-layer-table][5]  
표 2. Mobx와 Redux Layer 비교

### Service Layer - Store(Mobx) VS Reducer(Redux)

**Service Layer의 역할**을 담당하는 **Mobx의 Store**와 **Redux의 Reducer**를 예제 코드를 통하여 비교해 보도록 하겠습니다. 역할은 비슷하지만 선언 방식이 각각 Class, Function으로 스타일이 서로 다르고 Redux의 경우 ACTION 타입을 정의하고, ACTION을 생성하는 행위들이 추가로 들어가야 된다는 차이가 있습니다. 코드상으로는 Mobx의 Class 선언 방식이 서버개발자들에게 익숙해 보이는 모양새 입니다.

### Reducer - Redux

Redux에서 실제 **비즈니스 로직을 담당**하는 곳이라고 이해하면 될 듯 합니다.

> Redux에서 Action정의, Action생성 , Reducer생성 등이 각각 개별 파일에 하는 것이 기본 예제인데 구지? 라는 의문이 생겼고 그래서 그런지 ducks pattern 이라는 Action 정의, 생성 Reducer 생성을 하나의 파일에 하는 pattern이 있습니다. (Redux를 하다보면 계속 뭐가 나옵니다. 이런 점이 Redux가 아니라 React가 어렵다는 오래를 불러 일으킵니다.)

**ACTION TYPE 정의**
    
```javascript  
// RiderActionType.js

// ACTION 타입을 정의 한다. 정의한 액션 타입으로 액션을 생성하고
// 추후 React Component에서 action을 디스패치하면 해당하는 액션타입에
// 매핑된 리듀서가 호출된디..... 뭔가 번잡
const FIND_ALL = "rider/FIND_ALL";
const REMOVE_RIDER = "rider/REMOVE_RIDER";
const ACTIVE_RIDERs = "rider/ACTIVE_RISERS";
```

코드 8. action을 정의

**ACTION을 생성**
    
```javascript  
// RiderAction.js
// 정의된 ACTION 타입을 가지고 ACTION을 생성한다.
export function findAll(data) {
  return {
    type: type.FIND_ALL,
    payload: {
      data
    }
  };
}

export function removeRider(index) {
  return {
    type: type.REMOVE_RIDER,
    payload: {
      index
    }
  };
}

export function activeRider() {
  return {
    type: type.REMOVE_RIDER
  };
}
```

코드 9. action을 생성

**정의된 Action이 Dispatch 되었을 때 수행될 로직이 있는 Reducer**
    
```javascript    
// RiderReducer.js
// 정의된 액션션에 해당하는 비즈니스로직을 구현한다..
const initialState = {
  riderList: []s
};

function riderReducer(state = initialState, action) {
  switch (action.type) {
    case types.FIND_ALL:
      return {
        ...state,
        riderList: action.payload.data
      };
    case types.REMOVE_RIDER:
      return {
        ...state,
        riderList: state.riderList.splice(action.payload.index, 1)
      };
    case types.ACTIVE_RIDERS:
      return {
        ...state,
        activeRiders: state.riderList.filter(rider => rider.status === "ACTIVE")
      };
  }
}
```

코드 10. Reducer.

**Getter(Mobx getter와 유사한)에 해당하는 reselect를 이용한 로직**
    
```javascript  
import { createSelector } from "reselect";

const getRiderList = rider => rider.riderList;

const activeRiders = createSelector(
  [getRiderList],
  riderList => riderList.filter(rider => rider.status === "ACTIVE")
);

export default {
  getRiderList,
  activeRiders
};
```

코드 11. getter역할을 하는 reselect.

**React Component에서 Redux와의 연결**
    
```javascript    
class RiderContainer extends React.Component {
  constructor(props) {
    super(props);
  }

  componentDidMount() {
    const { riderAction } = this.props;
    console.info("onSubmitSearchRider", values);
    riderAction.findAll(values);
  }

  render() {
    const { riderList } = this.props.riderStore;
    return ;
  }
}

// Component와 Redux에 연결하기 위한 구문이 Component 아래에존재하는데. mobx는 @inject 데코레이트 하나면 연결되는데
// Redux는 명시적인 연결로직이 필요하다. mapStateToProps,mapDispatchToProps, bindActionCreators
// 의 이해와 사용 법을 알아야 하고 계속 이야기하는 javascript 문법자유도 때문에 예제들의 형식도 제각각이라 무척이나
// 학습하는데 가독성을 많이 떨어 뜨린다..
// 간단한 예제이지만 데이터와 메소드가 많아지면 복잡도는 무척이나 높아진다.
export default connect(
  // mapStateToProps
  state => {
    const rider = state.["rider"]);
    return {
      findAll: SearchRiderSelector.getRiderList(rider),
      activeRiders: SearchRiderSelector.activeRiders(rider),
    };
  },
  // mapDispatchToProps
  dispatch => ({
    riderAction: bindActionCreators(SearchRiderAction.searchRider, dispatch),
  })
)(RiderContainer);
```

코드 12. React Componet와 Redux의 연결.

> 개인적으로 느낀 아쉬운 점을 나열해 보자면
>
> 1. 어떤 행위를 하기위한. **하나의 메소드를 정의 하기위하여 3가지** 즉 ACTION 타입정의, ACTION생성, REDUCER 생성 해줘야 하는 것이 **조금 번잡 스럽다는 느낌**을 받았다.
> 2. 실제 비즈니스 로직이 들어가는 부분의 예제가 swich case문으로 되어 있어 생소한 느낌이 많이 아쉬었다. (실제로 redux를 사용해서 개발 할때는 사실 메소드를 분리해서 했다.)
> 3. **Javascript의 문법적 자유도가 높은 특징** 때문에 다양한 방식으로 선언된 예제들을 보면 혼란이 가중되고 코드 가독성이 떨어지는 느낌을 받았다.
> 4. 그런 불편 때문에 보일러 플레이트를 줄여주는 단순한 기능을 하는 라이브러리나, Util이 많이 존재하는데 Action을 생성 해주는 **createActions**, Action과 Reducer를 매핑해주는 **handleActions** Component와 Redux연결을 위한 **mapStateToProps**, **mapDispatchToProps**, **bindActionCreators** getter로직의 성능을 위해서 **reselect**(mobx @computed 처럼 값이 변경되었다 하더라도 동일한 값이면 불필요하게 다시 렌더링하지 않는다)등등 이런 라이브러리등을 선택하고 학습하는 것 또한 Redux러닝 커브를 높이는데 한 몫을 하는 것 같다

### Store - Mobx

Mobx의 **Store**는 **Java Spring의 Service와 유사**한 역할을 하며 내부 상태가 존재한다는 것이 차이점이라고 할 수 있습니다. Mobx의 **상태(State)**는 **@observable(관찰가능한)**로 지정하여 React Component에서 렌더링될 state로 사용합니다. React Component에서 사용할 Mobx Store의 Dependency Injection은 **@inject** 데코레이터 하나로 끝나는데 Redux에서는 **mapStateToProps, mapDispatchToProps**, 등의 함수로 **연결 해줘야하는 번잡스러운 보일러플레이트 코드**에 비해서 너무나도 간결하고 가독성도 뛰어납니다.
    
```javascript  
@Autobind
class RiderStore {
  @observable
  riderList = [];

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

  @asyncAction
  async *findAll(params) {
    const { data, status } = yield riderRepository.findAll(params);
    this.riderList = data.map(rider => new RiderModel(rider));
  }

  @action
  removeRider(index) {
    this.riderList.splice(index, 1);
  }

  @computed
  get activeRiders() {
    return this.riderList.filter(rider => rider.status === "ACTIVE");
  }
}
```

코드 13. Mobx Store.

**React Component에서 Mobx Store를 Inject하는 방식**
    
```javascript  
// inject 데코레이터로 riderStore를 inject 한다. reactcomponent의 props으로 접근할 수 있다.
// observer는 mobx가 @observable로 지정된 state를 적절히rerendering시킨다
@inject("riderStore")
@observer
class RiderListContainer extends React.Component {
  fetchRiderList() {
    const { riderStore } = this.props;
    riderStore.findAll({ page: 0 });
  }

  render() {
    const { riderList } = this.props.riderStore;
    return ;
  }
}
```

코드 14. React Component에서 Mobx Store를 Injection.

### Repository Layer - class(Mobx) VS redux-thunk 또는 redux-saga(Redux)

Api 호출같은 비동기 Action의 경우 Redux에서는 **redux-thunk라는 라이브러리를 주로 사용** 하다가 최근에는 **redux-saga**라는 라이브러리로 옮겨가는 추세입다. 특히나 redux-saga는 **es6 generator**를 사용하여 **callback 메소드 없이** 사용할 수 있는 장점 때문에 옮겨 가는 추세이긴 하지만 사용방법 자체는 **러닝커브도 있는 편**이고 익숙하지 않는 형태입니다. 그에 반에 Mobx는 Ajax call 하는 메소드를 Class로 정의 해놓고 Store에서 **async/await**, 또는 **generator** 를 사용하여 **callback 없이 사용**하면 됩니다.

> 별도의 추가 라이브러리를 사용할 필요가 없으며 문법적인 깔끔함을 유지하기 위해서 **@asyncAction**데코레이터, flow 같은 함수도 제공하기 때문에 제법 **깔끔한 코드**로 작성이 된다.

### Redux-Saga - Redux

비동기 Action을 처리하기위한 라이브러리인 Redux-Saga를 사용하여 비동기 Action을 구현 합니다. ES6 Generator를 사용하기 때문에 Callback Method 없이 구성이 가능한 장점이 있습니다. Redux-Saga 역시 Action이기 때문에 Action Type, Action을 생성해 주어야 합니다.

> Redux-Thunk 에서 Redux-Saga로 넘어가는 추세로 보이는데 혹자는 Redux 보다 Redux-Saga의 러닝커브가 더 높다고 이야기합니다. (Redux에 딸려오는 식구들이 왜 이렇게 많은지…)

**ACTION TYPE 정의**
    
```javascript   
// RiderActionType.js
// ACTION 타입을 정의 한다. redux-saga의 어떤 행위도 ACTION정의하고 ACTION을 dispatch하는 형태는 마찬가지다
const FETCH_RIDER_LIST = "rider/FETCH_RIDER_LIST";
```

코드 15. Redux-Saga Action Type 정의

**ACTION 생성**
    
```javascript   
// RiderAction.js
// 정의된 ACTION 타입을 가지고 ACTION을 생성한다.
export function fetchRiderList(data) {
  return {
    type: type.FETCH_RIDER_LIST,
    meta: { method: "get", url: "/v1/api/riders" },
    payload: {
      data
    }
  };
}
```


코드 16. Redux-Saga Action 생성.



**Redux-Saga Async Method**
    
```javascript   
// RiderSaga.js
/**
 * call, put, all, takeEvery 같은 redux-saga에서 제공하는 api를 학습해야 이해 할 수 있다.
 */
function* fetchRiderList({ payload, meta }) {
  const { data, status } = yield call(Axios.request, {
    ...meta,
    params: payload
  });

  if (status === 200) {
    yield put(actions.rider.fetchSuccess(data));
  } else {
    yield put(actions.rider.fetchFail(data));
  }
}
/**
 * types.FETCH_REQEUST <- 이 ACTION이 dispatch되는 것을 감지하고 있다가 dispatch 되면
 * getRiderListSuccess  task를 실행한다
 */
export default function* watchSearchRiderSaga() {
  yield all([takeEvery(actions.searchRider.fetchRequest, fetchRiderList)]);
}
```

코드 17. Redux-Saga Async Method 생성.

**Redux-Saga에서 받아온 API 데이터를 처리할 Reducer**
    
```javascript 
// RiderReducer.js
// riderList JSON을 처리할 reducer를 정의하고 redux-saga에서비동기 API를 호출 후
// 받아온 데이터를 fetch할 ACTION을 dispatch한다(중간 단계가 너무많다... 익숙하지 않은 스타일로..)
const initialState = {
  riderList: []
};

function riderReducer(state = initialState, action) {
  switch (action.type) {
    case types.FETCH_SUCCESS:
      return {
        ...state,
        riderList: action.payload.data
      };
    case types.FETCH_FAIL:
      return {
        ...state,
        isError: true
      };
  }
}
```

코드 18. Redux-Saga 에서 받아온 데이터를 처리할 Reducer.

### API 호출을 담당하는 Class - Mobx

Mobx에서는. **API호출을 담당하는 Repository 성격의 Class**를 만들고 Mobx Store에서 async 또는 generator를 사용해서 간단하게 callback 메소드 없는 패턴으로 구현되기 때문에 가독성도 좋습니다. **Redux 처럼 Redux-Saga, Redux-Thunk** 같은 추가 라이브러리는 필요하지 않습니다.
    
```javascript    
class RiderRepository {
  URL = "/v1/api/riders";

  constructor(attr) {
    Object.assign(this, attr);
  }

  findAll(params) {
    return axios.get(this.URL, { params });
  }

  findOne(riderAccountId) {
    return axios.get(`${this.URL}/${riderAccountId}`);
  }
}
// 싱글톤으로 리턴
export default new RiderRepository();
```

코드 19. Mobx Repository Layer.

**Mobx Store에서는 Callback Method 없이 사용**
    
```javascript  
@Autobind
class RiderStore {
  @observable
  riderList = [];

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

  // promise를 반환하는 riderReposiotry.findAll 메소드를 @asyncAction추가하고 generator를 사용하면
  // 콜백없이 마치 sync한 로직처럼 코드를 작성할 수 있어 가독성이 높다.
  @asyncAction
  async *findAll(params) {
    const { data, status } = yield riderRepository.findAll(params);
    this.riderList = data.map(rider => new RiderModel(rider));
  }

  @action
  removeRider(index) {
    this.riderList.splice(index, 1);
  }

  @computed
  get activeRiders() {
    return this.riderList.filter(rider => rider.status === "ACTIVE");
  }
}
```

코드 20. Mobx Store에서 Repository 를 사용하는 예.

### Model Layer - class(Mobx) VS Redux는 object 리터럴

Redux에서는 State를 Model layer로 구성하는 아키텍쳐를 권유하는 개념이 아닌 듯 합니다. 학습을 하다보면 Redux는 함수형프로그래밍, 불변성, 사이드 이펙트에 대한 이야기가 자주 나오는데, 설계적으로 나쁜 접근은 아닌듯 하나(오로지 개인적인 생각일 뿐입니다.) 쉽지는 않습니다… **Mobx에서는 객체지향적인 아키텍쳐를 권유** 하고 있고 State를 객체리터럴을 사용해서 단순하게 구성하는 것 보다는 **Model Class를 선언해서 도메인 자신의 로직은 자기가 가지고 있도록** 하여 비즈니스로직이 철저하게 분리되도록 권장하고 있습니다.

### Redux 불변성 State

Redux에서는 따로 Model Layer를 구성 하는 형태는 아니고 보편적인 **객체 리터럴 형태**를 그대로 사용 하고 있습니다. 그리고 불변성 유지를 위해 **스프레드 분법**을 사용해 변경된 데이터와 변경되지 않은 데이터를 가지고 적절히 잘 합쳐서 새 객체를 생성 합니다.

```javascript  
// 초기 값을 지정해 놓은 state, 그냥 단순한 객체 리터럴이다 자신의값을 처리하는 비즈니스 로직을 가진 메소드는
// 없다 그냥 값만 있을 뿐
const initialState = {
  isLoading: false,
  isOpent: false,
  riderList: []
};

// 리듀서에서는 불변성 유지를 위해서 기존 state를 변경하는 것이 아닌 새변경한 값으로 객체를 생성한 state를 리턴한다
// 변경하지 않는 값들은 스프레드로(...) 객체를 풀어서 넣어주고 변경할값들은 변경해서 원래 state와 같은 모양으로 만든다
function riderReducer(state = initialState, action) {
  switch (action.type) {
    case types.FETCH_SUCCESS:
      return {
        ...state,
        riderList: action.payload.data
      };
    case types.FETCH_FAIL:
      return {
        ...state,
        isError: true
      };
  }
}
```

코드 21. Redux 불변성 State.

**Redux 불변성 state를 편하게 하기 위한 ImmutableJs라이브러리를 사용한 예**
    
```javascript  
// immutablejs의 fromJS api를 사용하여 불변성 객체로 만든다.
const initialState = fromJS{
  isLoading: false,
  isOpent: false,
  riderList: []
};

// set api로 riderList의 값을 변경해주고 return하면 기존 값을변경 리턴하는 것이 아니고 기존값은 놔두고 새로운 변경된 state를return한다
// 비교적 스프레드 문법을 이용한 방법 보다 직관적이다
function riderReducer(state = initialState, action) {
  switch (action.type) {
    case types.FETCH_SUCCESS:
      return state.set('riderList', action.payload);
    case types.FETCH_FAIL:
      returnstate.set('isError', false);
  }
}
```

코드 22. Redux에서 ImmutableJS 적용

### Model Class - Mobx

Java Spring에서 DTO 선언하는 것과 별반 다르지 않습니다. 차이점이라면 Javascript 특성상 **프로퍼티를** 미리 추가 하지 않고 동적으로 추가할 수 있어서 그냥 **동적으로 추가 한것 뿐** 도메인 클래스 선언이 크게 차이나 보이지 않습니다. 도메인 자신의 **값과 자신의 값을 핸들링하는 메소드**를 가지고 있는 형태로 구성됩니다.

```javascript  
@Autobind
class RiderModel {
  constructor(data) {
    // Object.assign과 유사한 mobx가 제공하는 api를 사용하여 @observable(관찰가능한 state, rendering 되는) state로 만들어
    // RiderModel에 멤머 property로 추가해준다.
    extendObservable(this, data);
  }

  // 라이더명과 지점명을 합친 getter
  // @computed는 값이 변경되도 이전 값과 값이 같으면 불필요한 렌더링을 하지 않는다.
  @computed
  get riderWithAgency() {
    return `${this.riderName}(${this.agencyName})`;
  }

  @action
  changeRiderName(riderName) {
    this.riderName = riderName;
  }
}
```


코드 23. Mobx Model Layer

**서버에서 가져온 데이터를 RiderModel클래스로 생성**

> 마치 모양세가 Java Spring Service에서 **Repository에서 가져온 Entity를 stream을 이용해서 DTO로 변환하는 과정**과 별반 다를게 없다. Spring 개발자라면 Redux에 비해서 친숙 하게 다가 올 수 밖에 없는 것 같다.

```javascript
@Autobind
class RiderStore {
  @observable
  riderList = [];

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

  @asyncAction
  async *findAll(params) {
    const { data, status } = yield riderRepository.findAll(params);
    this.riderList = data.map(rider => new RiderModel(rider));
  }
}

export default RiderStore;
```

코드 24. Mobx Model 생성

## Mobx와 Java Spring의 유사성 비교

### Service Layer - Mobx Store, Spring Service

Mobx의 Layer와 Spring Layer의 비교 입니다. 언어적인 특성이 다르기 때문에 약간의 차이는 존재하지만 구현 해놓은 모양새는 크게 다르지 않고 비슷한 형태를 가지고 있습니다.

### Spring Service

```javascript
@Transactional
@Service
public class RiderServiceeImpl implements RiderService {
    @Autowired
    private RiderRepository riderRepository;

    @Override
    public List findAll(RiderSearchRequest request) {
        return riderRepository.findAll(reqeust).stream()
                .map(RiderDto::of)
                .collect(Collectors.toList());
    }

    @Override
    public List activeRiders() {
          return riderRepository.findAll(reqeust).stream()
                .filter(v -> v.isActive())
                .map(RiderDto::of)
                .collect(Collectors.toList());
    }
}
```

​    

코드 25. Spring Service

### Mobx Store

```javascript
@Autobind
class RiderStore {
  @observable
  riderList = [];

  constructor(rootStore) {
    this.rootStore = rootStore;
  }

  @asyncAction
  async *findAll(params) {
    const { data, status } = yield riderRepository.findAll(params);
    this.riderList = data.map(rider => new RiderModel(rider));
  }

  @computed
  get activeRiders() {
    // rider.isActive는 아래 소스 예제에서 Rider Model Class에서 선언된 메소드를 호출 하여 사용한다.
    // Rider Model의 비즈니스 로직은 자기가 담당한다.
    return this.riderList.filter(rider => rider.isActive);
  }
}
```

코드 26. Mobx Store

### Repository Layer

### Spring Repository

```java
public class RiderRepositoryImpl extendsQueryDslRepositorySupport implementsRiderRepositoryCustom {
    private QRider rider = QRider.rider;

    public RiderStartCashOnHandRepositoryImpl() {
        super(Rider.class);
    }

    @Override
    public List findAll(RiderSearchRequest request) {
        return from(rider)
                .where(rider.riderName.eq(request.getRiderName()))
                .fetch();
    }
}
```

코드 27. Spring Repository

### Mobx Repository class

```javascript
class RiderRepository {
  URL = "/v1/api/riders";

  constructor(attr) {
    Object.assign(this, attr);
  }

  findAll(params) {
    return axios.get(this.URL, { params });
  }
}
```

코드 28. Mobx Repository Layer

### Model Layer

### Java DTO

```java
@Getter
public class RiderDto {
    private Long id;
    private String riderName;
    private Status status;
    private String agencyName;

    private RiderDto() {}

    public static RiderDto of(Rider rider) {
        RiderDto instance = new RiderDto();
        instance.id = rider.getId();
        instance.riderName = rider.getRiderName();
        instance.status = rider.getStatus();
        instancee.agencyName = rider.getAgencyName()
        return instance;
    }

    public String riderWithAgency(){
        return String.format("%s(%s)", this.riderName, this.agencyName);
    }

    public boolean isActive(){
        return this.status == Status.ACTIVE
    }
}
```

코드 29. Java DTO

### Mobx Model Class

```javascript
@Autobind
class RiderModel {
  /**
  생성자에서 서버로 부터 받아온 JSON에 서 property와 value 추가 해버를 것 이므로  property를 미리 선안 안해도 관계는 없다.
  id;
  riderName;
  status;
  agencyName;
  */
  constructor(data) {
    extendObservable(this, data);
  }

  @computed
  get riderWithAgency() {
    return `${this.riderName}(${this.agencyName})`;
  }

  // 내부 처리시 상태값을 확인하는 메소드임으로
  // ReactComponent에서 redering 할일이 없는 값이라면 구지 @computed를 붙일 필요는 없다
  get isActive() {
    this.status === "ACTIVE";
  }
}
```

코드 30. Mobx Model Class

## React로 Project를 구성 할 때 하는 고민들

올해 중순 즈음 우리 팀의 **다음 Front-End Framework를 어떤 것을 사용할지에 대한 니즈**가 있었고 그 **임무가 나에게 떨어졌습니다. 면밀힌 조사**를 통하여 React로 해보기 로 했고, 실제 실무에서 사용하기 위해서는 어느 정도 까지 기반이 갖춰져 있는 수준까지 개발이 필요했기 때문에 같은 팀 **구인본님과 함께 React + Redux 조합의 P.O.C**를 진행 했었습니다. 그리고 최근 **Redux 보다 Mobx**가 장점이 많다는 것을 알게 되었고 개인적으로 **휴가와 주말을 거의 집근처 카페에서 보내며 새로 React + Mobx + Material-UI 조합**으로 tutorial 수준이 아닌 실제 사이트 수준의 개발을 진행 했습니다.

> 단순히 아웃풋을 빨리 뽑아내는데 집중한 것이 아니라 NPM에 넘쳐나는 React 관련 라이브러리들과 패턴을 관련 블로그나 문서들을 통하여 비교 조사하고 적용 해보면서 최상의 패턴과 라이브러리를 조합하는데 더 집중 하였습니다.

**Mobx와 Material-UI를 적용한 배달현황** ![brms-dashboard][6]  
그림 5. mobx와 material-ui를 배달현황

**Mobx와 Material-UI를 적용한 라이더관리** ![brms-search-rider][7]  
그림 5. mobx와 material-ui를 라이더 관리

이를 통하여 **마주한 고민**과 더불어 실제 **개발 해보면서 느낀 것** 들과 약간의 정보 그런 이야기를 한번 해보고자 합니다.

> 아마도 처름 React를 시작 하시는 분들이라면 이런 고민들을 마주 하게 될 가능성이 높다

### 효율적인 Directory 구조

보통 어느 프로젝트나 디렉토리를 구분할 때 **파일 종류별**로 아니면 **페이지별(관련이 높은 파일들)**로 디렉토리를 구성해 놓는 **2가지 방식을 많이 사용할 텐데** 별 것 아니지만 초반에 고민이 되이서 리서치를 해본결과 우선 **기본적으로는 페이지별**로 디렉토리를 나누어 관리하고 **공통적인 React Component**는 [atomic design][8]라는 **UI design 개념**을 도입하여 프로젝트 폴더 구조로 가는것이 Component 기반인 **React에서 아주 효율적인 구조**라고 판단 되서 적용 해보았습니다.

> 프로젝트의 폴더구조는 정답이 없어서 취향에 맞게 선택 하시면 될듯 합니다.

### Atomic Design 적용 예

간략한 특징은 다음과 같이 Atoms(원자), Molecules(분자), Organisms(유기체), Templates, Pages 분류로 React Component의 종류와 규모에 맞게 적절히 분류해서 생성하여 관리합니다.

**Atomic Design 단계 구분** ![atomic-design][9]  
그림 5. Atomic Design 단계구분

**Atomic Design 적용한 폴더 구조** ![atomic-design-folder][10]  
그림 5. Atomic Design 폴더 구조

### React Component의 유형 구분

기본적으로 React로 개발시 **2가지 형태의 Component를 생성** 할 수 있습니다. **React Component를 상속**받은 Class로 Component를 작성하는 방법과 **React Component를 상속받지않은 순수 함수 형태**로 작성하고 Props만 받아서 구현하는 방법이 있는데 두 형태의 차이는 **상속받은 Class는 내부적으로 state(변경하는 가능한 값), props(변경하지 못하는 값), react lifecycle method**등등으로 구성되어 있고 두번째 형태는 **내부에 state와 lifecycle 메소드가 없고 Props만 받아서 구현**하는 함수 형태입니다. 그럼 이 **두가지를 어떤 경우에 적절히 사용해야 할지 애매한 상황**이 생기는데 현재 facebook react 팀에 있고 redux를 개발한 **Dan Abramov는 두가지 경우로 구분하고 사용하기를 추천** 하고 있습니다. [원문링크][11] 위 개념을 바탕으로 실제 개발시에 **저의 경우는 대략 3가지 정도의 유형으로 구분** 해서 사용하고 있습닏다.

> React는 기본적으로 명확하게 두 영역을 구분해서 설계된 것이 아니기 때문에 Tutorial에서는 구분해서 코드를 설명하지는 않는다. 하지만 이런 식으로 두 영역을 명확하게 구분하여 개발하는 것이 좀더 구조적으로 유용하기 때문에 구분하는 것을 권장하고 있다.

**Component 유형구분** ![component-type][12]  
그림 5. React Component의 유형

**Container Component** React Component 를 상속한 RiderSearchContainer는 State와 Method를 적절히 중간에서 연결해 주는 **Controller와 같은 역할**만 수행하고 HTML 렌더링은 RiderListTemplate 같은 Presentational 영역을 담당하는 Component가 담당한다.
    
```javascript  
@inject('riderStore')
@observer
class RiderSearchContainer extends React.Component {
  // 검색버튼 클릭 시 수행 될 메소드(RiderSearchFromTemplate에서 사용)
  fetchRiderList() {
    const { riderStore } = this.props;
    riderStore.findAll({ page: 0 });
  }
  // 상세보기 클릭 시 수행 될 메소드(RiderListTemplate에서 사용)
  fineOneRider(e) {
    const { riderStore } = this.props;
    const { riderAccountId } = e.currentTarget.dataset;
    riderStore.findOne(riderAccountId);
  }
  
  render() {
    const { riderList } = this.props.riderStore;
    return (
     	
    );
  }
}
```

코드 31. Container Component

**Presentational Component**  
**React Component 를 상속하지 않은 순수한 Function**으로 Component를 작성하고 내부에 state를 생성할 수 없고 RiderSearchContainer가 Parameter로 넘긴 State와 Method를 받아서 **state는 렌터링 하고 method는 사용**하는 역할만 수행한다.
    
```javascript 
function RiderListTemplate(props) {
  const { riders, riderDetail } = props;
  return (
    <ul>
      <For each="rider" of={riders}>
        <li data-rider-account-id={rider.accountId}>{rider.id}</li>
        <li>{rider.name}</li>
        <li>{rider.type}</li>
      </For>
    </ul>
  );
}
```

코드 32. Presontational Component

**Component**  
공통으로 사용할 지점 리스트를 서버에서 가져와 SelectBox를 구성한 Component이다. 내부 비즈니스 로직은 별로 없고, 서버로부터 지점 리스트를 가져와 option들을 생성하는 로직만 존재하고 **Presentational을 따로 구분하여 개발 할 정도로 View영역 HTML 코드가 많지 않아** 이런 경우에는 구지 Layer를 나누지 않고 그야말로 Component로 구성해서 사용했습니다.

> Vue나 Angular 의 directive와 거의 같다고 보면 이해가 빠를 것 같다.

```javascript
class AgencySelectBoxComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      agencies: [],
    };
    this.getAgencyList();
  }

  async getAgencyList() {
    const { data, status } = await axios.get('/v1/api/agencies');

    if (status === 200) {
      this.setState({ agencies: data.agencies });
    }
  }

  render() {
    const { agencies } = this.state;
    return (
      <select name="agencyId">
        <For each="agency" of={agencies}>
          <option key={agency.id} value={agency.id}>
            {agency.name}
          </option>
        </For>
      </select>
    );
  }
}
```
코드 33. Component

### React용으로 재구성된 UI 라이브러리 선택

React용으로 재구성된 여러 UI 라이브러가 있고 Front-End 특성상 css style만 관련이 있는 것이 아니라 **UI 표현과 동작이 React 코드와 밀접하게 상호 작용**을 하고 특히 Redux나 Mobx와 함께 사용하는 경우 관련 라이브러리들 (Form Validation 라이브러리, Router 라이브러리 등등)과 밀접하게 연관이 있기 때문에 각 라이브러리간의 궁합과 이슈들이 어느정도 해결되어 안정적으로 관리되는 라이브러리 인지가 매우 중요한 요소입니다.

> Angular 1.x 때 경험에 비춰 보면 Angular용 Bootstrap 라이브러리의 Popover를 사용 했을 때 **Popover를 클릭하면 화면 모서리**에서 생성되어 **클릭지점으로 Popover가 이동되어 위치**되어 버리는 버그가 있었고 **버그가 픽스된 버전**은 **Angular 1의 마지막 버전과 Dependency**가 있어서 Angular의 버전을 올려야 했다. **Angular 마지막 버전과 사용하던 버전에서 변경된 Feature들이 많아** 프로젝트의 **모든 코드(Popover이슈와 직접적인 연관이 없는)를 수정**할 수 밖에 없었다… 이와 유사한 경우가 생겨버리면 프로젝트를 뒤엎는 수준의 리펙토링이 필요하거나 다른 라이브러리로 재개발을 하는 최악의 상황이 발생 할 수 도 있다.

적용해본 UI 라이브러는 각각 **Reactstrap(bootstrap4), React-Bootstrap(Bootstrap3), Material-UI(Google Material Design)**이고 사용해본 소감을 정리 해보고자 합니다.

#### Reactstrap(Bootstrap4)

**장점**  
최신 bootstrap 4를 react용으로 사용하도록 개발 된 라이브러리 이며 **bootstrap 4의 최신 style**을 적용할 수 있다.

**단점**  
React-Bootstrap(bootstrap3)보다 **미해결된 이슈**들이 남아 있었고 Redux-Form(form validation라이브러리)과 함께 사용시 몇 가지 **이슈로 인하여 component를 커스텀 해야하는 경우가 있는 경우**가 있는 것이 단점이다.

> 사용 시점이 2018년 5월 즈음이라 적용하고자 한다면 현재 상황을 살펴보는 것을 당부 드린다.

#### React-Bootstrap(Bootstrap3)

**장점**  
이슈들이 많이 해결되고 어느정도 성숙된 라이브러리

> 다른 라이브러리들(react-router-bootstrap,redux-form..)이 react-bootrap을 기본으로하고 개발된 경우가 많아  
> reactstrap보다 이슈가 적었다.

**단점**  
Bootstrap 4가 나온 상황에서 **style이 좀 예전 것**이라는 점

#### Material-UI

개인적으로 Mobx로 개발을 진행 했을 때 Material-UI도 같이 적용 했습니다. 사실 처음에 **우려되는 점이 다른 라이브러들과의 궁합**을 걱정했지만 이제 정식으로 3대 버전까지 개발되었으며 **Mobx-React-Form(form validation 라이브러리 with mobx)이 Material-UI를 지원**하기 때문에 input의 **유효성 검사와 그에 따른 UI의 변화와 오류 메시지 표시**등이 별다른 custom 작업없이 Style이 틀어지거나 하지 않고 자연스럽게 작동했고, 사용율도 Bootstrap 기반의 라이브러리들보다 훨씬 높은 수치를 나타내고 있습니다.

**최근 NPM 경향** ![npm-trend-ui-lib][13]  
그림 5. npm trend ui 라이브러리

**장점**

1. **Web, Mobile, Desktop**에 까지 광범위하게 사용되고 있는 **Google Material Design의 깔끔한 style**을 적용할 수 있다.
2. 테마변경의 경우 **css, sass를 새로 변경 rebuild**하는 것이 아니라 아주 **간편하게 코드상**으로 **변경**할 수 있어서 **Dark Mode** 및 **색상테마**를 사용자가 **직접 설정해서 변경 할수 있도록 기능으로도 제공** 할 수 도 있다.   
![brms-dashboard-dark][14]  
그림 5. material-ui dark mode

3. css를 적용하는 방식이 아닌 [**jss][15] 방식으로 style을 해당 페이지에서 사용하는 스타일만 따로 격리**된 javascript객체로 관리가능 하고 **동적으로 style 변경 적용이 직관적**이다
4. 우리가 모르는 사이 많이 적용되어 있던 **Google Material Design**이라 제공하는 컴포넌트로 **항샹된 UX**로 적용할 수 도 있다.

**단점**

1. bootstrap에 많이 적응된 사람이라면 Material-UI 적용을 위해서는 **약간의 학습**이 요구된다. 

> 처음 학습기간을 투자 한 이후로는 그다음에는 눈에 익어서 단계별 학습이 더 요구 되지는 않았다.

### Redux와 함께 주로 사용되는 라이브러리들

Redux만 npm에서 다운받으면 되는 줄 알았는데 **이게 다뭐야 하시는 분들**이 분명히 계실 것 이라고 생각됩니다. 리서치 하다보면 **꼬리에 꼬리를 무는 수많은 추가 라이브러리**들과 그 것들이 뭐하는 라이브러리인지 파악하는 일도 상당한 리소스가 투여되는 경험이였습니다. 일단 Redux를 사용 할 것이라면 2018년 5월 기준으로 보편 아래 나열한 키워드의 라이브러리들을 기본적으로 사용하면 됩니다.

**Redux와 관련 라이브러리**  
![redux-with-lib][16]  
그림 5. Redux와 관련 라이브러리들

> 물론 추가적인 조사는 필요합니다. 다른 라이브러리가 나왔을 수도 있기 때문에..

### Redux와 연동 하는 Form Validation 라이브러리들

**Form Validation 없이** 한땀 한땀 Dom을 Select하여 **값을 정규식과 비교**해서 일치하지 않는 경우 Submit를 중지하고 **UI 컴포넌트에 css class를 변경**하고 **오류메시지를 추가**하고 하는 작업은 정말 지겹고 **많은 라인의 보일러플레이트 코드를 양산**합니다. input 항목이 많으면 코드량도 만만치 않아서 **Form Validation 라이브러리의 사용은 개인적으로 꼭 필요**하다고 생각하는 편입니다. **실제로 적용 해본 라이브러리**는 **Redux-Form, React-Redux-Form** 두가지 이고 나머지 라이브러리들의 문서를 면밀히 검도 해본 리서치 결과 입니다. Redux와 함께 적용을 고려 해보시는 분들이라면 참고 하시면 좋을 듯 합니다.

#### [Redux-Form][17]

보편적으로 **Redux와 함게 가장 많이 사용하는 라이브러리**, 기능과 성숙도가 높으나 러닝커브가 조금 있습니다.

> 결론적으로는 Redux-Form 개발하다가 React-Redux-Form을 따로 적용 해보았고 2% 부족함을 느껴 결론적으로는 Redux-Form을 사용했습니다.(Redux로 개발 시)

#### [React-Redux-Form][18]

일단 문서가 깔끔하고 redux-form에 비하여 깔끔한 설정 때문에 코드를 구현 해보았습니다. 하지만 최종적로 몇가지 기능의 코드 패턴이 매우 번잡스러워 결국은 Redux-Form으로 돌아 왔습니다.

**편의성 3/5  
문서 4/5  
커스텀 스타일 : 가능  
코드구성 복잡도 : 3/5**

#### Redux-Form 과 비교

**장점**  
Redux 와 연결 작업 코드 구분은 Redux-Form에 비해 약간 깔끔, 기본적으로 Field에 설정하는 prop들이 Redux-Form에 비해 약간 간결한 느낌이였습니다. 문서는 Redux-Form보다 훨씬 잘되어 있다고 느겼습니다.

**단점**  
**Validation 구현 아주 깔끔하지 않고 번잡 스러움** (함수로 바로 구현 하거나 , 오류 메시지 부분도 따로 명시적으로 구현), Redux-Form이 더 깔끔, redux-form 은 validation 관련 boolean 과 메시지를 prop으로 제공해줘서 그냥 명시하면되는데 **React-Redux-Form은 validation 이후 error 메시지, error class 처리를 하기 위한 자유도가 낮고** 구현 하려면 번잡스럽게 구현(제일 큰단점), **redux store에 저장된 form state 접근이 redux-form 이 제공하는 것 보다 엄청 복잡**, state에 form의 값만 있는게 아니라 엄청 많은 값들이 들어 가 있는데 그걸 보고 작업자가 직접 추려야 ,됨 반면 redux-form은 다양한 selector를 제공해서 원하는 값을 딱 추려서 가져올수 있습니다.

**총평**  
React-Redux-Form이 Redux-Form을 불편한 기능을 해결해 줄 것을 기대 하였으나 **redux와 연결, fileld의 설청 prop들 몇가지 선언 구문은 깔끔**해 졌지만 **validation 처리를 깔끔하게 처리하는 기본 방법이 제공 되지 않는 것**이 제일 큰 **단점**으로 작용하고 오히려 **자동화된 기능이 redux-form보다 더 부족함**다는 느낌을 받았습니다. (많은 기대를 했는데 ㅠㅠ).

#### [Formik][19]

가벼운 라이브러리라고 표방하지만 validation이 각필드에 정의 되는 방식이 아니라 모든 필드의 validation을 한꺼번에 따로 설정하는 방식이라 **너저분한 방식의 느낌이 강함하고 문서가 가독성이 떨어지는 스타일로 제공**되고 있습니다.

**편의성 3/5  
문서 2/5  
커스텀 스타일 : 가능  
코드구성 복잡도 : 4/5  
redux-form 과 비교 : 장점이 낮음**

#### [Formsy-React][20]

Validation을 각필드에 선언 할수 있지만 그외 **error 메시지 표시, 필수여부 onClick onChange, onSumit같은 조작들은 한땀 한땀** 해줘야 하는 방식이라 library에서 제공하는 것이 많이 없음, 문서가 github 문서 뿐입니다.

**편의성 2/5  
문서 2/5  
커스텀 스타일 : 가능  
코드구성 복잡도 : 3/5  
redux-form 과 비교 : 장점이 낮음**

#### [React-Form][21]

Validation을 각필드에 구성하는 방식이지만, 나머지 **event 및 오류 표시는 직접 해야됨** 코스 스타일 복잡도도 그렇게 낮은 편은 아닙니다.

**편의성 2/5  
문서 3/5  
커스텀 스타일 : 가능  
코드구성 복잡도 : 3/5  
Redux-Form 과 비교 : 장점이 낮음**

### Mobx 연동 하는 Form Validation 라이브러리

Mobx 와 연동하여 사용하는 **Form Validation 라이브러리**는 [Mobx-React-Form][22]이 가장 보편적인데 **Material-UI와 연동에서도 별도의 커스텀 작업이 필요 없이 깔끔하게 연동**되었고 Form 에 포함된 Field들의 Validation Rule의 정의 와 Validation이 성공과 실패 했을 때 **처리 등등을 하나의 클래스로 완전히 격리 시켜 구현 하는 형태**로 코드의 분리가 아주 괜찮고 **Mobx Store에서 Form을 핸들링할 수 있는 API도 제공**해서 깔끔하게 구현이 가능했습니다.(다른 라이브러리의 필요성을 아직까지는 못느꼈다.)

### Mobx Model 라이브러리들

Mobx에서 Model Layer를 구현 하는데 순수 Class형태로 구현이 가능 하지만 Model 라이브러리를 사용하면 라이브러리에서 제공하는 기능들을 사용할 수 있는데 대부분의 라이브러리가 **기능이 너무 많은 경우와 결정적 기능이 없는 경우 그리고 선언방식이 제각각이라 결론적으로는 적용하지 않았습니다**. 하지만 한번쯤 살펴볼 만한 가치는 있는 듯 하여 아래는 평가항목을 기준으로 하여 그동안 리서치 및 적용 테스트 결과를 정리 했습니다.

> 순전히 주관적인 평가 임을 미리 밝혀 둡니다.

**평가항목**

1. 모델 선언 방식이 직관적인가? 
    * 객체 선언 및 다루는 코드스타일이 기본 JAVASCRIPT Class선언에 크게 벗어나지 않는 방식
    * 서버로부터 가져온 json 데이터를 객체로 생성하고 메소드를 추가 하는 방식이 편리하고 직관적인가?
2. 객체와 컬렉션을 다루는 편의 method가 제공 되나? 
    * 컬렉션을 find, filter, save 등등 기존 array의 map, filter, some을 대체 가능한가?
3. fetch, delete, fetchAll 처럼 제공되는 method 만으로 간편하게 rest api 통신을 통한 객체(데이터) 업데이트 가능한가?
4. serializer 제공 되는가? 
    * json -> observable 가능한 객체로 변환 및 -> json으로 변환 가능한 기능이 제공되는가

#### [Mobx-State-Tree][23]

선언 방식이 **class 문법이 아니고 객체 리터럴 방식** a = {} 으로 action, observable 을 선언하는 방식이라 **직관성이 떨어지고** 기존 react component 선언이 class 문법을 주로 사용하는데 이질감이 느껴집니다.

특징 - 모델객체를 생성할 때 객체차제 타입을 지정할 수 있는 기능을 제공함.

**직관성 - 1/5  
객체-컬렉션 편의 메소드 - X  
rest-api method - X**

#### [Json-Mobx][24]

모델라이브러리 이긴하나 원래값을 되돌리는 목적으로 만들어 진 듯 하며 찾고자하는 모델라이브러리의 목적에 부합하지 않았습니다.

**직관성 - 1  
객체-컬렉션 편의 메소드 - X  
rest-api method - X**

#### [Mobx-Rest][25] \- 직접테스트완료

모델생성과 rest api method 및 객체 관리 자체 편의 메소드가 제공됨 **유력한 후보**였으나 테스트로 사용해본결과 Model List 구조에서 array가 observable 안되는 경우가 있었습니다.

**직관성 - 4/5  
객체-컬렉션 편의 메소드 - 4/5  
Rest Api Method - 4/5**

### [Libx][26] \- 직접 테스트 완료

모델 선언 방식은 **직관적**임, **모델간에 관계**를 맺을 수도 있음, 모델 객체를 자체를 **편의 메소드도 제공됨** 다만 rest-api method는 없음 다른 객체와 관계 맺는 것이 조금 직관성이 떨어짐, 객체를 생성하고 다루는 방식이 별도의 제공되는 메소드를 이용해야지만 의도한대로 동작해서 혼란스럽고 개발시에 많은 삽질을 요구할 듯 합니다. collection을 지정된 모델에 매핑 할때 object.assign 즉 **기존 모델 class에 attribute를 추가하는게 아니라 완전히 대체** 해버립니다. 그래서 Model class 에 메소드나 기본 property를 선언 해놓고 **api 호출한 데이터의 property를 추가하고 싶을 경우**에는 사용 할 수가 없었습니다.

**직관성 - 4/5  
객체-컬렉션 편의 메소드 - 4/5  
Rest Api Method - X**

### [Mobx-Collection-Store][27]

모델선언방식이 이상함 클래스안에 프로퍼티를 선언하는게 아니고 클래스를 생성해놓고 이후에 프로퍼티를 추가하는 형태 (직관성이 떨어짐) rest-api method가 포함된건 mobx-jsonapi-store를 사용해야합니다.

> 같은 라이브러린데 rest-api 로 호출한 객체를 생성하는 거라서 따로 써야됨

**직관성 - 1/5  
객체-컬렉션 편의 메소드 - 3/5  
Rest Api Method - X**

#### [Mobx-Model][28]

모델 선언 시 프로퍼티를 선언하는 방식이 static 으로 선언하게 되어 있습니다. 관계도 설정할 수 있는데 좀 번잡스러운 느낌, rest-api method 제공되는데 제일 번잡스럽습니다.

**직관성 - 1/5  
객체-컬렉션 편의 메소드 - 1/5  
Rest Api Method - X**

#### [Mobx-Spine][29]

모델선언은 직관적이나 관계형성 serialize, 객체 관리 편의 메소드가 미제공, rest-api method는 제공되나 store에 선언하는 방식임 전제적으로 좀 빈약합니다.

**직관성 - 2/5  
객체-컬렉션 편의 메소드 - X   
Rest Api Method - 1/5**

#### [Mmlpx][30]

모델선언 직관적임, 특이점은 DI 방식을 적용 @inject 데코레이션으로 다른 mobx 스토어나 mobx model 객체를 Inject 할 수 있습니다.(spring과 유사함) 그러나 객체나 컬렉션 자체 편의 메소드는 없고, rest-api method 없음, 특징으로는 snapshot기능이 있어서 변경되기 전 데이터로 돌리기 하는 기능이 있는 있습니다.

**직관성 - 4/5  
객체-컬렉션 편의 메소드 - X  
rest-api method - X**

## 이글을 마치며

주관적 경험을 통해 느낀점이지만 Mobx는 Redux에 비해서 낮은 러닝 커브와 높은 가독성이 확실한 장점으로 다가 왔습니다. React State관리 라이브러리를 선택할 때 조금이나마 도움이 되었으면 하는 바램과 함께 글을 마칩니다.

[1]: /post-img/react-mobx/redux-diagram.jpg
[2]: https://repo.yona.io/doortts/blog/post/297
[3]: /post-img/react-mobx/mobx-data-flow.jpg
[4]: /post-img/react-mobx/mobx-spring-layer.png
[5]: /post-img/react-mobx/mobx-redux-layer.png
[6]: /post-img/react-mobx/brms-dashboard.png
[7]: /post-img/react-mobx/brms-search-rider.png
[8]: https://brunch.co.kr/@ultra0034/63
[9]: /post-img/react-mobx/atomic-design.png
[10]: /post-img/react-mobx/atomic-design-folder.png
[11]: https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0
[12]: /post-img/react-mobx/component-type.png
[13]: /post-img/react-mobx/npm-trend-ui-lib.png
[14]: /post-img/react-mobx/brms-dashboard-dark.png
[15]: https://medium.com/@oleg008/jss-is-css-d7d41400b635
[16]: /post-img/react-mobx/redux-with-libs.png
[17]: https://redux-form.com/8.1.0/
[18]: https://davidkpiano.github.io/react-redux-form/docs.html
[19]: https://github.com/jaredpalmer/formik
[20]: https://github.com/formsy/formsy-react/
[21]: https://github.com/react-tools/react-form
[22]: https://foxhound87.github.io/mobx-react-form/
[23]: https://github.com/mobxjs/mobx-state-tree
[24]: https://github.com/danielearwicker/json-mobx
[25]: https://github.com/masylum/mobx-rest
[26]: https://github.com/jeffijoe/libx
[27]: https://github.com/infinum/mobx-collection-store
[28]: http://wearevolt.github.io/mobx-model/
[29]: https://github.com/CodeYellowBV/mobx-spine
[30]: https://github.com/mmlpxjs/mmlpx

  