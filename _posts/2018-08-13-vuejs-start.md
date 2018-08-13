---

layout: post
title: ! "Vue.js 시작하기"
excerpt_separator: <!--more-->

tags:
 - vue.js
 - web
 - vuetify
 - chaem

---

안녕하세용  
웹 프론트엔드계에서 뜨거운 vue.js, react.js, angular.js에 대해 알아보았습니당  
사실은 `vue.js`에 대해서 주로 말할거에요! ㅎㅎ

<!--more-->

일단 뷰는 프론트엔트계에서 혜성같이 떠오른 자바스크립트 프레임워크라고 할 수 있습니다  
매우매우 많은 사람들이 뷰는 쉽다, 입문하기 좋다 등등 호평을 하고 있더라구요!  
이렇게 한국어로된 공식 사이트도 있고요!  

[vue.js공식한글사이트](https://kr.vuejs.org/v2/guide/#%EC%8B%9C%EC%9E%91%ED%95%98%EA%B8%B0)  

그래서 용감하게 도전하였으나... 읭?!  
웹좀해봤으면 진짜 간단하게 쭉쭉 짤수잇을 것 같더라구요!! but, 나는 아니라구 ㅠㅠ
우선, 처음에 vue.js, react.js, angular.js 중에 뭘 사용해볼까 좀 찾아봤었습니다.  

---

## vue.js
- 매우작고 가벼우며, 복잡도가 낮음
- react처럼 view에만 초점을 두기때문에 다른 라이브러리, 프레임워크와 혼용 용이
- CDN으로 불러와 사용하기 간편, webpack으로 구성 가능 (webpack이 프로젝트 관리 시 편함)
- 높은 프레임 속도의 데이터 시각화 또는 애니메이션을 프로토 타이핑 할 때 Vue는 개발시 초당 10 프레임을 처리(react보다 빠름)
- 템플릿 사용 : 다른 html파일에서 바로 사용할 수 있음 (react는 JSX)
- 주의사항 : v-html 을 사용하는 경우엔, 서버측에서 불필요한 부분은 필터링하게 할 것! 잘못하면 XSS 나 불필요한 악성 태그가 렌더링 될 수도 있음 (나름 보안요소라 써놓음ㅎㅎ)

### vue.js 정리
대부분 Webpack 같은 모듈 번들러를 사용하여 프로젝트를 구성해야하는 앵귤러와 리액트와 달리, 단순히 CDN 에 있는 파일을 로딩 하는 형태로 스크립트를 불러와서 사용하기 편하다. HTML 을 템플릿처럼 그대로 사용 할 수도 있어서 마크업을 만들어주는 디자이너/퍼블리셔가 있는 경우 작업 흐름이 매우 매끄럽다. 공식 라우터, 상태관리 라이브러리가 존재한다.

## react.js
- 오직 data -> UI 변경만 가능 (단방향 / UI -> data 변경 불가) : angular의 경우 view나 model에 의해 data가 변경됨

### react.js 정리
“컴포넌트” 라는 개념에 집중이 되어있는 라이브러리이다. 데이터를 넣으면 우리가 지정한 유저 인터페이스를 조립해서 보여주는 방식인 것이다. 라이브러리의 성능과 개발자 경험을 개선하기 위해 많은 연구를 한 결과이다. 생태계가 엄청 넓고, 사용하는 곳도 많다. HTTP 클라이언트, 라우터, 심화적 상태 관리 등의 기능들은 내장되어있지 않다. 따로 공식 라이브러리가 있는 것도 아니여서, 개발자가 원하는 스택을 맘대로 골라서 사용 할 수 있다. (혹은 직접 라이브러리를 만들어서 쓸 수 있다.)

### react와 vue 공통점
- 가상 DOM을 활용
- 반응적이고 조합 가능한 컴포넌트를 제공
- 코어 라이브러리에만 집중하고 있고 라우팅 및 전역 상태를 관리하는 컴패니언 라이브러리 존재

## Angular
### angular.js 정리
UI 를 구현할 때, 앵귤러만의 문법이 다양하게 존재한다. 특정 기능을 구현 할 때, 편리하게 대신 해주는 것들이 많다. 라우터, HTTP 클라이언트 등 웹 프로젝트에서 필요한 대부분의 도구들이 프레임워크 안에 내장되어 있다. 앵귤러1의 경우 만들어진지 꽤 오래 됐고, 기업에서 많이 사용이 돼서, 유지보수하고 있는 프로젝트가 많아서 사용률이 높은 편이다. 앵귤러2의 경우 매우 성숙하긴 하지만, 인지도 측면에선 아직 성장하는 단계이며, 주로 타입스크립트랑 함께 사용된다.

끗!  

---

사실 vue.js에 치우쳐 조사하긴 했습니다 ㅎㅎ 그치만 조사하고 나니, vue.js가 확실히 직관적이고 필요로하는 사전지식도 적고, 이식성이 좋다고 생각되었어요!!  

그렇다면 vue.js 사용 과정을 보도록 하겠습니다 ㅎㅎ  

## vue.js 사용방법
1. 브라우저 단에서 바로 적용 (chrome extension사용 가능)
-> 기존 프로젝트에 추가로 적용하는 용도

unpkg를 통해서 CDN주소를 제공하므로 다음 주소 HTML페이지에 추가하면 끝!
<script src="https://unpkg.com/vue/dist/vue.js"></script>

2. vue-cli 사용하기 (npm모듈로 제작되어 있어서 node.js가 필수적으로 설치되어야 쉽게 사용가능)
-> 새로운 프로젝트를 적용할때는 이 방식이 더 유용

## 온라인 컴파일러 사용하기

온라인 컴파일러들이 많이 존재하는데, jsbin, jsfiddle 등이 있습니다!

- [jsbin](https://jsbin.com/?html,js,output)  
- [jsfiddle](https://jsfiddle.net/boilerplate/vue)  

온라인 컴파일러로 간단히 사용해보면 확실히 간단하게 구현됩니다!! 그런데 어떤 기능이 있는지 몰라서 잘 못씀 ㅠ 레퍼런스가 많긴한데, 내가 원하는 것을 찾기 매우 어려웠어요 ㅠ    
개발을 시작하고 나서 일단 주먹구구 식으로 노가다 하기 시작!  
일단 온라인 컴파일러 안녕~  

## vue-cli사용하기

vue-cli를 셋팅해서 본격적으로 vue를 해보도록하겠습니다.  
vue-cli의 셋팅 방법은 생각보다 간단했습니다!!!
처음에 당연히 ubuntu server에서 하려다가 window에서도 간단하게 사용할 수 있어서 window cmd창을 켰다ㅎㅎ  
내가 vue에 대한 글들을 찾다가 느낀건데, vue 설치하는 것에 대한 글은 엄청 많은듯 ㅎㅎ  
우선, vue-cli는 npm을 이용하기 때문에 npm과 nodejs를 설치해 줘야합니다  

- npm과 nodejs가 설치되어있다면 버전확인!  

```
node -v
npm -v
```

- 설치 안되어있으면 설치!!  

```
apt install nodejs-legacy
sudo npm install -g npm //npm 패키지 설치
```

- vue-cli 설치하기  

```
sudo npm install -g vue-cli //vue-cli 설치
```

- vue 프로젝트 생성하기  

```
vue init webpack 프로젝트명 //프로젝트를 생성하고 webpack이라는 템플릿 사용
```

![]({{ site.baseurl }}/images/chaem/vue/vue_01.PNG)

- 프로젝트 실행하기  

```
cd 프로젝트명 //프로젝트 폴더로 이동
npm run dev // 프로젝트 실행
```

프로젝트를 실행시킨 후 웹에서 `http://localhost:8080`에 접속하면 다음과 같은 기본 화면을 볼 수 있습니다.  
![]({{ site.baseurl }}/images/chaem/vue/vue_02.PNG)

저는 코드 수정을 `VSCode`를 이용하여 했습니다.  
VSCode에서 파일 - 폴더열기 - 해당폴더 선택하여 코드를 열면, src경로에서 App.vue, main.js를 볼 수 있으며, components폴더에 HelloWorld.vue파일이 기본적으로 있습니다.  

```
assets :  외부에서 가져온 이미지, 파일, css, js파일 등
components : vuejs에서 사용하는 vue파일들을 생성하고 구현
router & index.js : vue router를 사용할 수 있는 곳
App.vue : root components
main.js : 전역설정을 하는 곳, 프로젝트의 base파일
```  

components에 page1파일을 추가했다고 가정하면, 웹에서 `http://localhost:8080/page1`로 접근할 수 있습니다.  
<br>
프로젝트 설정할때를 보면 라우터를 설정하였습니다. 저는 라우터를 사용안하였으나, 라우터에 관한 내용은 이 블로그에서 잘 설명해 주고있으니, 참고하면 좋을 것 같습니다.  

[vue-router살펴보기](http://blog.jeonghwan.net/2018/04/07/vue-router.html) 

<br>
또, vue에서는 bootstrap처럼 UI레이아웃을 제공하는 vuetify, bootstrap vue등이 있어요!!  

- vuetify 설치  
cmd창에서 다음과 같이 설치하고!  

```
npm install vuetify --save
```

main.js파일에 다음 코드를 설치해주면 vuetify에 있는 것을 사용할 수 있습니다!!!

```js  
import Vuetify from 'vuetify'
Vue.use(Vuetify);
```  

자세한 내용은 홈페이지에서 참고 가능합니다!  

[vuetify 홈페이지](https://vuetifyjs.com/ko/getting-started/quick-start)  

