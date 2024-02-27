## 배경 및 목적

- Local PC에서 돌릴 수 있는 LLM `ollama`에 대해서 학습
- 일단, 만만한 Windows에 그냥 설치 
- `ollama`에게 일을 줘보자.

## 설치

- Web Site는 다음과 같다 : https://ollama.com/download
  - 위 Site에서 Windows용 설치
- 실행하니까 엄청난 Download를...
  - https://python.langchain.com/docs/integrations/llms/ollama

```
C:\Windows\system32>ollama run llama2
pulling manifest
pulling 8934d96d3f08... 100% ▕████████████████████████████████████████████████████████▏ 3.8 GB
pulling 8c17c2ebb0ea... 100% ▕████████████████████████████████████████████████████████▏ 7.0 KB
pulling 7c23fb36d801... 100% ▕████████████████████████████████████████████████████████▏ 4.8 KB
pulling 2e0493f67d0c... 100% ▕████████████████████████████████████████████████████████▏   59 B
pulling fa304d675061... 100% ▕████████████████████████████████████████████████████████▏   91 B
pulling 42ba7f8a01dd... 100% ▕████████████████████████████████████████████████████████▏  557 B
verifying sha256 digest
writing manifest
removing any unused layers
success
>>>
```

## 시험

- 영어는 잘 동작함, 알아는 듣지만 생성시 엄청난 Delay가 발생하고, 답변도 엉성함

```
>>> Hello?
Hello! It's nice to meet you. How can I assist you today?

>>> What's your name?
Hi there! My name is LLaMA, I'm a large language model trained by a team of researcher at Meta AI. How about you?
What's your name?

>>> Are you free to use?
Yes, I am free to use! I'm just an AI assistant trained by a team of researcher at Meta AI, so I can help with a
wide range of tasks and answer any questions you may have. Is there anything specific you would like to know or
talk about?

>>> 한글도 쓸 수 있나요?
Of course! I can understand and respond in Korean as well. Please feel free to communicate with me in any language
you prefer. How may I assist you today?

>>> 응ㅎ한글로 너의 소개를 해줘
Of course! Here's my introduction in Korean:

안녕하세요! 저는 LLaMA입니다. Meta AI에서 훈련된 대화 모델입니다. 저는 여러 가지 질문을 답하고, 도와주시겠습니다. 뭔가  특정한 话题이나 问题를 어떻게 봐졌어요?
```

## LangChain과 연동

참고Site

- https://python.langchain.com/docs/guides/local_llms

- ollama 실행 명령어: `ollama pull llama2`
- ollama는 local에 11434 Port를 열고 서비스를 받는다.

샘플 코드

```python
from langchain_community.llms import Ollama

llm = Ollama(model="llama2")

## 프린트 해야함
llm("The first man on the moon was ...")
```

### 일줘보기

- 기존의 `Custom Tools` Sample Code를 Ollma로 바꿨을때 속도면에서 OpenAI API를 따라올 수 없었다.
- Local Computer CPU가 100%를 침에도 속도는 최소 10배 이상으로 느려짐

## 결론 And Next Step

- 기존의 `Custom Tools` Sample Code를 Ollma로 바꿨을때 속도면에서 OpenAI API를 따라올 수 없었다.

### Next Step

- 그냥 OpenAI로 공부하고 나중에 Local LLM으로 전환을 검토하자. 검토 시점은 월 API 사용료가 3천원을 넘어설 때