---
layout: post
title: "[typescript] npm 모듈 import시 오류를 해결하는 방법(d.ts 파일 생성법)"
description: "typescript npm module import error d.ts"
author: kimchanjung
date: 2020-07-05 21:00:00 +0900
modDate: 2020-07-05 21:00:00 +0900
categories: typescrpt
published: true
---

# Typescript에서 js npm 모듈 import시 오류를 해결하는 방법
typescript로 프로젝트를 할 때 npm에서 필요한 모듈들을 설치하고 import해서 사용하려고 할때 `모듈 'module name'에 대한 선언 파일을 찾을 수 없습니다.` 라는 메시지와 함께 모듈을 import할 수가 없습니다.
> 원인은 기존 js 파일에서만 import 하여 사용가능한 형태만 지원하고 npm모듈을 만든 개발자가 typescript사용할 수 있도록 처리를 하지 않았기 때문입니다.

구글링 키워드가 잘 못 되었는지 정확한 문제 해결에 대한 답변을 찾을 수가 없었고 stack overflow의 답변들을 적용해보았으나 명확히 해결되지 않았습니다. `삽질 시간이 꽤 소모되어서 다른분들은 부디 삽질 하지 마시길 바라며 본 포스팅을 남깁니다`.

## 해결방법
### d.ts 파일 생성
```typescript
// @types/모듈명/index.d.ts 
declare module '모듈명';
```
> - 모듈명 - import 모듈 from '모듈명'에서 사용하던 모듈명과 동일합니다.

### d.ts 파일 생성 위치
```bash
app /src
    /node_modules
    /assets
    /@types <= 폴더를 생성
           /모듈명 <= 모듈명과 같은 폴더명을 생성
                /index.d.ts <= index.d.ts파일 생성
```
> 생성위치 - app/@types/모듈명/index.d.ts 

### import
```typescript
import 모듈 from '모듈명';
```
> import 하는 곳에서는 다른 부분없이 그대로 평소 처럼 import하여 사용한다.

### tsconfig.json 설정
설정을 추가해야한다고 하지만 본인의 경우에는 추가하지 않아도 동작하였습니다. `혹시 동작하지 않으신 분들은 추가 하세요`
```javascript
{
  "compilerOptions": {
    "typeRoots" : ["./@types", "./node_modules/@types"]
  },
}
```
> 새로 생성한 @types폴터를 추가해 줍니다.

### 마무리
typescript사용시 npm 모듈이 import되지 않는 문제 해결방법을 알아 보았습니다.