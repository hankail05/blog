+++
title = "이맥스 개종 가이드 3 - 개발 환경 구축"
date = 2020-05-10T21:42:00
tags = ["emacs"]
draft = false
thumbnail = "/images/real-programmers-use-emacs.png"
+++

이맥스는 마음만 먹으면 다채로운 기능을 가진 에디터로 만들 수 있다. 파이썬을 예시로 이맥스를 ide 비슷하게 개조해보자.

<!--more-->

이 장에서는 본격적으로 프로그래밍에 관련된 이야기를 할 것이다. 간단하게 파이썬 개발 환경을 구축하고 깃을 활용 하는 것을 예시로 들 것이다. 본인이 다른 언어를 주력으로 한다면 맨 밑에 나열된 패키지 목록을 참고해서 직접 설치해 보자.


## 모드 {#모드}

전술했듯 이맥스의 모드는 빔의 그것과는 다르다. 이맥스의 모드는 `매이저 모드(major-mode)` 와 `마이너 모드(minor-mode)` 로 나뉜다. 메이저 모드는 현재 버퍼에서 어떤 편집 기능을 활성화할 지 결정한다. c 소스코드에서는 c 전용 기능을 활성화하는 매이저 모드가, 파이썬 소스코드에서는 파이썬 전용 기능을 활성화 하는 매이저 모드가 선택되는 식이다. 매이저 모드는 특별한 패키지가 없다면 버퍼 당 단 하나만 가질 수 있다. 마이너 모드는 매이저 모드 밑에서 활성화 되는 모드로, 어떤 것들은 버퍼 단위로, 어떤 것들은 전역에 활성화된다. 마이너 모드는 갯수 제한이 없고, 이 마이너 모드들을 매이저 모드와 엮어서 매이저 모드가 활성화될 때 같이 활성화되도록 만들 수 있다. 이러한 작업을 `hooking` 이라고 하며, 이맥스 개발 환경 구축의 핵심이다.

{{< figure src="/images/modes.png" >}}

파이썬 파일을 열면 이렇게 괄호 안의 첫번째에 메이저 모드, 그 뒤로 마이너 모드가 표시된다. 보통 모드에는 테마, 키맵, 훅 등이 들어있다.

내장된 파이썬 매이저모드만으로도 하이라이팅, repl, 커스텀 키맵 `(C-c)` , 문서화 등을 제공해 idle 정도로 써먹을 수는 있지만, 그래서는 ide를 참칭할 수가 없다. 개조하자.


## LSP {#lsp}

LSP 는 Language Server Protocol의 약자로, 마이크로소프트가 개발한 프로토콜이다. 에디터에 구애받지 않는 별도의 서버를 실행하고 그 서버와 통신해 모든 에디터에서 일관적인 개발 환경을 제공받을 있도록 해준다. 따라서 vscode에서든, 빔에서든, 이맥스에서든 똑같은 자동완성, 린팅, 스코프 기반 리네이밍 등의 기능을 활용할 수 있다.

LSP를 설치하는 과정은 조금 복잡하다. 우선 이맥스에서 LSP와 통신하기 위한 클라이언트를 설치해야한다. `M-x package-install RET lsp-mode RET` 를 입력해 설치하자. 그 다음으로 필요한 것은 파이썬 LSP 서버이다. 터미널을 열고 다음을 입력한다.

```text
pip install 'python-language-server[all]'
```

이제 클라이언트와 서버를 연결해주면 된다.

```emacs-lisp
(require 'lsp-mode)
(require 'lsl-clients)
(setq lsp-prefer-flymake nil)
(add-hook 'prog-mode-hook 'lsp-deferred)
```

모드에 hook을 달아놓으면 그 모드가 활성화 될 때 hook에 걸린 함수도 한꺼번에 실행된다. prog-mode-hook은 프로그래밍 관련된 모든 매이저 모드가 실행될 때 발동되는 hook이다. 즉 위 코드는 파이썬 모드를 활성화 하면 LSP를 작동시키는 것과 같다.

lsp-mode에서는 기본 키맵을 지원한다. 다른 패키지들도 키뱁을 지원하는 경우가 일반적이다. 다만 lsp는 그 키맵이 조금 불편하다. 무려 `S-l` 에서 파생되여, 윈도우든 리눅스든 시스템 단축키와 쉽게 겹쳐버린다. 따라서 우리는 이 키맵을 살짝 수정할 것이다. 마침 `S-l` 을 이후의 키맵은 조합키를 쓰지 않으니 키맵을 그대로 들고 다른 단축키에 집어넣자. 간단하게 패키지 내에 있는 키맵을 찾아서 기존과 같이 할당해주면 된다. 이 상황에서는 다음과 같다.

