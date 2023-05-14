---
layout: default
title: 2. ApexCarts 세팅
parent: Nuxt3
nav_order: 2
---

# Nuxt3 - 초기 세팅하기
## Apex Charts 세팅하기
* 참고 링크
  1. [Github issues](https://github.com/apexcharts/vue3-apexcharts/issues/9)

---

vue3 에서 사용하던 라이브러리를 `nuxt3.js` 에서 사용하는 것이 쉬운 일이 아니다. (`vue3-easy-data-table`은 실패했다)  
프로젝트에 `ApexCharts` 사용법을 찾아보다가 헤맸던 부분을 정리했다.

1. 필요 모듈 설치
  ```console.log
  yarn add apexcharts vue-apexcharts --dev
  ```
  
2. `plugins` 폴더 생성

3. `plugins` 폴더 안에 `apexcharts.client.ts` 파일 생성

  ```javascript
  import VueApexCharts from "vue3-apexcharts";
  export default defineNuxtPlugin((nuxtApp) => {
    nuxtApp.vueApp.use(VueApexCharts);
  });
  ```
vue3-apexchart 세팅은 모두 끝났다. 플러그인 파일에 `.client`가 추가된 이유는 [공홈](https://nuxt.com/docs/guide/directory-structure/plugins)에서 확인할 수 있는데, `.client` 접미사를 사용함으로서 클라이언트 측에서만 플러그인을 로드하여 에러를 방지한다.(안하면 `window is not defined` 에러 발생함) `.server` & `.client` 에 대해서는 추가적으로 스터디 필요할 듯 하다.  
이제 `ApexCharts`가 잘 동작하는지 확인해보자.

4. 테스트 해볼 코드

  ```html
  <!-- template -->
  <template>
    <ClientOnly>
      <apexchart
        width="500"
        type="bar"
        :options="chartOptions"
        :series="series"
      ></apexchart>
    </ClientOnly>
  </template>
  ```
  
  ```javascript
  // script
  <script setup>
  const chartOptions = {
    chart: {
      id: "vuechart-example",
    },
    xaxis: {
      categories: [1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998],
    },
  };
  const series = [
    {
      name: "series-1",
      data: [30, 40, 35, 50, 49, 60, 70, 91],
    },
  ];
  </script>
  ```

  추가로 클라이언트에서만 렌더링 되는 플러그인은 `Client-Only` 태그로 감싸아야 한다. [공홈 문서](https://nuxt.com/docs/api/components/client-only)
5. 끝!