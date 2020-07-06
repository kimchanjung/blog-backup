---
layout: post
title: "[React] Mobx의 async action(비동기 액션) 처리를 가장 깔끔하게 구현하는 방법"
description: "Mobx 공식 문서에 제공하는 비동기액션 처리의 예제보다 더 깔끔한 처리방법을 살펴봅니다"
author: kimchanjung
date: 2020-06-24 14:00:00 +0900
categories: react
published: true
---

# Mobx의 async action(비동기 액션) 처리를 가장 깔끔하게 구현하는 방법
Mobx에서 `상태를 수정하는 함수는 @action 데코레이터를 추가하여 상태를 변경`합니다. api를 호출하는 `비동기 처리의 경우 Mobx에서는
몇가지 방식을 제공`합니다. 그러나 별로 깔끔하지가 않습니다. `Mobx 공식 문서에 제공하는 비동기액션 처리의 예제보다 더 깔끔한 처리방법`을 살펴봅니다.

## Mobx의 일반적인 상태변경 action 처리
```javascript
class UserStore {
  @observable
  name;

  @action
  changeName(name) {
      this.name = name;
  }
}
```
> changeName 메소드에 @action 데코레이터를 붙이므로써 name 값이 변경된다.

## 예상과 달리 동작하지 않는 Mobx의 비동기 action 처리
비동기 api를 호출하는경우 `아래와 같이 코드를 작성하는 것이 일반`적이나 비동기 api 호출 완료후 결과값을 처리하는 `콜백함수는 Mobx의 action에서 처리되지 않습니다`.
```javascript
class UserStore {
  @observable
  users;

  @action
  getUsers(name) {
    axios.get('/users')
      .then(response => {  
        // 서버로부터 가져온 데이터를 저장하는 콜백처리는 동작하지 않는다.
        this.users = response.data;
      })
      .catch(error => {
        console.log(error);
      })
  }
}
```
> Mobx의 action 에서 비동기 처리 콜백은 동작하지 않기 때문에 특별한 처리를 해주어야 한다

## Mobx에서 비동기함수를 사용하여 상태를 변경하는 경우 action 처리
Mobx 공식 문서에는 여러가지 방법을 제시하고 있습니다. 하지만 그리 깔끔한 방법들이 아니란 것이 아쉽습니다.

### 비동기 콜백 함수를 따로 분리하는 방법
```javascript
class UserStore {
  @observable
  users;

  @action
  getUsers() {
    axios.get('/users')
      .then(this.getUsersCallback)
      .catch(this.getUsersError)
  }
  
  @action.bound
  getUsersCallback(response) {
      this.users = response.data;
  }
  @action.bound
  getUsersError(error) {
      console.log(error);
  }
}
```
> 콜백함수를 따로 분리하여 action 테코레이터를 추가한 메소드로 만든다  
> 하지만 메소드가 개수가 많아지고 깔끔한 방법은 아니다.

### 비동기 콜백 함수를 action 키워드로 래핑하는 방법
```javascript
class UserStore {
  @observable
  users;

  @action
  getUsers(name) {
    axios.get('/users')
      .then(action("successUsers", response => {  
        this.users = response.data;
      })
      .catch(action("errorUsers", error => {
        console.log(error);
      })
  }
}
```
> 전보다 조금 깔끔하지만 action으로 한번더 래핑하는 것이 번거롭다..

### runInAction를 사용하는 방법
```javascript
class UserStore {
  @observable
  users;

  @action
  getUsers(name) {
    axios.get('/users')
      .then(response => {  
        runInAction(() => {  
          this.users = response.data;
        })
      })
      .catch(error => {
        runInAction(() => {  
          console.log(error);
        })
      })
  }
}
```
> 이방법도 그리 깔끔하지는 않다.

### async/await를 사용하는 방법
```javascript
class UserStore {
  @observable
  users;

  @action
  async getUsers(name) {
     const res = await axios.get('/users');
     // 마찬가지로 await이후 구문은 처리되지 않기 때문에 runInAction으로 래핑처리를 해주어야한다.
     runInAction(() => {
       this.users = res.data
     });
  }
}
```
> async/await를 사용한 비동기 처리시에도 await이후 구문은 동작하지 않는다.  
> runInAction으로 래핑처리를 추가해야만 처리가 된다.
> Mobx공식 문서에서 제공하는 방법들이 모두다 깔끔하지가 않다.

### flow를 사용하여 처리하는 방법
```javascript
class UserStore {
  @observable
  users;

  getUsers = flow(function*() {
    const res = yield axios.get('/users');
    this.users = res.data
  }
}
```
> 이방법도 앞선 방법보다 깔끔하지만 flow로 래핑처리를 해야해서 그리 깔끔하지 못하다
> 그리고 클래스의 메소드 선언이 함수표현식의 형태라 함수명만 선언하는 방식보다 번잡하다  

## mobx-utils 이용하여 깔끔하게 처리하는 2가지 방법
### 1. mobx-utils의 @asyncAction을 용하여 래핑없이 깔끔하게 비동기액션을 처리
```javascript
import { asyncAction } from 'mobx-utils';

class UserStore {
  @observable
  users;

  @asyncAction
  async* getUsers() {
    const res = yield axios.get('/users');
    this.users = res.data;
  }
}
```
> - mobx-utils 라이브러리를 추가하고 import합니다
> - @asyncAction 데코레이터를 import하여 함수에 추가합니다.
> - 메소드명 앞에 async* 추가하고 generator yield 이용하여 비동기 처리를 합니다.

### 2. mobx-utils의 @actionAsync과 task를 이용하여 async/await 키워드 사용으로 깔끔하게 비동기액션을 처리
> 예전 버전에는 없는 기능이기 때문에 Mobx와 mobx-utils의 최신버전이 필요합니다.  

```javascript
import { actionAsync, task } from 'mobx-utils';

class UserStore {
  @observable
  users;

  @actionAsync
  async getUsers() {
    // task로 한번 래핑이 필요하긴 합니다.
    const res = await task(axios.get('/users'));
    this.users = res.data;
  }
}
```
> - mobx-utils 라이브러리를 추가하고 import합니다
> - actionAsync, task import하여 @actionAsync 함수에 추가합니다 
> - 메소드명 앞에 async 추가하고 await를 통하여 비동기 함수를 호출합니다
> - 다만 비동기 함수에 task를 이용하여 래핑이 한번 필요합니다.

## 마무리
Mobx 공식 문서에서 제공하는 비동기액션의 처리방법은 래핑이 많아 깔끔하지 못합니다. 그래서 mobx-utils 라이브러이에서 제공하는
데코레이터와 generator 또는 async/await 비동기처리 키워드의 조합을 통하여 직관적이고 깔끔한 방법으로 비동기처리하는 방법을 알아보았습니다.

