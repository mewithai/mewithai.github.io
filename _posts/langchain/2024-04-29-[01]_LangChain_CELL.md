---
permalink: /langchain/About-LangChain-CELL/
title: "LangChain CELL에 대해서 쉽게 정리"
excerpt: "정확하게 개념을 잡아 놓자"
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

LangChain에서 LCELL가 핵심인데, 잘 이해가 안된듯하다. 쉽게 정리해둔다.

### 목표물

- Keep it Simple. Stupid!
  - 다음에 쓱 읽어보면 이해될만큼 단순하고 명확하게 정리

## LangChain and LCELL

- Language Chain?
  - LLM에 질문을 답을 내는 과정이 단계, 단계로 연결되어 있는 것: `Chain` 처럼
- LCELL?
  - Chain의 구성요소를 `|`의 기호를 사용하여 나타낸 것 

![]({{ site.url }}{{ site.baseurl }}/assets/images/images/20240429142245.png)

### LangChain의 장단점

- 장점: 편하고 좋으니까 쓰겠지.
- 단점: 처음보는 개념이라 어렵다!

그외 장단점은 사용해 보면서 느끼자. 

### LCELL을 쓴 LangChain 예시

```python
## Load Config
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv(), override=True)

## Load OpenAI
from langchain_openai import ChatOpenAI
from langchain_core.prompts import PromptTemplate
from langchain_core.output_parsers import StrOutputParser

## 객체 생성
llm = ChatOpenAI(model = "gpt-3.5-turbo")
output_parser = StrOutputParser()

## 입력
template = """You are a helpful assistant. Hello?"""
prompt = PromptTemplate.from_template(template)

## Chain 정의, LCELL 사용
chain = prompt | llm | output_parser

# Chain 실행 및 결과출력
print(chain.invoke({}))
```

- Chain을 정의하고 Invoke 시켰음. Invoke안에는 `Dict`형태의 입력을 줘야하는데 위 경우에는 없기에 빈 Dict를 넘겨주었다.

## Runnable in LangChain

`Runnable`은 LangChain에서 단위, 단위의 `프로세스`, `TASK`, `Thread`같은 개념이다. 아래의 대표적인 형태는 아래 3가지.

- `stream`: 답이 한번에 딱 안나오고 질질질 나온다.
- `invoke`: 
- `batch`: 여러 개를 동시에

비동기로 실행할 수 있다. [문서참고]()

### 예시

## Runnable Interface

`LCELL`을 통해서 각 단계를 연결 시킬려면 이전 단계의 `OutPut`이 다음단계의 `Input`이 되어야한다. 또한, 이렇게 전달되는 것들이 `일정한 규칙`을 가지고 있어야 한다.

Chain의 각 단계들은 다음의 3가지의 요소를 가지고 다른 단계들과 연결된다.

- input_schema
- output_schema
- config_schema

어떻게 정의되어 있는지 궁금하다면 `#단계#Or#Chain#.input_schema.schema()`를 통해서 출력해볼수 있음

```python
# PromptTemplateOutput의 output_schema를 출력시켜봄
prompt.output_schema.schema() 

{'title': 'PromptTemplateOutput', 'anyOf': [{...}, {...}], 'definitions': {'StringPromptValue': {...}, 'ToolCall': {...}, 'InvalidToolCall': {...}, 'AIMessage': {...}, 'HumanMessage': {...}, 'ChatMessage': {...}, 'SystemMessage': {...}, 'FunctionMessage': {...}, 'ToolMessage': {...}, 'ChatPromptValueConcrete': {...}}}

```

## RunnableParallel and RunnablePassthrough

- `Parallel`이니까 동시에 시킨다는 말
- `Passthrough`이니까 뭔가를 건너뛰고 바로 간다는 말

```python
retriever_a = vectordb.as_retriever()

prompt_str = """Answer the question below using the context:

Context: {context}

Question: {question}

Answer: """

prompt = ChatPromptTemplate.from_template(prompt_str)
inputPara = RunnableParallel(
    {
        "context": retriever_a,
        "question": RunnablePassthrough()
    }
)

chain = inputPara | prompt | model | output_parser
```

![]({{ site.url }}{{ site.baseurl }}/assets/images/images/20240429143646.png)

- 위에서 `inputPara`를 만들어서 입력으로 `Prompt` 생성
- `context`를 얻기위해선 `retriever_a` 실행하는데 `question`은 작업이 필요없음
- 따라서 `question`은 `RunnablePassthrough()`를 통해서 바로 전달
- `invoke("When was Rupesh born?")`를 `invoke({"question":"When was Rupesh born?"})`로 쓰는 것이 좀 더 이해하기 쉽다.

## 참고

- [참고Site#1]()