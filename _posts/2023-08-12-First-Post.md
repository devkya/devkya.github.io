---
title: Python Black Formmater 사용하기(VsCode)
date: 2019-08-11 00:34:00 +0900
author: devkya
categories: [Django]
tags: []
---

#

# 🙂들어가며...

일반적인 블로그에서 가상환경을 생성하고 (필자는 pipenv를 사용하고 있음) `pipenv install black` 로 `black` 패키지를 설치하고 (1) vscode에서 `python > Formatting: Provider` 를 `black` 으로 변경 (2) `Format On Save` 을 체크하면 활성화가 된다고 한다.

필자도 처음에는 잘 작동하였으나, 어떤 이유에서인지 `black` 패키지는 설치되었고 터미널에서 커맨드를 통해서는 동작하지만 저장 시 활성화가 안되는 문제가 발생하였다.

(아직까지 원인을 파악하지 못했다. 이유를 아시는 분은 댓글에 남겨주시길 바란다.)

VsCode 설정(User, Workspace …)이 뭔가 꼬였다고 판단된다. **스택오버플로우**의 해결 방안들도 모두 시도해보았지만 해결되지 않았다.

(해당 링크 : **[Formatter black is not working on my VSCode...but why?](https://stackoverflow.com/questions/65101442/formatter-black-is-not-working-on-my-vscode-but-why))**

혼자 작업을 하기에 코드 포맷을 누군가와 공유하는 일이 적기 때문에 VsCode Extensions의 `Black Fommater` 를 활용하기로 했다.

# 🛠트러블 슈팅

[공식문서를 참조하자!](https://github.com/microsoft/vscode-black-formatter)

아래 방법이 잘못되었을 수도 있으니, 공식 문서를 반드시 확인하자…

1. `pip install black` 을 수행하였을 때 VsCode Workspace 세팅이 변경된다. `.vscode` 폴더 또는 settings.json을 `"python.formatting.provider": "none"` 으로 수정

   <aside>
   ❗ **참고사항 - VsCode 설정 우선순위**
   .vscode 폴더 설정 → 워크스페이스 → 사용자 설정

   </aside>

![vscode setting](/assets/img/posts/vscode-setting.png){: width="700" height="400" }

2. VsCode Extension `black formmater` 설치

![black fomatter](/assets/img/posts/black-fomatter.png){: width="700" height="400" }

3. `cmd/ctrl` + `shift` + `p` 를 눌러 settings.json(user) 열기

   아래 코드 붙여넣기(`"python.formatting.provider": "none"` 인지 확인)

   ```json
   "[python]": {
       "editor.defaultFormatter": "ms-python.black-formatter",
       "editor.formatOnSave": true
     }
   ```

4. 작동됨

# 👋🏻여담

공식 문서에서도 `black formatter` 는 실험적이라고 나와 있다. 지속적으로 업데이트하고 있는 것으로 보인다.

문제 해결이 안될 시 사용해보면 좋을듯하다.
