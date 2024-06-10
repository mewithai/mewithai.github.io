---
permalink: /langchain/About-LangChain_Runnable-240524/
title: "LangChain에서 Runnable은 어떤 것인가?"
excerpt: "LangChain Runnable은 단위 Process나 Task로 이해하면 된다. "
author: "JS Kim"
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

### 목표

- `LangChanin`에서 `Runnable`에 대해서 이해하고 `Runnable Interface`로 개념을 연결하기

## 본론

### LangChain에선 Runnable이란?

 `Runnable`이라고 하면 주로 자바(Java)에서 `멀티프로세싱`을 위한 병렬 처리 작업(Task)을 정의하는 데 사용. 하지만 이 개념을 `LangChain`에서 그대로 적용하려고 하면 혼란스러울 수 있음. 따라서 LangChain에서의 Runnable은 JAVA의 개념을 버리고 새로운 개념으로 이해하는 것이 더 쉽다.
 
 LangChain에서의 Runnable은 고유한 의미를 지니며, 주로 `Language Models`(예: OpenAI의 GPT)와의 상호작용을 관리하고 조정하는 `작업 단위`를 나타냄
 
 따라서 LangChain에서의 Runnable은 `자연여 처리 파이프라인` 내에서 특정 작업(예: 텍스트 생성, 질의 응답 등)을 수행할 수 있는 컴포넌트로  `NLP 모델의 입력과 출력을 관리하는 것`이라고 이해할 것

### 기본형태의 Runnable

 Runnable은 Runnable을 상속받아서 생성가능

 Runnable을 상속받으면 Runnable의 기본능력을 모두 이어 받게 된다. 아래는 invoke를 Override한 Case..

```python
# Runnable 정의
class TextGeneratorRunnable(Runnable):
    def __init__(self, model_name: str):
        self.model = OpenAI(model_name=model_name)
    
    def invoke(self, prompt: str):
        return self.model.generate(prompt)
    
# 객체 생성
text_gen = TextGeneratorRunnable(model_name="gpt-4")

# invoke 메서드 사용 예시
result_invoke = text_gen.invoke("What is the capital of Germany?")
print(result_invoke)  # "The capital of Germany is Berlin."
```
 
 그런데, 이런 단순한 기능은 함수로도 충분히 구현이 가능한데 왜 `Runnable`을 사용을 할까?

### Why Runnable?

 LangChain은 Chain으로 연결된 `Component`들을 통하면서 응답이 만들어 지는데, 각각의 Chain `Component`들은 서로 다른 입출력 구조를 가짐. 그래서 서로 연결하려면 Socket처럼 뭔가 공통의 구조를 가진 애를 만들어야 하는데 그 역활을 하는 애가 `Runnable`이다. 그리고 같은 구조로 서로를 연결하는 것을 `Runnable Interface`라고 한다.

 
| Component    | Input                                              | Output           |
|--------------|----------------------------------------------------|------------------|
| Prompt       | Dictionary                                         | PromptValue      |
| ChatModel    | Single String, List of chat messages or a prompt value | ChatMessage      |
| LLM          | Single String, List of chat messages or a prompt value | String           |
| OutputParser | Output of an LLM or ChatModel                      | Depends on the parser |
| Retriever    | Single string                                      | List of documents|
| Tool         | Single string or dictionary depending on the tool  | Depends on the tool   |

위의 예시처럼 각각의 `Component`들은 서로 입출력형태를 가진다. 얘들을 모두 Runnable형태로 바꿔서 통일시키면 서로서로 그냥 딱딱 붙을 수 있는 구조가 되는 것이다.

### Runnable의 잇점

Runnable 사용하면 각 작업을 `구조화 및 모듈화`시키고 이를 통해 `로깅, 디버깅`을 용이하게 하며 `어려운 로직`을 구현할 수 있게 해주는 등 많은 잇점이 있다고 한다. 하지만 가장 내게 와 닿은 것은 `LCELL(|)`을 사용할 수 있게 해주는 것이였다.

아래의 코드는 `Runnable`을 상속받아 만든 객체가 `Runnable Interface`를 활용해서 `LCELL`로 연결되는 모습이다.