```emacs-lisp
(evil-leader/set-key
  "l" 'lsp-command-map)
```

위 코드를 eval하면 이제 prefix로 `S-l` 대신 `SPC l` 를 입력할 수 있다. 이렇게 키맵을 하나씩 지정하는 것 말고도 기존 키맵을 다른 단축키에 박아버리는 방법도 존재한다.

LSP를 켜는 단축키는 `<prefix> s s` , 함수를 정의를 찾으려면 `<prefix> g g` 를 입력하면 된다. 자세한 키맵은 [여길](<https://emacs-lsp.github.io/lsp-mode/page/keybindings/>) 참고하면 된다.


## company {#company}

[company](<https://github.com/company-mode/company-mode>)는 이맥스에서 가장 유명한 자동완성 패키지이다. company 자체는 백엔드에서 받은 데이터를 현재 입력에 맞춰 보여주기만 하므로, 언어별 데이터를 제공해주는 백엔드(여기서는 LSP)를 설치해야한다. 긴 말이 필요한가. 바로 설치하자.

`M-x package-install RET company RET` 과 `M-x package-install RET company-lsp RET` 으로 필요한 패키지를 설치한다. 그 후

```emacs-lisp
(require 'company)
(add-hook 'prog-mode-hook 'company-mode)
(setq company-backends '((company-dabbrev-code :separate company-capf company-keywords)
                         company-files
                         company-keywords
                         company-capf
                         company-yasnippet
                         company-abbrev
                         company-dabbrev))
(define-key 'company-active-map (kbd "TAB") 'company-complete)

(require 'company-lsp)
(add-to-list 'company-backends 'company-lsp)
```

로 company를 실행하고 LSP에서 받은 데이터로 자동완성할 수 있다. `add-to-list` 함수는 리스트 앞에 원소를 추가하는 함수로, 한 번에 리스트를 만들면 에러가 날 때 저렇게 나눠서 리스트를 생성하면 된다. 백엔드 순서를 저렇게 세팅하면 최근 자동완성한 목록이 제일 위에 뜨므로 편하다.


## flycheck {#flycheck}

[flycheck](<https://github.com/flycheck/flycheck>)는 린팅 패키지이다. company와 같이 백엔드에서 받은 데이터가 있어야한다. company와 달리 LSP와 연돌할 때에는 별도로 설치할 패키지는 없다. `M-x package-install RET flycheck RET` 로 설치한 후, 다음과 같이 설정한다.

```emacs-lisp
(require 'flycheck)
(add-hook 'prog-mode-hook 'flycheck-mode)
```

LSP와 연동하여 오류를 뿜어내는 우리의 코드를 볼 수 있을 것이다. 혹시 위에 `lsp-prefer-flymake` 변수를 눈치챘는가? 저 변수는 또 다른 린터인 flymake를 사용하는지 여부를 결정한다. flymake는 내장패키지지만, 워낙 오래되기도 했고, 이게 더 좋다.

company와 flycheck 백엔드는 LSP만 있는 것이 아니다. LSP에는 없는 언어도 백엔드가 있고, 심지어는 cmake같은 설정파일을 지원하기도 한다.


## 가상환경 지원 {#가상환경-지원}

파이썬은 그놈의 가상환경 때문에 조금 더 설정을 해줘야한다. 기껏 파이썬 개발할 준비를 다 했는데 이맥스 내에서 가상환경 이 돌아가지 않으면 슬플 것이다. 그런 상황을 미연에 방지해주는 고마운 패키지가 바로 [virtualenvwrapper](<https://github.com/porterjamesj/virtualenvwrapper.el>)이다. 그리고 이와 연동해 가상환경을 자동으로 활성화해주는 패키지로 [auto-virtualenvwrapper](<https://github.com/robert-zaremba/auto-virtualenvwrapper.el>)가 있다.

`M-x package-install RET virtualenvwrapper RET` 와 `M-x package-install RET auto-virtualenvwrapper` 로 설치한 뒤,

```emacs-lisp
(require 'viraulenvwrapper)
(require 'auto-virtualenvwrapper)
(add-hook 'python-mode-hook '(lambda ()
                               (unless (eq buffer-file-name nil)
                                 (setq-local venv-location (directory-file-name buffer-file-name)))))
(add-hook 'python-mode-hook 'auto-virtualenvwrapper-activate)
```

로 설정하면 파이썬 파일을 열었을 때 알아서 프로젝트 루트에 있는 `venv` 디렉토리를 찾아 가상완경을 활성화한다.


## 기타 파이썬 프로그래밍에 도움되는 것들 {#기타-파이썬-프로그래밍에-도움되는-것들}

`C-c C-p` 로 파이썬 쉘을 실행할 수 있다. 그 후 코드를 드래그(혹은 visual state로 선택)한 뒤 `C-c C-r` 을 누르면 지정된 소스코드가 쉘로 전송되어 인터프리터에 로드된다. 이로인해 이맥스에서는 repl이 쉽다.

[pip-requirements](<https://github.com/Wilfred/pip-requirements.el>) 로 requirements.txt 전용 모드를 설정할 수도 있고, [pytest.el](<https://github.com/ionrock/pytest-el>)로 pytest를 이맥스 안에서 돌릴 수 있다. [디버거도 지원한다.](<https://github.com/realgud/realgud>)


## git tool {#git-tool}

[magit](<https://github.com/magit/magit>)은 이맥스의 킬러 앱 중 하나이다. 깃을 이맥스에서 사용하게 해줄 수 있는 패키지인데, 매우 강력한 기능성을 갖추고 있다. 키보드 몇 번 누르는 것 만으로 깃의 거의 모든 기능을 쓸 수 있으며, 그 인터페이스 또한 혁신적이다.[^fn:1] 다만 깃의 모든 기능을 끌어다 쓰기 때문에 매뉴얼 또한 길다. 따라서 이 가이드에서는 가장 기초적인 add, commit, push 만 다루겠다.

우리는 evil을 쓰기 때문에 magit의 키맵을 evil사양으로 바꿔주는 evil-magit도 같이 깔 것이다. `M-x package-install RET magit RET` 와 =M-x package-install RET evil-magit RET=으로 설치한다.

그 후

```emacs-lisp
(require 'magit)
(evil-leader/set-key
  "m" 'magit-status)

(require 'evil-magit)
(evil-magit-init)
```

을 입력하면 설정이 끝난다. magit-status의 기본 키맵은 `C-x g` 이지만, 위와 같이, 혹은 자기 마음대로 설정해도 된다. magit-status 함수를 실행하면 아래와 같은 화면이 뜰 것이다.

{{< figure src="/images/magit-status.png" >}}

스테이지[^fn:2]하고 싶은 파일에 커서를 올리거나 visual-state로 긁은 뒤 `s` 를 누르면 스테이징 된다. 전체를 스테이지 하고 싶으면 `S` 를 누르면 된다.

{{< figure src="/images/magit-stage.png" >}}

그 후 `c` 를 누르면 커밋 설정이 뜬다.

{{< figure src="/images/magit-commit-menu.png" >}}

여기서는 그냥 커밋을 할 것이니 `c` 를 누른다. 그러면 커밋메세지를 입력하는 창과 diff창이 뜬다.

{{< figure src="/images/magit-commit-confirmation.png" >}}

커밋 메세지를 입력하고 `C-c C-c` 를 누르면 커밋이 완료된다. 이제 푸시를 해보자. 푸시를 하려면 `p` 를 누른다.

{{< figure src="/images/magit-push-menu.png" >}}

이때 upstream을 지정하지 않았다면 지정해달라 할 것이고, 아니라면 `p` 를 다시 눌러서 푸쉬하면 된다. ssh 토큰을 발행해놨다면 바로 푸쉬될 것이고, 아니라면 아이디와 비밀번호를 입력하면 된다.

이외에도 머지, 리베이스, 브랜치 관리 등을 전부 magit 내에서 할 수 있고, 하위 패키지로써 깃헙과 연동하거나 TODO를 status에 띄워주는 패키지도 있다.

이 정도면 코딩하는 데에 크게 불편한 점은 없을 것이다. 불편하다해도 자신이 원하는 기능을 제공하는 패키지를 깔아서 쓰거나 적당히 리슾코드를 작성하면 된다. 하지만 이맥스에는 훌륭한 패키지들이 아직도 있다! 뉴비들의 입에 가득히 넣어주고 싶을 정도로 많다! 그런 작성자 추천 패키지들은 다음 장에서 소개하도록 하겠다.

[^fn:1]: [너무나 좋은 나머지 별도의 패키지로도 나와있다.](<https://github.com/magit/transient>)
[^fn:2]: add == stage
