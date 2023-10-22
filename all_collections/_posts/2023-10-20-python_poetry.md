---
layout: post
title: 🧾 Python poetry로 가상환경을 만들어보자
date: 2023-10-22
categories: ["SamBeak", "Python", "Poetry"]
---

종사자가 되면서 <br>
여간 해보지 않았던 자바를 공부하느라 <br>
블로그에 정리할 시간이 여간 쉽지 않았다. <br><br>

여전히 새로운 것을 익히고 풀어간다는 것은 <br>
여느 때보다 설레는 일이기에 <br>
Python의 poetry를 정리하고자 한다. <br>

> # Poetry란 무엇인가?

<br>
포트리는 가상환경을 제공하는 역할을 수행한다. <br>
자바를 처음 배울 때 기억이 생생한데 <br>
그 이유는 환경설정 때문이다. <br><br>

톰캣 서버를 설치하면서부터 <br>
버전을 선택하고, 환경 변수를 설정하는 과정이 <br>
리액트를 공부하다 넘어갔을 때 상당히 이해하기 어려웠다. <br><br>

호환성의 문제가 있는 것인지 <br>
버전이 다르다는 이유로 왜 오류가 발생하는 것인지 <br>
그렇다면, 버전이 다를 때는 어떻게 설정을 해줘야하고 <br>
오류가 발생하지 않으려면 어떻게 해야하는 것인지 <br>
이해하고 생각할 것이 너무나도 많았었다. <br><br>

근데 이게 알고보니 자바에 국한된 것이 아니더라 <br>
어디서나 고려해야할 부분이었다. <br>
장고도 여러 버전을 전역으로 설치할 필요 없이 <br>
포트리를 사용하면 가상환경을 제공받을 수 있다. <br><br>

> # 포트리 설치하기

<br>
Mac은 Home brew 를 사용해서 설치하는게 <br>
환경변수 설정이라든지, 설치 과정의 오류라든지 <br>
복잡하지 않게 설치할 수 있어 권장한다. <br>

```
brew intall poetry
```

Window는 파이썬 설치가 되었다면, <br>
파이썬 명령어로 설치하는 것을 권장한다. <br><br>

```
pip install poetry
pip3 install poetry
```

이 외 방법을 사용하여 설치하려면 <br>
포트리 사이트에 접속하여 명령어를 참고해 <br>
터미널에서 설치하도록 한다. <br><br>

> # 가상환경에 설치하기

<br>

포트리를 시작하면 가상환경을 설정해야한다.<br>

우선 프로젝트의 터미널에서 입력한다. <br>

```
poetry init
```

licence는 MIT로 설정하고, <br>
main dependencies = no <br>
develop dependencise = no <br>
confirm = yes <br>
를 차례로 입력해준다. <br>
이러면 나의 가상환경이 생성된다. <br><br>

가상환경이 생성되었다면 <br>
django를 설치해주자 <br>

```
poetry add django
```

이렇게 가상환경에 추가 된 것들은 <br>
전부 pyproject에 기입된다. <br>
마치 npm이나 yarn과 비슷하다. <br><br>

> # 포트리 실행하기

<br>
이제 포트리로 만든 가상환경을 실행해보자.<br>
그러기 위해서는 우선, 가상환경에 진입해야한다. <br>

```
poetry shell
```

가상환경에 진입하였다면,<br>
설치한 것들을 실행해보자 <br>

우리는 django를 설치하였으니 아래처럼 입력해보자 <br>

```
django-admin
```

이러면 터미널에 사용 가능한 명령어를 보여줄 것이다. <br>
여기서 project를 시작해보자. <br>

```
django-admin startproject config .
```

config는 폴더명이고, 닷은 현재 위치를 지칭한다. <br>
그러면 현재 프로젝트에 config라는 폴더와 <br>
manage.py라는 파일이 생성될 것이다. <br><br>

> # 포트리 종료하기

<br>
가상환경을 실행하고 사용을 다 하였다면, <br>
가상환경을 종료도 해줘야한다. <br>

```
exit
```

심플하지만, 이러면 가상환경을 탈출함으로써 <br>
사용을 끝마칠 수 있다. <br><br>
