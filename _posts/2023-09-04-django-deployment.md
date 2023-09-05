---
title: Django - AWS EC2 & Nginx & Gunicorn 배포하기
date: 2023-09-04 00:34:00 +0900
author: devkya
categories: [Django]
tags: [python, django, setting, deployment]
---

# 🙂**들어가며...**

`AWS EC2` `ubuntu 22.04` 환경에서 `Nginx & Gunicorn`을 사용하여 `Django`를 배포하고자 한다.

# **세팅**
## **git을 사용하여 ubuntu에 프로젝트 클론하기**

`AWS EC2 & RDS`를 설정이 되어 있다고 가정한다.  
`git`을 사용하여 로컬 환경에서 개발하였던 프로젝트를 클론하여 EC2 환경에 가져온다.  
`/srv/` 폴더에 클론을 하기 전에, 해당 폴더는 root 사용자의 소유이기에 현재 사용자에 대한 권한을 부여해야 한다.  

1. `sudo chown -R $(whoami):$(whoami) /srv/` 명령어를 수행하여 권한을 변경한다.
2. `git clone [repo]` 레포지토리를 복제한다.

## **`pyenv`와 `pipenv`를 사용하여 가상환경 구축**

python 버젼 관리와 가상환경 구축을 위해 필자는 보통 `pyenv` & `pipenv`를 사용한다.  
ubuntu 환경에 설치를 진행한다.

1. `pyenv` 설치를 위해 필요한 패키지를 설치한다.
    ```bash
    sudo apt-get update
    sudo apt-get install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
    libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
    xz-utils tk-dev libffi-dev liblzma-dev python3-openssl git

    ```

2. `curl https://pyenv.run | bash` 명령어를 수행하여 `pyenv`를 설치한다.
3. `pyenv` 환경변수를 설정한다.
    ```bash
    export PATH="$HOME/.pyenv/bin:$PATH"
    eval "$(pyenv init --path)"pip
    eval "$(pyenv virtualenv-init -)"
    ```
4. 변경사항 적용을 위해 쉘을 재시작하거나, `source ~/.bashrc` 명령어를 수행한다.
5. 프로젝트에 맞는 파이썬 버젼 설치 및 관리(현재 ec2에서 하나의 프로젝트만 수행하므로, global로 설정함)
    ```
    pyenv install 3.11.2
    pyenv global 3.11.2
    ```
6. `pip install pipenv` pipenv 패키지 설치
7. 프로젝트에 `pipenv.lock` 디렉토리로 이동 후 `pipenv install & pipenv shell` 수행


# **`Gunicorn` & `Nginx` 세팅**
## **`Gunicorn`**
1. `sudo vi /etc/systemd/system/gunicorn.service` 생성 및 편집  
`gunicorn.sock` 파일은 자동으로 생성된다. /run/ 폴더로 지정하면 권한 문제를 해결해야 한다.  
가상환경 위치는 `pipenv --venv` 명령어로 찾을 수 있다.
    ```
    [Unit]
    Description=Deployment Server
    After=network.target

    [Service]
    User=ubuntu
    Group=www-data
    WorkingDirectory=[manange.py의 디렉토리]
    ExecStart=[가상환경 위치]/bin/gunicorn \
              --access-logfile - \
              --workers 1 \
              --bind unix:/tmp/gunicorn.sock \
              server.wsgi:application

    [Install]
    WantedBy=multi-user.target
    ```
    

2. `sudo systemctl start gunicorn.service & sudo systemctl enable gunicorn.service` 명령어 실행 
3. `sudo systemctl status gunicorn.service` 상태 확인 -> active 상태가 정상이다.


## **`Nginx`**
1. `sudo apt install nginx python3-pip python3-dev libpq-dev` 패키지 설치
2. `sudo vi /etc/nginx/sites-available/server` 파일 생성 및 편집  
    * static 파일 설정에 따라 다르지만, `alias`가 아닌 `root`로 설정할 경우 웹서버가 `staticfiles/static` 이런 식으로 정적 파일을 찾는다.  
    * 필자는 ssl 인증서를 사용하여 https 통신을 하기 때문에 if문이 추가되었다. SSL 인증 방법은 다음 포스트에서 설명할 것이다.
        ```
        if ($http_x_forwarded_proto = 'http'){
                return 301 https://$host$request_uri;
            }
        ```
    
    ```
    server {
        listen 80;
        server_name [IP or Domain];

        
        location /static/ {
            alias [프로젝트 디렉토리]/staticfiles/;
        }

        location / {
            include proxy_params;
            proxy_pass http://unix:/tmp/gunicorn.sock;
        }

        if ($http_x_forwarded_proto = 'http'){
            return 301 https://$host$request_uri;
        }
    }
    ```

3. `sudo ln -sf /etc/nginx/sites-available/server /etc/nginx/sites-enabled` 복사
4. `sudo nginx -t` -> Nginx 구성 검사
5. `sudo systemctl restart nginx` -> Nginx 재시작
6. `sudo ufw allow 'Nginx Full'`

## **Error log 확인 방법**
1. Gunicorn
    * `sudo systemctl status gunicorn.service` gunicorn 상태 확인
    * `sudo journalctl -u gunicorn.service -n 100 --no-pager` 최근 100개의 로그 확인
    * `sudo systemctl daemon-reload & sudo systemctl restart gunicorn.service` 재시작 명령어
2. Nginx
    * `cat /var/log/nginx/error.log` nginx 명령어 확인

## **도움이 되는 세팅**
1. 시간 설정
    * `sudo timedatectl set-timezone Asia/Seoul`
2. 사용자 확인
    * `whoami`
3. 폴더 권한 설정  
    `/srv/` 폴더에 권한 설정할 때, 
    * `sudo chown ubuntu:ubuntu /srv/`
4. (프로젝트의 환경을 나눈 경우)`production.py` 적용을 위해,
    * `export DJANGO_SETTINGS_MODULE=server.config.settings.production` => 터미널 세션이 종료되면 다시해줘야 해서 귀찮다
    * `~/.bashrc` 또는 `~/.bash_profile` 수정: Bash 쉘을 사용하는 경우, 해당 파일에 export DJANGO_SETTINGS_MODULE=server.config.settings.production 라인을 추가하자





# **마치며**
내용을 조금 더 검토하고 수정해야 한다. 아마도 따라해보면 잡다한 에러들이 발생할 것이다. 댓글에 남겨주거나, 구글링을 통해 해결하도록 하자!
