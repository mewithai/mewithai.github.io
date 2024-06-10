---
permalink: /langchain/register_prompt_with_langsmith_and_run_240610/
title: "LangSmith를 활용하여 Prompt 관리"
excerpt: "Prompt를 Hard Coding으로 넣는 것은 이제 그만 하고, 체계적으로 관리하자."
author: "JS Kim"
published: false
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - vscode
tags:
  - vscode
---

## 배경과 목표

Python Code에다가 Prompt를 붙여 놓는 것과 최신 Prompt를 git 처럼 땡겨 오고, History를 관리하는 것 중 어느것이 효과적일까?

장단점이 있을 것 같은데.. 일단, 경험해보자.

### 목표

- LangSmith를 통해서 효율적으로 최신 Prompt와 History를 관리하는 방법을 배운다.

## 본론

### LangSmith에서 Prompt 등록하기

`LangSmith`에는 왼쪽 메뉴 중에 제일 하단에 Prompt를 추가할 수 있는 버튼이 있다.

Prompt의 종류는 다음과 같다.

- 공개 여부 : Private or Public
- Prompt 종류 : Chat-Stype Prompt or Instruct-Style Prompt

![]({{ site.url }}{{ site.baseurl }}/assets/images/images/20240610140239.png)

### Instruct-Style Prompt

아래처럼 하면 된다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/20240610141456.png)

Prompt는 PlayGround로 이동하여 바로 Try 해볼 수 있다.

여러개의 입력을 시험하려면 Dataset을 그렇지 않으면 Manual로 바로 `f-String`으로 바꿔 볼 수 있음

![]({{ site.url }}{{ site.baseurl }}/assets/images/20240610143332.png)

시험 결과에 대해서 하단에 Save Example후 Commit를 하면 시험결과에 대해서도 이력이 남아서 관리됨

![]({{ site.url }}{{ site.baseurl }}/assets/images/20240610143510.png)

### Chat-Stype Prompt

똑같다.

## 결론

- LangSmith에 Prompt를 저장해놓고 시험해보면서 사용하면 나중에 Debug를 하기가 편하다.
- 시험에 사용한 Test Parameter을 Example로 저장해 놓을 수 있어서 굉장히 유용하다.
- 비공개, 공개로 나눠서 쓸 수 있어서 다른 사람과 공유하기에도 편하다.

## 참고

- [LangSmith](https://www.langchain.com/langsmith)
- [Using Langsmith for Prompt Generation](https://www.youtube.com/watch?v=fzYUudv4g70)
- [Evaluations in the prompt playground](https://www.youtube.com/watch?v=IJxI-4YdySE)

### 기타

- Provider에서 OpenAI와 `ChatOpenAI`가 있는데, `GTP4-o`는 `ChatOpenAI`이다
