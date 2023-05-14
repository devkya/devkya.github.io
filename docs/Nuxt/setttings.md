---
layout: default
title: 1. 초기 세팅
parent: Nuxt3
nav_order: 1
---
# 1. Nuxt3 - 초기 세팅하기
## Vuetify3 세팅하기
* 참고 링크
  1. [Code Nontecou - How to use Vuetify with Nuxt 3](https://codybontecou.com/how-to-use-vuetify-with-nuxt-3.html)
---
`npm`보다 `yarn`이 더 호환이 좋아 `yarn`을 사용한다. Nuxt3 에서 현재는 `Typescript`를 사용하고 있지만, 불편한게 한 두가지가 아니여서 장단점을 따져보는 중이다.
1. 프로젝트 폴더에서(`package.json`이 있는 디렉토리) `yarn add vuetify@next sass` 실행
  ```javascript
  // package.json 에 추가된 모듈
  ...
    "dependencies": {
    ...
    "sass": "^1.62.1",
    "vuetify": "^3.2.2"
    ...
  }
  ...
  ```
2. `plugin` 폴더 생성
3. `plugin` 폴더 안에 `vuetify.js` or `vuetify.ts` 폴더 생성
  ```javascript
  import { createVuetify } from "vuetify";
  import * as components from "vuetify/components";
  import * as directives from "vuetify/directives";
  import "@mdi/font/css/materialdesignicons.css";
  export default defineNuxtPlugin((nuxtApp) => {
    const vuetify = createVuetify({
      ssr: true,
      components,
      directives,
    });

    nuxtApp.vueApp.use(vuetify);
  });
  ```
4. `nuxt.config.ts` 해당 코드 추가
  ```javascript
  // In defineNuxtConfig ...
  css: ['vuetify/lib/styles/main.sass'],
  build: {
    transpile: ['vuetify'],
  },
  vite: {
    define: {
      'process.env.DEBUG': false,
    },
  },
  ```
5. 끝!

## Eslint & Prettier 세팅하기
* 참고 링크
  1. [Adding ESLint and Prettier to Nuxt 3 (2023)](https://dev.to/tao/adding-eslint-and-prettier-to-nuxt-3-2023-5bg)

