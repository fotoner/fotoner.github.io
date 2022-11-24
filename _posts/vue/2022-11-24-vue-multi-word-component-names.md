---
title: "Vue: vue/multi-word-component-names 해결"
last_modified_at: 2022-11-24T20:50:00
categories: 
 - Vue
tags:
 - ESLint
 - Vue
---
사실 결론부터 말하자면 Vue에서는 컴포넌트의 이름은 두개 이상의 단어 조합을 사용해야 한다고 한다.

![image](/assets/images/posts/vue/01.png)

이는 HTML 태그와 혼동을 일으킬 수 있는 컴포넌트를 제거하기 위해서라고 한다. (앞글자를 대문자로 만들어서 중복을 피하면 어떨까 싶긴한데...)

쨋든 이러한 에러가 발생하는 이유는 Vue에서 빌드를 하는 과정에서 번들링을 하기전 ESLint를 이용하여 코드 컨벤션과 관련된 정적분석을 거치게 된다


## ESLint
ESLint는 니콜라스 자카스(Nicholas C. Zakas)가 만든 도구로 최근 가장 널리 사용된다. 특히 Airbnb, PayPal, facebook 과 같은 대형회사에서도 활발하게 사용될 정도로 신뢰할 수 있는 도구이다.

특히나 자바스크립트에서는 다른 언어와 달리 정말 쓸데 없을정도로(...) 유연한 문법구조를 갖는다. 그렇다 보니 개발자에따라 정말 다앙한 코드를 만들게 되며 어떤 경우에는 의도를 파악하기 정말 난해한 코드를 만들거나 런타임 전까지는 오류를 파악하기 힘든 코드를 만드는 경우도 자주 생기게 된다. 

이럴때 ESLint를 사용하여 규칙을 만들어 프로젝트 코드를 관리하게 되면 전체 코드 컨벤션을 유지하면서 비교적 오류가 적은 코드를 만들 수 있게 된다. 

![image](/assets/images/posts/vue/02.png)

Vue에서는 이러한 ESLint가 프로젝트를 생성하는 시점부터 기본적으로 설치가 되어 있으며, 추가적으로 Vue에만 있는 문법역시 지원하기위하여 자체적인 ESLint 플러그인을 만들었다.

<br/>

## 그래서 이걸 꼭 지켜야 하는가? 
그렇진 않다. 이를 피하기 위해서는 크게 3가지 방법이 있다.

첫째로 config 상으로 eslint자체를 구동시키지 않는 것이다(...)

```js
/// vue.config.js
const { defineConfig } = require('@vue/cli-service')
module.exports = defineConfig({
  transpileDependencies: true,
  lintOnSave:false
})
```

vue.config.js에서 다음과 같이 lintOnSave:false를 추가하게 되면 ESLint자체가 동작하지 않는다... 하지만 이렇게 되면 고작 네이밍 컨벤션 하나 나한테 맞게 고치겠다고 다른 오류 검사등은 하지 않게 된다.(초가삼간 다 불태우는 꼴)

그래서 나는 이 방법은 패스하였다.

두번째로 vue 컴포넌트 파일에 예외 처리를 하는 것이다.
```js
<script>
export default {
  // eslint-disable-next-line vue/multi-word-component-names
  name:"Home"
}
</script>
```
다음과 같이 vue파일 내부에 다음과 같은 주석을 삽입하면 된다.

하지만 매번 이렇게 export 부분에 주석을 삽입해야하는 번거로움이 존재하기에 정말 일부의 컴포넌트에 적용할게 아니면 꺼려지는게 사실이다.

세번째로 필자가 가장 선호하는 방법으로 rule를 추가하여 multi-word-component-names 예외 처리하는 하는 것이다.
```json
  // package.json 에다가 다음과 같은 룰을 적용했지만 
  // 프로젝트에 따라서는 별도의 .eslint.js 에 다음과 같은 룰을 적용하기도 한다. 
  "eslintConfig": {
    "root": true,
    ...(생략)...
    "rules": {  
      "vue/multi-word-component-names": ["error", {
        "ignores": ["default"]
      }]
    }
  },
```
다음과 같이 룰에다가 예외를 지정할 수 있다. 'default'를 하면 모든 파일에 적용되며 'main' , 'product'처럼 특정 파일에만 적용할 수도 있다. 또한 별도의 디렉터리 별로 룰을 지정할 수도 있긴 한데 본 포스트에서는 다루지 않겠다.

## 결론
ESLint와 관련된 이슈는 오늘 살펴본 주제 외에도 매우 방대하다. 그렇지만 글의 서두에서도 본것처럼 이와 같은 룰을 적용시킨데에는 그만한 설계 의도가 있을 것이고 왠만해서는 그것을 따르는 것이 권장된다. 하지만 개발의 편의성이나 팀의 개발 방침에 따라 다음과 같은 예외가 필요할 수 있으리라 생각된다. ~~그렇다고 다 끄지말고~~