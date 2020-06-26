---
layout: post
title: "[React] 페이지 이동시 스크롤을 상단으로 초기화 하는 방법(scroll top)"
description: "react scroll top"
author: kimchanjung
date: 2020-06-25 16:00:00 +0900
categories: react
published: true
---

# [React] 페이지 이동시 스크롤을 상단으로 초기화 하는 방법(scroll top)
아니러니 하게도 **React**로 개발 하다보면 기본 매카니즘이 **상태란 것 남아 있기 때문**에 나는 분명 **다른페이지로 이동** 했는데도 새로 고침을 하였는데도 불구하고 **이전의 스크롤위치로 페이지가 나타나는 현상**이 있습니다. 

사실은 이런 장점 때문에 **SPA없이 개발하던 방식**은 **검색값들을 유지**하기 위해서 페이지 이동시 마다 **query string을 url에 포함**시켜 가지고 다니는 번거로운 작업을 했던 것에 비하면 **React에서 상태가 남아 있는 것은 이런 작업이 필요없는 편리한 장점**이 있습니다.    

하지만 스크롤의 경우에는 페이지를 왔다갔다 하는 사이에도 **페이지 스크롤위치가 엉뚱하게 중간즘이나 아래**에 있는 경우가 나타다 당황스러웠던 기억이 납니다. 본 포스팅에서 **페이지 이동시 스크롤을 상단에 위치하는 방법**을 알아봅니다.

## 페이지를 확인하여 Scroll을 상단으로 이동시키는 컴포넌트 작성
```react
class ScrollToTop extends React.Component {
  
  getSnapshotBeforeUpdate(prevProps, prevState) {
    const { location = {} } = prevProps;

    // 컴포넌트 업데이트 시 페이지 이동인지 확인
    if (this.props.location.pathname !== location.pathname) {
      window.scrollTo(0, 0);
    }
  }
  
  render() {
    return this.props.children;
  }
}

export default ScrollToTop;
```
- **getSnapshotBeforeUpdate** 메소드에서 매번 업데이트가 발생 할 때 마다 **현제 페이지와 이전페이지를 비교**하여 같은 페이지가 아니라면 즉 페이지 이동이 있었다면 **스크롤을 상단으로 이동**시킵니다.  
-  **페이지비교 조건이 없으면** 해당페이지에서 무엇인가 **업데이트가 발생할 때 마다 스크롤이 이동**되어 버리니 꼭 필요한 조건입니다.   

### this.props.children에 대한 설명
- **단순히 스크롤이동 처리만하는 컴포넌트**이므로 실제 동작하는 페이지 상위에 위치합니다.
- 실제 페이지가 구현된 **하위 컴포넌트는 this.props.children props**에 있습니다.
- 스크롤처리를 하고 **단순히 하위 컴포넌트는 그대로 렌더링**합니다.


## 생성한 ScrollTop 컴포넌트를 적용한다.
```react
import ScrollToTop from './ScrollToTop';

class ContentRoot extends React.Component {
  render() {
    return (
      <Router history={history}>
        <ScrollToTop>
          <UserPage />
        </ScrollToTop>
      </Router>
    );
  }
}

export default ContentRoot;
```
- **ScrollToTop** 컴포넌트를 실제 페이지를 구성하는 페이지 상단에 위치 시킵니다.  
- **ScrollToTop**은 페이지 정보를 사용해야 하므로 **react-router**의 라우팅 설정 컴포넌트의 하위에 위치해야합니다.
- **Router(라우팅관련 정보 설정) > ScrollTop(페이지명을 받아옴) > UserPage(실제페이지는 렌더링)**

## 마무리
실제 **동작페이지 컴포넌트**와 **라우팅설정컴포넌트** 사이에 **스크롤상단처리컴포넌트**를 추가하여 **페이지 이동시** 스크롤을 상단으로 **초기화 하는 방법**을 알아보았습니다.