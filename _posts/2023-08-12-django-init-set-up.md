---
title: 반드시 알아야 할 Django 초기 세팅
date: 2023-08-12 00:34:00 +0900
author: devkya
categories: [Django]
tags: [python, django, setting]
---

# 🙂**들어가며...**

현업에서 `Django`를 사용하여 프로젝트들을 진행할때, 초기에 해야하는 세팅을 정리해보고자 한다.
더 좋은 방법이 있으면 댓글을 통해 알려주길 바란다!

# **세팅**

## **python & 가상환경 설정하기**

필자는 `pyenv` 를 사용하여 python version을 관리한다.

의존성 문제에서 조금 더 자유롭기 위해 프로젝트별로 python을 세팅하고 프로젝트에 설정된 python으로 가상환경을 설정한다.

### `pyenv` 설정 방법

1. OS에 따라 `pyenv` 를 설치 및 설정(라이브러리 설치 관련 내용은 본문에서는 스킵하겠다)
2. 원하는 python version 설치 `pyenv install 3.11.2`
3. 잘 설치되었는지 `pyenv`리스트 확인 `pyenv versions`
4. 로컬 프로젝트에 사용한 python version 설정 `pyenv local 3.11.2`

파이썬 가상환경은 여러가지가 있다.

`Venv`, `Pipenv`, `virtualenv` 등등… 장단점이 있겠지만 필자는 크게 체감하지 못하므로, 가장 익숙한 `pipenv` 를 사용한다.

### `pipenv` 설정 방법