```python
from langchain_core.runnables import Runnable

# Define Runnable classes
class Tokenizer(Runnable):
    def invoke(self, input, config=None):
        return input.split()

class TokenLengthCalculator(Runnable):
    def invoke(self, tokens, config=None):
        return [len(token) for token in tokens]

class TokenLengthConcatenator(Runnable):
    def invoke(self, token_lengths, config=None):
        return ' '.join(map(str, token_lengths))

# Instantiate Runnable objects
tokenizer = Tokenizer()
length_calculator = TokenLengthCalculator()
length_concatenator = TokenLengthConcatenator()

# Chain the Runnable objects
chain = tokenizer | length_calculator | length_concatenator

# Execute the chain
input_text = "LangChain makes building applications with LLMs easy"
output = chain.invoke(input_text)

print(output)  # Output: "9 5 8 11 9 4"
```

 Runnable 클래스를 상속받아 구현하면, 각 단계가 명확하게 분리되고, 체인을 구성하는 데 있어 보다 직관적인 표현을 사용할 있다. 또한, LangChain의 기능과의 통합이 용이해져 복잡한 파이프라인을 구성할 때 유리하다.

### Runnable의 기본 Method

 `Runnable`의 기본 정보는 [langchain_core.runnables.base.Runnable](https://api.python.langchain.com/en/latest/runnables/langchain_core.runnables.base.Runnable.html#)에서 확인할 수 있다.

문서에 나오는 다수의 Mehtod 중 아래의 3*2 가지만 기억하자.

| 메서드       | 설명                                                                  |
|--------------|-----------------------------------------------------------------------|
| `invoke`/`ainvoke` | 단일 입력을 출력으로 변환, `a`는 비동기를 의미 |
| `batch`/`abatch` | 여러 입력을 변환 처리, `a`는 비동기를 의미 | 
| `stream`/`astream` | 단일 입력으로부터 생성된 출력을 스트리밍하는 메서드, `a`는 비동기       |

 모든 Method는 `Config`를 통해서 Tracing이나 Debugging 위한 Tag나 메타데이터를 추가할 있고,  `input_schema`, `output_schema`, `config_schema`를 통해서 디테일하게 설정을 할 수 있다.

### Runnable의 입출력은 Pydantic Type

각각의 Component들을 `Runnable Interface`로 연결한다면 데이터도 이가 딱 맞아야 한다. `Runnable`에서는 입출력 데이터 `input_schema`, `output_schema`를 `Pydantic`을 이용하여 정의함으로써 딱 주어진 형태의 데이터가 입출력되도록 제한한다.

앞서 코드 `Tokenizer`에서 입출력의 형태를 출력해보면 다음과 같다.

```python
from langchain_core.runnables import Runnable

class Tokenizer(Runnable):
    def invoke(self, input, config=None):
        return input.split()

tokenizer = Tokenizer()

>>> tokenizer.get_input_schema()
<class 'pydantic.v1.main.TokenizerInput'

>>> tokenizer.get_input_schema().schema()
{'title': 'TokenizerInput'}

```

`pydantic.v1.main.TokenizerOutput`에서 입출력이 Pydantic임을 알 수 있다.

### Runnable의 입출력 Schema()를 통해서 Json으로 확인

`input_schema`, `output_schema`은 Pydantic인데 이는 scheme()를 통해서 JSON 형태로 확인할 수 있다.

## 결론

- LangChain에서 Runnable은 Chain을 구성 요소의 작업단위를 나타낸다.
- 각각의 Chain은 Runnable로 연결될 수 있으며 그것을 Runnable Interface라고 칭한다.

## 참고

- [LangChain Interface](https://js.langchain.com/v0.1/docs/expression_language/interface/)
- [LangChainn Runnable Interface](https://python.langchain.com/v0.1/docs/expression_language/interface/)
- [LangChain streaming](https://js.langchain.com/v0.1/docs/expression_language/streaming/)
- [LangChain Input/Output Scheme](https://python.langchain.com/v0.1/docs/expression_language/interface/#input-schema)
- [참고](https://jordan-mungujakisa.medium.com/the-langchain-interface-chains-and-runnables-cd2f2cb6b4d6)