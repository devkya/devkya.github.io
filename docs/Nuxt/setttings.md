---
layout: default
title: 1. 초기 세팅
parent: Nuxt3
nav_order: 1
---
# Nuxt3 - 초기 세팅
## Vuetify3 세팅
* 참고 링크
  1. [Code Nontecou - How to use Vuetify with Nuxt 3](https://codybontecou.com/how-to-use-vuetify-with-nuxt-3.html)

---

`npm`보다 `yarn`이 더 호환이 좋아 `yarn`을 사용한다. Nuxt3 에서 현재는 `Typescript`를 사용하고 있지만, 불편한게 한 두가지가 아니여서 장단점을 따져보는 중이다.
1. 프로젝트 폴더에서(`package.json`이 있는 디렉토리) 커멘드 실행
  ```
  yarn add vuetify@next sass
  ```
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

## Eslint & Prettier 세팅
* 참고 링크
  1. [Adding ESLint and Prettier to Nuxt 3 (2023)](https://dev.to/tao/adding-eslint-and-prettier-to-nuxt-3-2023-5bg)
  2. [ESLint, Prettier Setting, 헤매지 말고 정확히 알고 설정하자](https://helloinyong.tistory.com/325)

---

참고 링크 2번은 `Eslint`, `Prettier`가 어떤 역할을 하는지 잘 설명되어 있다.  
또한 VSCode extension 설치 vs 플러그인 직접 설치의 사용도 잘 설명해주니 참고하면 좋을 듯 하다.  
나는 보통 플러그인 직접 설치로 사용하기 때문에 `Nuxt3`에서도 마찬가지로 세팅한다.   

1. 필요한 플러그인 설치 
  ```
  # ESLint
  yarn add --dev eslint

  # Prettier
  yarn add --dev prettier eslint-config-prettier eslint-plugin-prettier

  # TypeScript support
  yarn add --dev typescript @typescript-eslint/parser @nuxtjs/eslint-config-typescript
  ```

2. `nuxt.config.ts`와 같은 레벨에 `.eslintrc.cjs` 생성
> rule에 해당하는 부분은 [eslint 공식홈페이지](https://eslint.org/docs/latest/rules/)에서 확인 가능하다. 나는 no-console 제외 따로 설정하지 않고, `plugin:prettier/recommended` 에서 제공되는 것만 기본적으로 사용하는 편이다.(건들수록 문제가 발생하는 느낌...)
  ```javascript
  // .eslintrc.cjs
  module.exports = {
    root: true,
    env: {
      browser: true,
      node: true,
    },
    parser: "vue-eslint-parser",
    parserOptions: {
      parser: "@typescript-eslint/parser",
    },
    extends: ["@nuxtjs/eslint-config-typescript", "plugin:prettier/recommended"],
    plugins: [],
    rules: {
      "no-console": "off",
      ...
    },
  };
  ```

3. `package.json`에 script 추가 
```javascript
// package.json
{
  // ...
  "scripts": {
    // ...
    "lint:js": "eslint --ext \".ts,.vue\" --ignore-path .gitignore .",
    "lint:prettier": "prettier --check .",
    "lint": "yarn lint:js && yarn lint:prettier",
    "lintfix": "prettier --write --list-different . && yarn lint:js --fix"
    // ...
  }
  // ...
}
```
4. `yarn lint` 실행하여 error 체크
5. 끝!