[설치 및 소개 참고 링크](https://velog.io/@doondoony/pipenv-101)

1. `pipenv` 설치 및 프로젝트 디렉토리로 이동
2. 프로젝트에 설정된 python version으로 가상환경 생성 `pipenv --python `pynev which python``
3. 가상환경 실행 `pipenv shell`

## **settings.py 분리하기**

분리 목적은 환경(`development`, `test`, `production`)에 따라 다른 설정을 하고 싶을 때, 보안 문제, 코드 가독성과 유지보수성 여러가지 장점이 있다.

본문은 설정 방법에 대해서만 설명할 예정이다. 자세한 내용은 직접 찾아보길 바란다!

### 설정 방법

1. `settings.py` 가 있는 디렉토리에 `config/settings` 폴더 생성

   <aside>
   ❓ 필자 **TMI** 
   config 폴더를 생성하지 않고 settings 폴더만 생성해도 되지만, 필자는 config 폴더에 배포를 위한 `nginx` `gunicorn` 등의 스크립트를 추가함

   </aside>

   1. `cd "settings.py가 있는 디렉토리"`
   2. `mkdir -p config/settings`

2. settings 폴더로 `settings.py` 옮긴 후에 `base.py` 파일명 변경
3. **(중요)** `settings.py`를 이동하면 `base.py` BASE_DIR 변경해야 함 ⇒ `BASE_DIR = Path(__file__).resolve().parents[3]`
4. `development.py` `production.py` 파일 생성

   ```python
   # development.py
   from .base import *

   DEBUG = True
   ALLOWED_HOSTS = ["*"]
   ```

   ```python
   # production.py
   from .base import *

   DEBUG = False
   ALLOWED_HOSTS = []  # TODO: 배포할 도메인 주소 또는 IP를 넣을 예정
   ```

5. 환경 별로 설정 추가하기(위에 코드는 예시)
6. `DJANGO_SETTINGS_MODULE` 환경 변수는 Django가 어떤 설정 파일을 사용해야 하는지에 대해 사용됨
7. `export DJANGO_SETTINGS_MODULE=server.config.settings.development` 를 수행하면 `python [manage.py](http://manage.py) runserver` 개발 환경에서 수행 가능. `export DJANGO_SETTINGS_MODULE=server.config.settings.production` 서비스 서버에서 수행하면 `[production.py](http://production.py)` 가 수행됨

## **환경 변수 분리하기**

`settings.py` 를 분리하는 것과 마찬가지의 이유로 환경 변수는 따로 분리하여 관리하는 것이 좋다.

`.gitignore` 에 `.env` 파일을 넣어 놓는 것도 잊지 말도록 하자!

`django-environ` 과 `python-decouple` 라이브러리를 사용할 수 있다. 필자는 `django-environ` 을 사용하여 환경 변수를 분리한다.

### 설정 방법

1. 프로젝트 디렉토리에 `.env` 파일 생성
2. (git을 사용하는 경우) `.gitignore` 파일에 `.env` 추가하여 원격 저장소에서 관리하지 못하도록 설정
3. `pipenv install django-environ` 설치
4. `base.py` 다음 코드 추가하기

   ```python
   # base.py
   # env 설정
   import environ

   env = environ.Env()
   environ.Env.read_env(env_file=BASE_DIR / ".env")
   SECRET_KEY = env("SECRET_KEY")
   ```

5. `.env` 파일에 환경 변수 설정하기

   ```python
   SECRET_KEY="secret key"
   DATABASES={"default": {"ENGINE": "django.db.backends.sqlite3", "NAME": "db.sqlite3"}}
   ...
   ```

   <aside>
   ❓ **필자 TMI**
   테스트 서버와 프로덕션 서버를 분리하여 사용하는 경우, 필요 환경 변수가 다를 수 있다.
   그럴 때에는 `.env.development` `.env.production` 을 나누어 사용 가능하다.

   </aside>

## **CORS(Cross Origin Resource Sharing) 설정하기**

[cors 이해하기 참고 링크](https://evan-moon.github.io/2020/05/21/about-cors/)

CORS는 웹 브라우저의 보안 정책인 `Same-Origin Policy` 와 관련이 있다. 웹 사이트가 자신과 다른 출처에서 리소스를 로드할 때 발생하는 보안 문제를 방지하기 위해 설계되었다. 따라서 모바일 앱(Flutter, React Native …) 또는 동일한 도메인에서 호스팅 되는 경우에는 필요하지 않을 수 있다.

### CORS 설정 방법

1. `pipenv install django-cors-headers` 설치하기
2. `settings.py` 설정 추가

   ```python
   INSTALLED_APPS = [
   ...
   "corsheaders",
   ...
   ]

   MIDDLEWARE = [
   ...
   "corsheaders.middleware.CorsMiddleware",
   "django.middleware.common.CommonMiddleware",
   ...
   ]
   CORS_ALLOWED_ORIGINS = [...] # 허용하고자 하는 호스트 추가
   # CORS_ORIGIN_WHITELIST 변수는 3.5.0 version 이전에 사용됨
   # CORS_ORIGIN_ALLOW_ALL = True # 모든 호스트 허용
   # CORS_ALLOW_CREDENTIALS = True
   ```

   `Credential` 을 허용하면 은 쿠키, 인증 헤더 등을 다른 출처에서도 서버에 전송할 수 있게 한다.

## **TIME 설정하기**

[Time Zone Django 공식 문서](https://docs.djangoproject.com/en/4.2/topics/i18n/timezones/)

글로벌 진출을 고려한다면 타임존 문제를 미리 해결하고 가는 것이 좋다. **Django 공식 문서에서는 DB 데이터를 UTC로 저장하는 것을 권장한다.**

`USE_TZ = True` 를 설정하면, 시스템은 `aware datetime` 객체를 사용하고 날짜/시간 정보는 `TIME_ZONE` 설정에 지정된 시간대를 사용한다.

`TIME_ZONE` 설정은 Django 애플리케이션 내에서 날짜와 시간을 어떻게 다룰지 결정한다. 이 설정은 DB에 어떻게 저장되는지에 대해서는 직접적으로 제어하지 않는다.

```python
# base.py
TIME_ZONE = 'Asia/Seoul'
USE_TZ =  True
```

<aside>
❓ **필자 TMI**
`datetime` 은 머리에서 지우자. `django.utils.timezone` ****을 사용하도록 하자. 
********`datetime` 모듈은 `native datetime` 객체이므로 일괄되게 처리되지 않는다. `django.utils.timezone` 은 `awareness datetime` 객체이므로 일괄되게 처리한다.

</aside>

## **Static & Media 설정하기**

사람마다 헷갈리는 부분이 각자 다를 것이다. 필자는 이 부분이 왜 이렇게 이해가 안됐던 건지 모르겠다…

### static 설정 방법

1. `[base.py](http://base.py)` 설정 추가

   ```python
   # base.py
   # Static
   STATIC_URL = "/static/" # 절대 경로로 지정
   STATICFILES_DIRS = [
       os.path.join(BASE_DIR, "static"), # 쉼표 필수
   ] # 정적 파일을 찾는 장소
   STATIC_ROOT = os.path.join(BASE_DIR, "staticfiles") # collectstatic 수행 시 정적 파일을 모으는 장소
   ```

   `STATIC_URL` 은 정적 파일에 접근할 때 해당 경로로 요청을 보낸다.

   개발 서버는 이 URL 접두사를 사용하여 `STATICFILES_DIRS` 디렉터리의 정적 파일을 찾아 제공한다. 하지만 프로덕션 서버에서는 Django가 직접 서빙하지 않고, `Nginx` 와 같은 웹 서버가 정적 파일을 찾아 서빙하게 된다. Nginx 설정을

   ```python
   # nginx
   location /static/ {
       alias /path/to/your/project/staticfiles/;
   }
   ```

   다음과 같이 하게 되면 복잡성을 줄이고 최적화 할 수 있다.

   `STATICFILES_DIRS` 는 주로 개발 단계에서 사용한다. 개발 단계에서는 이 디렉터리에 있는 정적 파일을 그대로 제공한다.

   `DEBUG = True` 인 경우, 일반적으로 `STATIC_ROOT` 는 동작하지 않는다. 따라서 프러덕션 환경에서만 `collectstatic` 명령을 실행하여 웹 서버 ex) `nginx`에서는 `STATIC_ROOT` 정적 파일을 제공해야 한다.

   의문이 생겼다. 웹 서버 에서도 `STATICFILES_DIRS` 를 그래도 사용하면 되는거 아닌가? ⇒ 이럴 경우, 앱 내에 있는 `static` 디렉터리를 `STATICFILES_DIRS` 에 수동으로 추가해야 하고 최적화가 어려울 수 있다.

   말이 두서가 없지만, 필자가 생각하기에는 가장 이해하기 쉬운 내용이다.

### media 설정 방법

1. `[base.py](http://base.py)` 설정 추가

   ```python
   # base.py
   # Media
   MEDIA_URL = "/media/"
   MEDIA_ROOT = os.path.join(BASE_DIR, "media")
   ```

2. `server/urls.py` 수정

   ```python
   # urls.py
   from django.conf import settings
   from django.conf.urls.static import static

   urlpatterns = [
       # ... 기존 URL 패턴
   ]

   if settings.DEBUG:
       urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
   ```

   실제 배포 시에는 웹 서버 (예: Nginx, Apache)를 통해 미디어 파일을 제공해야 한다.

   Django는 `DEBUG = False` 일 때 미디어 파일을 직접 제공하지 않으므로 웹 서버 설정이 필요하다.

## 마치며

이렇게 기본적인 프로젝트 초기 세팅에 대해 설명했다.
더 괜찮은 내용이나 빼먹은 내용이 있으면 지속적으로 업데이트 예정이다.
