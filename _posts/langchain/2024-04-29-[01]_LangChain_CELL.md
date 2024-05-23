---
permalink: /langchain/About-LangChain-LCELL-240429/
title: "LangChain의 LCELL 쉽게 이해하기"
excerpt: "LangChain의 핵심 개념인 LCELL을 쉽게 이해할 수 있도록 정리합니다."
author: "Jinsu Kim"
published: true
toc: true
toc_sticky: true
toc_label: "목차"
categories:
  - langchain
tags:
  - 자동화Bot
  - AI Agent
---

## 배경과 목표

LangChain은 복잡한 개념을 포함하고 있어 처음 접하는 사람들에게 어려울 수 있는데, 이 글에서는 LangChain의 핵심 개념인 LCELL을 쉽게 설명하여, 단순하고 명확하게 이해할 수 있도록 돕고자함.

### 목표

- 간단하고 명확하게 정리하여, 나중에 다시 읽어봐도 쉽게 이해할 수 있도록 하기
- LangChain을 보면 Chain이 생각나고 각각의 고리들이 LCELL(`|`)로 연결되었다는 이미지가 머리에 떠오를 것

## LangChain and LCELL

### LangChain이란?

 LangChain은 대규모 언어 모델(LLM)을 사용하여 질문에 답하는 과정을 단계적으로 연결하는 프레임워크. 각 단계는 연쇄적으로 연결되어 `Chain`을 형성.

LCELL은 Chain의 구성 요소를 나타내며, 각 단계는 `|` 기호로 구분됨.

![]({{ site.url }}{{ site.baseurl }}/assets/images/20240429142245.png)

### LangChain의 장단점

#### 장점

- **편리함**: 다양한 작업을 자동화하여 효율성을 높임
- **확장성**: 여러 LLM과 쉽게 통합할 수 있음

#### 단점

- **학습 곡선**: 처음 접하는 사람들에게는 새로운 개념이 많아 어려울 수 있음
- **복잡성**: 설정과 구성 요소가 많아 초기 설정이 복잡할 수 있음

그외 장단점은 사용해 보면서 느끼자.

### LCELL을 쓴 LangChain 예시

다음은 LCELL을 사용하여 LangChain을 구성하는 예시

- `chain = prompt | llm | output_parser`
  - `Prompt -> LLM -> OutPutParser`이 `Chain`으로 연결되었음

```python
# 환경 변수 로드
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv(), override=True)

# LangChain관련 라이브러리 로드
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

# 객체 생성, ChatModel 사용
llm = ChatOpenAI(model = "gpt-3.5-turbo")
output_parser = StrOutputParser()

# 입력 템플릿 정의, 파라미터 없음
template = """You are a helpful assistant. Hello?"""
prompt = PromptTemplate.from_template(template)

# LCELL을 사용한 Chain 정의
chain = prompt | llm | output_parser

# Chain 실행 및 결과 출력
print(chain.invoke({}))

```

## 정리

위 예시에서 `prompt | llm | output_parser`은 LCELL로 연결된 구조이고, 이것이 `Chain` 처럼 보여야 LangChain의 기본을 이해하였다고 볼 수 있음.

## 참고

- [LangChain 공식 문서](https://langchain.com/docs)
- [LCELL 사용 예시](https://langchain.com/examples)