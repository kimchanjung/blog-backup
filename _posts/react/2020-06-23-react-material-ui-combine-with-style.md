---
layout: post
title: "[React] Material-UI에서 withStyle 사용시 1개 이상 여러개 style을 함께 사용하는 방법(multiple)"
description: "Material-UI에서 withStyle 함수로 JSS 스타일을 적용할때 1개 이상 여러개(multiple) 스타일을 같이 사용하는 방법을 알아봅니다."
author: kimchanjung
date: 2020-06-24 14:00:00 +0900
categories: react
published: true
---

# React Material-UI에서 withStyle 사용시 1개 이상 style을 함께 사용하는 방법
Material-UI사용시 스타일(css)를 적용하는 방법은 `JSS(javascript 객체방식으로 style을 정의)로 정의한 파일`을 withStyle(Higher-order component 방식으로 Style을 적용)함수를 이용하여 템플릿에 적용하고 `style class를 가져와 사용하는 구조가 일반`적입니다.  
그러나 `withStyle을 사용할때 JSS로 정의된 파일 즉 객체를 1개만 사용`할 수 있기 때문에 일반적으로 `공통 스타일을 정의 하고 개별 템플릿에 따로 정의한 스타일이 있는 경우 2개를 한꺼번에 사용하기가 어렵`습니다. 
> 이번포스팅은 withStyle 함수로 JSS 스타일을 적용할때 `1개 이상 스타일을 같이 사용하는 방법`을 알아봅니다.

## Material-UI 스타일을 적용하는 일반적인 방식
일련의 코드는 사실상 css를 import하고 css class를 사용하는 보편적인 방식과 같은 Material-UI를 사용할 때 적용하는 방식을 설명한 것이라보 보면됩니다.  

### CSS 대신 JSS로 스타일을 선언한다.
```javascript
// 파일명 CustomButtonStyle
export default {
  customButton: {
    maxWidth: 400,
    maxHeight: 400
    marginTop: 10
  }
}
```
> css를 javascript 객체형식으로 선언해서 사용한다. 또한 하나의 객체

### 템플릿에 스타일을 적용한다
```react
import { withStyles } from '@material-ui/styles';
// 스타일을 정의한 객체를 import 한다
import style from 'CustomButtonStyle';

// CustomButtonStyle 선언했던 customButton 글래스를 사용하여 스타일을 적용한다.
function CustomButton(props) {
  const { classes } = props;
  return <Button className={classes.customButton}>custom button</Button>;
}

// withStyle 함수를 이용하여 CustomButton 템플릿에 style을 적용한다
export default withStyles(styles)(CustomButton);
```
> 스타일 파일을 import해서 선언한 스타일객체를 withStyle 함수로 적용한다 

### withStyle은 하나의 스타일 객체만 적용가능하다.
큰 프로젝트를 하다보면 상황에 따라 적절히 파일을 나누기도하는 경우도 있습니다. 예를 들면 `공통으로 사용하는 스타일을 정의한 공통스타일 JSS객체가` 있고 `특정 템플릿에만 사용하는 스타일만 정의한 JSS객체`가 있을 수도 있는 경우 입니다.  

하지만 Material-UI의 `withStyle을 사용하여 1개의 JSS객체의 만 적용가능`하기 때문에 공통부분을 따로 묶은 JSS객체와 별도로 선언한 JSS객체 즉 1가지 이상의 스타일객체를 적용할 수가 없습니다.  

그냥 파일 하나에 모두 선언하고 가져다 사용해야 합니다. 그래서 `1개 이상의 스타일객체를 매개변수로 받아서 두객체를 1개로 합치는 함수를 추가하여 1개이상의 스타일객체를 사용하는 방법`을 알아봅니다.

### 1개 이상의 스타일 객체를 1개의 객체로 합치는 함수를 작성
```javascript
// 파일명 CombineWithStyles 
import { withStyles } from '@material-ui/core/styles';

function CombineWithStyles(...styles) {
  function combineStyles() {
    return function CombineStyles(theme) {
      const outStyles = styles.map((arg) => {
        if (typeof arg === 'function') {
          return arg(theme);
        }
        return arg;
      });
      
      return outStyles.reduce((acc, val) => Object.assign(acc, val));
    };
  }
  
  return withStyles(combineStyles(...styles), { withTheme: true });
}

export default CombineWithStyles;
```
> withStyle함수를 래핑하여 1개이상의 스타일객체를 매겨변수로 받아와서 합치는 기능을 추가한다.

### withStyle 함수를 래핑한 함수를 사용한다.
```react
import { withStyles } from '@material-ui/styles';
import combineWithStyles from 'core/CombineWithStyles';
// 공통스타일을 import
import commonStyle from 'CommonStyle';
// 개별스타일을 import
import style from 'CustomButtonStyle';

// CustomButtonStyle 선언했던 customButton 글래스를 사용하여 스타일을 적용한다.
function CustomButton(props) {
  const { classes } = props;
  return <Button className={classes.customButton}>custom button</Button>;
}

// withStyle 함수를 이용하여 style과 공통스타일(commonStyle)을 동시에 적용할 수 있다.
// combineWithStyles(commonStyle, style1, style2, ....) <= 매개변수의 개수 한개는 없다.
export default combineWithStyles(commonStyle, style)(CustomButton);
```
> combineWithStyles 래핑 함수로 스타일객체를 1개~N개 까지 적용이 가능하다.


## 마무리
Material-UI withStyle 함수의 스타일 적용 개수가 1개 뿐이라 적절히 구조적으로 나눈 스타일 파일을 상황에 맞게 여러개 적용하고 싶을 때
해결하는 방법을 알아 보았습니다. 더 좋은 방법을 알고 계신분은 코멘트 남겨주세요