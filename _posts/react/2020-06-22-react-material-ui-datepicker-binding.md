---
layout: post
title: "[react] mobx-react-form 사용시 material-ui datepicker의 date format이 제대로 동작하지 않는 문제 해결"
description: "[react] mobx-react-form 사용시 material-ui datepicker의 date format이 제대로 동작하지 않는 문제 해결"
author: kimchanjung
date: 2020-06-22 12:00:00 +0900
categories: react
published: true
---

# mobx-react-form 사용시 material-ui datepicker의 date format이 제대로 동작하지 않는 문제 해결
**mobx**사용시 form validation 라이브러리로 **mobx-react-form**을 사용했을 때 material-ui **datepicker의 날짜를 변경**한 후 화면에 표시되는 날짜는 **설정한 date format 대로 표시**가 되지만 **실제 값은 적용되어 있지 않은 문제**가 있습니다.

![material-ui-datepicker](/post-img/react/material-ui-datepicker.png){: width="40%"}{: .img-center}

**datepicker** 에서 **날짜를 선택** 했고 설정한 날짜형식은 **yyyy-MM-dd** 대로 **입직일** 항목에 **2019-05-31**로 표시가 되어 있지만
실제값은 **yyyy-MM-dd**가 아닌 **Fry May 31 2019 13:44:31 GMT+0900** 되어 있는 버그입니다  

### 문제가 발생했던 라이브러리의 버전정보
- mobx v5.9
- mobx-react-form v2.0.2
- material-ui v3.9.3
- material-ui-pickers v2.2.4

### mobx-react-form의 input 필드의 일반적인 바인딩의 예시
```react
// material-ui 컴포넌트와 mobx-react-form의 필드 바인딩
<TextField {...form.$('phoneNumber').bind()} />

// 실제 input 필드는 아래 처럼 name, onClick 같은 속성과 이벤트리스너가 생성됩니다.
<input name="phoneNumber" id="phoneNumber" onClick="..." onChange="..." ....
```
- **TextField**는 material-ui가 제공하는 input 필드의 컴포넌트입니다
- form.$('phoneNumber').bind() 추가하게 되면 name, id, onClick, onChange등이 생성됩니다.

### material-ui datepicker와 mobx-react-form의 바인딩
```react
<DatePicker
    format="yyyy-MM-dd"
    {...form.$('retireDate').bind()}
/>    
```
- DatePicker 컴포넌트에 mobx-react-form을 바인딩합니다. 
- DatePicker 컴포넌트가 제공하는 속성값 **format="yyyy-MM-dd"** 에 날짜 포맷을 설정합니다
- **{...form.$('retireDate').bind()} mobx-react-form** 을 사용하여 바인딩합니다
- 날짜변경 및 화면에 선택된 날짜는 **yyyy-MM-dd** 형식으로 잘 표시 되지만 실제 **retireDate**의 값을 보면 **Fry May 31 2019 13:44:31 GMT+0900** 값으로 설정되어 있습니다.
- **{...form.$('retireDate').bind()}** 바인딩 처리를 했기 때문에 내부적으로 **onChange이벤트리스너**에 변경 함수가 자동적으로 생성이 되어 있는데 이 함수가 **yyyy-MM-dd** 형식으로 값을 적용하지 못하는 문제입니다.  

### datepicker onChange 메소드를 오버라이드 하여 해결합니다.
```react
<DatePicker
    format="yyyy-MM-dd"
    {...form.$('retireDate').bind()}
    onChange={value => form
      .$('retireDate')
      .onChange(format(new Date(value), 'yyyy-MM-dd'))
    }
/>    
```
- **{...form.$('retireDate').bind()}**에 의해서 **onChange이벤트리스너**가 이미 생성되어 있지만 **onChange**를 임의로 추가하여 자동으로 생성된 **onChange를 오버라이드** 합니다.
- **form.$('retireDate') <= mobx-react-form**의 retireDate 값에 접근합니다.
- **form.$('retireDate').onChange** 메소드에 매개변수를 넘길때 retireDate의 선택된 날짜 값을 **yyyy-MM-dd** 변환하여 넘기도록 처리합니다.

### 마무리
**mobx-react-form**에도 많은 기능이 있기 때문에 **mobx-react-form**으로 선언한 form validation 처리 **class내부에서 처리**하는 방법이 일반적인데 여러가지 동작에서 제대로 작동하지 않고 제시한 방법처럼 처리해야만 해결이 가능 하였습니다.

혹시 최신버전에서 해결이되었거나 다른 깔끔한 방법을 알고 계시면 댓글을 남겨 주세요



