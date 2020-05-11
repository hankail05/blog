+++
title = "이맥스 개종 가이드 4 - 기타 유틸리티"
date = 2020-05-11T20:47:00
tags = ["emacs"]
draft = false
thumbnail = "/images/real-programmers-use-emacs.png"
+++

작성자 엄선 이맥스 패키지를 마구 깔아서 편의성을 극대화해보자

<!--more-->

이 장은 본인이 엄선한 패키지를 나열한 페이지다. 자신은 추천도 필요없고 스스로 다 비교해보면서 구축하겠다는 사람은 이 장을 건너뛰면 된다. 근데 그러면 이 글을 쓴 의미가... 아악


## leaf.el {#leaf-dot-el}

[leaf.el](<https://github.com/conao3/leaf.el>)은 패키지 관리를 보다 쉽게 할 수 있도록 도와주는 패키지 이다. 이 패키지를 사용하면 패키지 하나를 설정할 때, 코드들을 하나의 블럭으로 묶을 수 있어 `init.el` 파일이 간략하고 깔끔해진다.

가령 본인의 company 설정

```emacs-lisp
(require 'company)

(add-hook 'prog-mode-hook 'company-mode)
(add-hook 'org-mode-hook 'company-mode)

(setq company-idle-delay 0)
(setq company-minimum-prefix-length 2)
(setq company-backends '((company-dabbrev-code :separate company-capf company-keywords)
                         company-files
                         company-keywords
                         company-capf
                         company-yasnippet
                         company-abbrev
                         company-dabbrev))

(setq company-echo-truncate-lines t)
(setq company-tooltip-align-annotations t)

(define-key company-active-map (kbd "TAB") 'company-complete)
```

을

```emacs-lisp
(leaf company
  :ensure t
  :hook (((prog-mode-hook org-mode-hook) . company-mode))
  :setq ((company-idle-delay . 0)
         (company-minimum-prefix-length . 2)
         (company-backends . '((company-dabbrev-code :separate company-capf company-keywords)
                               company-files
                               company-keywords
                               company-capf
                               company-yasnippet
                               company-abbrev
                               company-dabbrev))
         (company-echo-truncate-lines . t)
         (company-tooltip-align-annotations . t))
  :bind (:company-active-map
         ("<tab>" . company-complete)))
```

이런 식으로 깔끔하게 만들 수 있다. leaf의 인자로 패키지를 받을 경우 `require` 로 패키지를 로드하는 것과 같은 효과를 낸다. leaf의 대부분의 키워드는 변수와 변수에 집어넣을 값을 `.` 으로 구분한다. 다음은 leaf를 쓸 때 외워두면 좋은 키워드를 나열한 것이다.

```text
:hook (some-hook . some-function) - (add-hook 'some-hook 'some-function) 과 같음
:setq (some-variable . some-value) - (setq some-variable some-value) 와 같음
:bind (some-keymap
       (some-key . some-function)) - (define-key some-keymap some-key 'some-function) 와 같음
:init (some-function) - 패키지가 로드 되기 이전에 실행되는 함수
:config (some-function) - 패키지가 로드 된 이후 실행되는 함수. 상기 키워드를 제외한 나머지를 넣으면 됨
:load-path (some-path) - 패키지 매니저로 설치하지 않은 패키지를 로드할 때 사용. 다음 장에 설명
```

패키지 초기화 함수 같은 경우에는 패키지가 로드된 뒤 실행해야 하므로 `:config` 키워드 뒤에 붙이면 된다.


## 괄호 관련 패키지 {#괄호-관련-패키지}

우리 모두 괄호에 대해 안 좋은 경험을 갖고 있을 것이다. 페어 안 맞는다고 터지고, 괄호를 일일이 지우고 새로 붙인다고 왔다갔다 거리고, 리슾 프로그래밍을 한 사람은 무수한 괄호의 홍수에 파묻혔던 기억도 있을 것이다.
![](https://imgs.xkcd.com/comics/lisp%5Fcycles.png)

다행히, 동서고금을 통틀어 괄호에 대해 학을 뗀 사람들은 에디터에 상관없이 있고, 그들이 만든 좋은 패키지 역시 있다. 이맥스도 마찬가지다.

[smartparens](<https://github.com/Fuco1/smartparens>)는 괄호 삽입 및 수정을 다루는 패키지이다. 괼호 위치 이동, 괄호 삽입/교체/제거, 이어붙이기, 등등 좋은 기능도 많지만, 가장 중요한 기능은 strict pairing이다. 괄호를 열면 커서 바로 옆에 닫아주고, 괄호를 반드시 짝 맞추기 때문에 백스페이스 연타해도 실수로 괄호를 지울 염려가 없다.

smartparens는 문서에 예제가 잘 되어 있어 여기서 다루지 않겠다. 다만 제대로 써먹기 위해선 약간 설정이 필요해 그것만 언급하겠다.

```emacs-lisp
(leaf smartparens
  :init (require 'smartparens-config)
  :setq ((sp-ignore-modes-list . (delete 'minibuffer-inactive-mode sp-ignore-modes-list))
         (sp-escape-quotes-after-insert . nil))
  :config
  (smartparens-global-strict-mode)
  (sp-local-pair 'minibuffer-inactive-mode "'" nil :actions nil)
  (when (version<= "27" emacs-version)
    (dolist (parens '(c-electric-paren c-electric-brace c-electric-slash))
      (add-to-list 'sp--special-self-insert-commands parens))))
```

위와 같이 설정해줘야 C에서 괄호가 정상 작동하며 emacs-lisp을 다룰 때 `'` 가 하나만 찍힌다. 리슾계열에서 `'` 는 하나만 쓰므로 위와 같이 설정해줘야 오류를 뿜지 않는다. evil 유저는 [evil-smartparens](<https://github.com/expez/evil-smartparens>)를 깔아야 evil과 잘 굴러간다.

다음 설명할 패키지는 [evil-surround](<https://github.com/emacs-evil/evil-surround>)이다. 빔 플러그인이 포팅된 패키지로, 괄호 삽입/교체/삭제 만 따지자면 evil에 더 잘 녹아들어 편하다. 이 패키지 역시 문서에 예제가 잘 되어있어 언급하지 않겠다.

마지막 패키지는 [rainbow-delimiters](<https://github.com/Fanael/rainbow-delimiters>)이다. 괄호쌍을 색칠해주는 패키지로, 이것만 있으면 괄호의 늪을 해쳐나갈 수 있다.

```emacs-lisp
(leaf rainbow-delimiters
  :straight t
  :hook ((prog-mode-hook org-mode-hook) . rainbow-delimiters-mode))
```

로 설치만 해두면 괄호가 알록달록해져 코드 블록이 더 잘 구분된다.

{{< figure src="/images/parens.png" >}}


## yasnippet {#yasnippet}

[yasnippet](<https://github.com/joaotavora/yasnippet>)은 이맥스의 템플릿 패키지이다. 다양한 언어를 지원하며, 축약된 단어를 입력하고 TAB을 누르거나 `M-x yas-insert-snippet` 으로 템플릿을 넣을 수 있어 보일러플레이트를 입력하기 편해진다.


## pdf-tools {#pdf-tools}

[pdf-tools](<https://github.com/politza/pdf-tools>)는 이맥스에서 pdf를 렌더링해주는 패키지이다. 기본으로 지원하는 Docview보다 성능이 준수하고 기능성도 더 좋아서 이것을 쓴다. 굳이 다른 걸 냅두고 이맥스 내에서 뷰어를 쓰는 이유는 키맵을 그대로 끌어다 끌 수 있기 때문이다. 이것은 다른 패키지(메일, 뮤직 플레이어 등[^fn:1])에도 그대로 적용된다.

pdf-tools는 설치 과정이 하나 더 있다. 평소대로 패키지 설치를 한 뒤, 다음 함수를 실행해야 정상적으로 작동한다.

```emacs-lisp
(pdf-tools-install)
```


## which-key {#which-key}

[which-key](<https://github.com/justbur/emacs-which-key>) 키 입력 기반 치트시트 패키지이다. 매 번 키맵을 까먹을 때 마다 레퍼런스나 설정 파일을 열 필요 없이 그냥 기억나는 키 시퀀스를 입력하면 그에 해당되는 키맵을 화면 아래에 보여주기 때문에 키맵에 익숙해지는 데에 큰 도움을 준다. 한가지 팁이 있는데, prefix가 없는 키맵을 보고 싶다면 which-key 치트시트를 연 뒤 `C-h u` 를 입력하면 된다.


## 테마 {#테마}

의외로 테마도 중요한 요소이다. 하루종일 화면만 들여다 볼텐데 배경이 하얗거나 폰트 컬러링이 엉망이면 눈이 썩어들어갈 것이다. 따라서 적절한 테마를 찾아서 설치해야 눈을 보호하는 지름길로 나아갈 수 있다. 본인은 배경이 검은 테마가 없어 직접 테마를 작성했지만 처음부터 테마를 작성하는 짓은 가혹하므로 둘러봤던 테마 중 하나를 추천한다. 바로 [doom-themes](<https://github.com/hlissner/emacs-doom-themes>)의 one-dark 테마이다. 구성 자체도 깔끔하고, 많은 패키지의 테마를 지원해 어지간한 패키지에 다 적용된다.


## evil 하위 패키지 {#evil-하위-패키지}

기본 단축키를 확장해 키보드를 연타할 일을 없애주는 [evil-easymotion](<https://github.com/PythonNut/evil-easymotion>), 텍스트 교환을 수월하게 만들어주는 [evil-exchange](<https://github.com/Dewdrops/evil-exchange>), 코드를 쉽게 주석으로 만들고 풀어주는 [evil-nerd-commenter](<https://github.com/redguardtoo/evil-nerd-commenter>), 동일한 단어들을 한번에 수정하도록 도와주는 [evil-multiedit](<https://github.com/hlissner/evil-multiedit>) 등이 있다.

추천 패키지는 이 정도다. 나머지는 그리 자주 안 쓰거나 굳이 필요없다고 생각했거나 까먹은 것들이다. 다른 패키지는 스스로 찾아보자. 그리고 더 좋은 패키지를 발견하면 널리 공유하자. 다음이자 마지막 장에서 설정할 때 유용한 팁으로 가이드를 마치겠다.

[^fn:1]: 웹브라우징 제외. css를 지원하지 않는다...
