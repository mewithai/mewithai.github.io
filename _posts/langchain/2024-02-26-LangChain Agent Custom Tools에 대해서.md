---
toc: true
toc_sticky: true
toc_label: "LangChain Agent Custom Tools에 대해서"
categories:
  - langchain
tags:
  - 자동화Bot
  - AI Agent
---

## 배경과 목표

업무 자동화를 위하여 LangChain의 Agent에게 일을 시킬 꺼예요. 밥은 못사줘도 도구(`Tools`)라도 쥐어줘야죠. Tool이 어떻게 동작하는지 이해합시다.

- 관련문서: LangChain -> Modules -> Agent -> Tools -> Defining Custom TopologicalSorter
- [참고 Site: enhancing-langchain-agents-with-custom-tools](https://www.comet.com/site/blog/enhancing-langchain-agents-with-custom-tools/)

## Tools을 만드는 방법

툴은 다음의 방법으로 만들 수 있음

1. `Tool.from_function()` 를 사용
1. `@Tool` Decorator를 사용
3. `BaseTool class`를 상속

Agent가 사용할 툴은 다음의 것들을 정의해두어야함.

- **name (str)** is required and must be unique within a set of tools provided to an agent
- **description (str)** is optional but recommended, as an agent uses it to determine tool use
- **return_direct (bool)**, defaults to False, 나온 것을 바로 답으로 쓸지 말지 결정
  - 일반적으로 LLM이 해석을 해서 답을 주는데 이걸 `True`로 하면 바로 답으로 사용됨
- **args_schema (Pydantic BaseModel)**, is optional but recommended and can be used to provide more information (e.g., few-shot examples) or validation for expected parameters.

주) `Pydantic BaseModel`은 잘 모르는 개념 : [자료 작성 필요]

위에 설명한 방법으로 Tool을 만들어 보자.

### Using the Tool dataclass

- 인터넷에서 찾아서 결과를 주는 Tool을 만든다면
  - accept a single string input and return a string output

```python
duck_search = DuckDuckGoSearchRun()

search_tool = Tool.from_function(
    func=duck_search.run,
    name="Search",
    description="useful for when you need to search the internet for information"
)
```

- Tool의 이름은 `Search`
- Tool의 역활은 "useful for when you need to search the internet for information"

- `Tool.from_fuction`은 Tool을 생성하는 함수
  - 이 툴은 `duck_search` 인스턴스의 `run`이라는 함수를 실행
  - `Input`과 `Output`에 대한 설명이 생략되어 있음

```log
> Entering new AgentExecutor chain...
I need to search for the Sacramento Kings win-loss record for the 2022-2023 season. Then I need to add the games won and games lost and multiply that result by 2.
Action: Search
Action Input: "Sacramento Kings win-loss record 2022-2023 season"
Observation: 어쩌고 저쩌고
```

 Tool을 만들때 기술하진 않았지만 입력으로 `String`을 출력으로 `String`을 받아온다.
 돌아온 `String`을 가지고 다음번 `Chain`의 입력으로 사용

**눈여겨 볼 것**: `args_schema`가 그냥 입력은 String, 출력도 String으로 하는 듯

### `@tool` Decorator

- 위에 것 보다 조금 더 고수
- `@tool` 안에다가 원하는 파라미터를 넣을 수 있음
- 다음 줄의 함수를 Tool로 만들어준다, Class의 Method정의로도 활용 가능

**잠깐!**: Python Decorator에 대해서도 다시 정리할 필요 있음


```python
from langchain.tools import tool

@tool("lower_case", return_direct=True)
def to_lower_case(input:str) -> str:
  """Returns the input as all lower case."""
  return input.lower()
```

`args_schema`에 대해서 `Description` 안에다가 써도 된다.

**잠깐!** `args_schema`를 따로 선언하는 것과 `Description` 안에 쓰는 것 차이가 무엇인지 확인 필요

아래는 IoT DB에서 Inspection이 필요한 Devices를 조회해서 가지고 오는 Tool을 샘플로 만들어 보았다.

```python
class IoTTools():

    @tool("get Devices which are not working")
    def get_Connected_Devices(server : str ) -> list:
        """Useful to get Devices which is not working for a certain time.
           These devices are needed to be checked and fixed.

           :input value: server, Blank is good by default
           :return value: list of tuple, (device name, ipaddress, last found date and time)
           """
        try:
            db = SlDbEngine()
            result = db.get_1H_Not_Working_Device()

            return f"Not working devices on are {result}."
        except Exception:
            return "Error with the input for the tool."
```

- 실행결과는 LangSmith로 분석해보면 된다. [LangSmith분석]

### Subclassing from the BaseTool class

BaseTool class를 상속하면 좀 더 아름다운 툴이 나온다. Callback 함수도 넣을 수 있음
**이건 나중에 추가**

### Modifying existing tools

기존의 툴을 수정하려면 내부 member 변수로 접근해서 수정하면 된다. 

```python
WikipediaQueryRun(name='Wikipedia', description='A wrapper around Wikipedia. Useful for when you need to answer general questions about people, places, companies, facts, historical events, or other subjects. Input should be a search query.', args_schema=None, return_direct=False, verbose=False, callbacks=None, callback_manager=None, tags=None, metadata=None, handle_tool_error=False, api_wrapper=WikipediaAPIWrapper(wiki_client=<module 'wikipedia' from '/usr/local/lib/python3.10/dist-packages/wikipedia/__init__.py'>, top_k_results=3, lang='en', load_all_available_meta=False, doc_content_chars_max=4000))
```

- name, description, return_direct, verbose, callbacks, tags, metadata, handle_tool_error. api_wrapper, top_k_result, lang, load_all_avaiable_meta, doc_conetent_chars_max 같은 애들이 있다.
- 이중에서 `description`를 수정해서 쓰는 것이 대부분일 듯

## 결론과 Next Step

LangChain에서 Agent가 사용하는 `Tools`에 대해서 알아 보고 개념을 이해하였다.

### Next Step

- LangSmith로 LangChain이 들아가는 과정을 분석해보기
- Python Decorator 정리해두기
- Subclassing from the BaseTool class 정리 하기
- 비동기방식의 Tool의 구동 과정 이해하기