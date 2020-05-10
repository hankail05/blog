+++
title = "이맥스 개종 가이드 2 - 사람다운 이맥스 만들기"
date = 2020-05-10T15:38:00
tags = ["emacs"]
draft = false
thumbnail = "/images/real-programmers-use-emacs.png"
+++

순정 이맥스는 안 타는 쓰레기라서 그냥 쓰면 기대 수명이 짧아진다. 100세 시대를 이룩하기 위해 본격적으로 패키지를 깔고 설정하자.

<!--more-->

이맥스는 좋은 에디터이긴 하지만, 그렇다고 순정 이맥스가 좋다는 의미는 아니다. 키맵이 의미 기준으로 배열되어 있고 조합키 사용 빈도가 너무 높아 자주 쓰면 손목이 아프다. 빔이나 vscode가 순정으로도 쓸 만한 것돠는 대조된다. 물론 우리는 순정 상태의 기능을 최대한 쓰지 않을 것이다. 쓴다 하더라도 좀 더 누르기 편한 방식으로 조정할 것이다. 이 장에서 설명할 것들은 어떻게든 이맥스를 편하게 쓰려고 몸을 비튼 결과들이다.


## evil {#evil}

[evil](<https://github.com/emacs-evil/evil>) 은 vim의 그 키보드 레이아웃을 이맥스에 적용하는 패키지이다. 빔키맵은 워낙 유명해서 들어봤다 싶은 에디터에는 빔 키맵 플러그인이 다 있지만, 이맥스의 evil은 그중에서도 완성도가 높다. 하위 패키지로써 빔 플러그인을 포팅한 것들도 있을 정도다. 혹시 빔 키맵을 써보지 않았다면 이번 기회에 배워보는 걸 추천한다. `hjkl` 만 쓸 줄 알아도 손이 상당히 편해진다.

이 가이드에서 깔 패키지는 evil과 [evil-leader](<https://github.com/cofi/evil-leader>)이다. evil-leader는 리더키라는 하나의 키를 만들어 그 키를 기반으로 한 키맵을 찍어낼 수 있도록 해주는 빔 플러그인 포팅 패키지로, 최소한으로 키맵을 수정하면서 내가 원하는 키맵을 구축할 수 있다. 기존에 있는 키맵을 갈아엎어도 좋지만, 그렇게 되면 이맥스 레퍼런스(이맥스 키맵 수정 시) 또는 순정 빔(evil 키맵 수정 시)과 차이가 심해지기 때문에 본인은 리더키로 커스텀 키맵을 제작하는 편이다.

`M-x package-install RET evil RET` 과 `M-x pakcage-install RET evil-leader RET` 으로 두 패키지를 설치한 뒤, 다음 코드를 `init.el` 에 작성한다.

```emacs-lisp
(require 'evil)
(evil-mode 1)

(require 'evil-leader)
(global-evil-leader-mode t)
(evil-leader/set-leader "<SPC>")
```

`require` 는 패키지를 로드하는 함수이다. 앞으로 패키지를 로드하면 제일 먼저 작성해야하는 코드이니 별도로 적는다. 인자를 넣을 때 앞에 `'` 이 있어야하므로 유의하길 바란다.

위와 같이 하면 빔 키맵과 리더키를 이맥스에서 쓸 수 있다. 리더키는 insert state[^fn:1]를 제외한 나머지 state에 설정된다. 예시에서는 스페이스바를 리더키로 설정했지만, 본인이 원하는 키를 넣어도 상관없다.

이제 리더키를 이용해 간단한 세팅을 해보자. 가령 `C-x C-f` 를 누르기 고통스러운 사람들은

```emacs-lisp
(evil-leader/set-key
  "f" 'find-file)
```

을 `init.el` 에 입력하고 eval하면 `SPC f` 로 파일을 열 수 있다.


## ivy {#ivy}

ivy를 설치하면 패키지가 두 개 같이 온다. counsel과 swiper가 그것들이다. counsel은 이맥스의 기본적인 명령어를 ivy식으로 대체하고, swiper는 찾기 명령어를 ivy식으로 대체한다. 이 셋에 약간의 설정을 해주는 것으로, 더 나은 사용 경험을 구축할 수 있다.

`init.el` 에 다음 함수를 입력하고 eval하자.

```emacs-lisp
(require 'counsel)
(require 'swiper)
(setq ivy-wrap t
      ivy-re-builders-alist '((t . ivy--regex-fuzzy))
      ivy-use-selectable-prompt t)

(define-key ivy-switch-buffer-map "C-j" 'ivy-next-line)
(define-key ivy-switch-buffer-map "C-k" 'ivy-previous-line)
(define-key ivy-minibuffer-map "C-j" 'ivy-next-line)
(define-key ivy-minibuffer-map "C-k" 'ivy-previous-line)
(global-set-key (kbd "M-x") 'counsel-M-x)

(define-key evil-normal-state-map "C-s" 'swiper)
```

`setq` 는 그냥 변수 할당이다. 다만 위와 같이 여러 변수를 한꺼번에 할당할 수 있다. `define-key` 는 임의의 키에 함수를 할당한다. map은 다음 장에 설명한다.

위 함수는 ivy의 검색목록이 끝에 다다렀을 때 맨 위로 올려주고, 퍼지검색을 하도록 만들며, 목록 만이 아니라 입력한 것도 선택하게 해준다. 목록을 위아래로 움직일 때 `C-j` , `C-k` 을 누르도록 키를 설정하며, 찾기 명령어 (`C-s`) 를 ivy 사양으로 작동하도록 만들어준다.


## smex {#smex}

`M-x` 에서 히스토리 기반 정렬을 해주는 패키지 이다. 자주 쓰는 길고 긴 함수를 매번 입력할 필요 없이 목록을 조금씩 움직이면 바로 실행할 수 있기 때문에 정말 편하다. ivy를 설치했다면 별도의 설정 없이 설치만으로 적용된다.

`M-x package-install RET smex RET` 로 설치한 뒤 `init.el` 에 다음을 작성한다.

```emacs-lisp
(require 'smex)
(smex-initialize)
```


## undo-tree {#undo-tree}

이맥스에는 redo가 없다! 그래서 redo를 지원해주는 별도의 패키지를 설치해야한다. [undo-tree](<https://elpa.gnu.org/packages/undo-tree.html>)는 트리형식의 undo와 redo를 지원해, 우리가 흔히 아는 undo, redo는 물론이고, 브랜치를 옮기면서 여러 undo들을 redo할 수 있다.

undo-tree는 evil을 설치할 때 같이 설치되므로 따로 설치할 필요가 없다.

```emacs-lisp
(require 'undo-tree)
(global-undo-tree-mode t)
(evil-leader/set-key
  "u" 'undo-tree-visualize)
```

상기 코드를 작성하고 eval 하자. 이제 `SPC u` 로 현재 undo, redo 상태를 시각화할 수 있다.

{{< figure src="/images/undo-tree.png" >}}

이 상태에서 `k` 로 undo, `j` 로 redo할 수 있으며, `h` 와 `l` 로 브랜치를 바꿔 undo, redo할 내용을 바꿀 수 있다.

이렇게 하면 이맥스는 순정 폐기물에서 빔 정도의 쓸만한 에디터로 변모한다. 이 상태로 텍스트 파일을 편집하는 데에 큰 지장은 없지만, 이 글을 읽는 여러분의 목적은 이맥스로 코딩하기일 것이다. 다음 장에서는 파이썬을 예시 삼아 이맥스에서 코딩 환경을 세팅하는 방법을 설명할 것이다.

[^fn:1]: evil 에서는 빔의 모드를 state 라고 한다. 이맥스에는 모드가 따로 있다. 다음 편에 작성
