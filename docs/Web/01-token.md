---
layout: default
title: JWT 인증
parent: Web
nav_order: 1
---

# Vue3 & Django Http Only Cookies JWT 구현하기

- 참고 링크

1. [Part-1 VueJS JWT Auth Cookie - Access Token Usage](https://www.learmoreseekmore.com/2021/04/part-1-vuejs-jwt-auth-cookie-access-token-usage.html)

---

## Setting

### Backend 세팅
1. 필요 라이브러리 설치
  참고 : 필자는 `pipenv` 가상환경을 사용함

    ```console
    pipenv install django djangorestframework 
    pipenv install django-cors-headers djangorestframework-simplejwt
    ```

2. Project & App 생성
  ```console
  django-admin startproject backend
  cd backend
  python manage.py createapp accounts
  ```

3. `settings.py` 
  각각의 라이브러리의 세팅은 공식 문서를 참고하는 것이 가장 실수가 적다.
  
    ```python
    INSTALLED_APPS = [
      'django.contrib.admin',
      'django.contrib.auth',
      'django.contrib.contenttypes',
      'django.contrib.sessions',
      'django.contrib.messages',
      'django.contrib.staticfiles',
      
      'rest_framework',
      'rest_framework_simplejwt',
      'corsheaders',
      
      'accounts',
    ]

    # DRF
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES': [
            'rest_framework.permissions.IsAuthenticated'
        ],
        'DEFAULT_AUTHENTICATION_CLASSES': [
            'rest_framework_simplejwt.authentication.JWTAuthentication',
        ]
    }
    # simplejwt
    SIMPLE_JWT = {
        "ACCESS_TOKEN_LIFETIME": timedelta(minutes=30),
        "REFRESH_TOKEN_LIFETIME": timedelta(days=7),
        "ROTATE_REFRESH_TOKENS": False,
    }
    # CORS
    CORS_ALLOWED_ORIGINS = [
        "http://localhost:5173"
    ]

    # CommonMiddleWare 바로 위에 
    # 로그인 & 회원가입 구현시 SessionMiddleware, SecurityMiddleware 충돌 방지
    MIDDLEWARE = [
    ...
    'corsheaders.middleware.CorsMiddleware',
    ...
    ]
    ```
### FrontEnd 세팅
`Nuxt.js`로 변경하기 위해 스터디 중이지만, 현재 프로젝트가 `vite vue3` 를 사용하고 있으므로 이를 사용한다.

1. 프로젝트 생성
  `pinia`, `test` 등 각자 사용하고자 하는 세팅 사용
  ```console
  npm init vue@latest
  cd frontend
  npm install 
  ```

2. Login & DashBoard 페이지 생성 
  