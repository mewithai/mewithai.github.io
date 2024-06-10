---
permalink: /blog/About-LangChain_Output_Parser-240528/
title: "d"
excerpt: "ddd"
author: "ddd"
published: false
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

LLM 모델을 사용해 보면서 예상치 못한 답변에 어려움을 겪은 적이 있으신가요?

LLM은 질문에 대한 답변을 제공하지만, 때때로 불필요한 정보가 포함될 수 있습니다. 저도 LLM에게 다양한 질문을 던져보고 답변을 받고, 이를 자동화해 보았지만, 응답 형식이 매번 조금씩 달라져 프로그램이 원활히 작동하지 않는 경우가 많았습니다. **원하는 형태로 질문을 전달하고 필요한 답만 받는 것이 생각보다 쉽지 않았습니다.**

이번 글에서는 OutputParser를 활용하여 LLM에게 효과적으로 LLM의 응답에서 필요한 정보를 추출하는 방법을 다루겠습니다. 이를 통해 작업의 효율성을 높이고 보다 안정적인 결과를 얻을 수 있을 것입니다.

### 목표

- OutPutParser의 필요성에 공감하기
- LangChain에서 OutPut Parser 잘 사용하기

## OutputParser

### format_instructions

LangChain없이 그냥 Prompt를 사용하여 어떤 형태의 정형화된 답을 얻고자 한다면, Prompt 밑에다가 Sample을 들면서 내용을 적으면 된다. 아래 처럼 말이다.

```
List five ice cream flavors.

Your response should be a list of comma separated values, eg: `foo, bar, baz`
```

아래에서 어떻게 답을 하라고 지시하는 것이 `format_instructions`이다. 근데, LangChain은 이것을 Platform에서 쉽게 사용하게 해준다. 단순하게 Prompt에 붙이는 것 이상으로 Pydantic을 사용하여 구조체를 형성한다.

```python
from langchain_core.output_parsers import JsonOutputParser
from langchain_core.pydantic_v1 import BaseModel, Field
from typing import List

# 자료구조 정의 (pydantic)
class CusineRecipe(BaseModel):
    name: str = Field(description="name of a cusine")
    recipe: str = Field(description="recipe to cook the cusine")

class CusineRecipeList(BaseModel):
    recipes: List[CusineRecipe] = Field(description="List of cusine recipes")

# Nested Pydantic도 가능
jsonListParser = JsonOutputParser(pydantic_object=CusineRecipeList)
format_instructions = jsonParser.get_format_instructions()
print(format_instructions)
```

위의 코드를 실행시켜보고 `대박!`이라고 외쳤다. Pydantic으로 OutPut 형태를 지정하면 LangChain이 Instruction을 쓰고 그대로 만들어 준다는데 더이상 무엇이 필요할까? 그리고 내가 직접 Prompt에 지정한 것 보다 훨씬더 잘 동작한다.

실제로 위에서 작성된 `format_instructions`을 출력해본다.

- Json Parser format instructions

```
The output should be formatted as a JSON instance that conforms to the JSON schema below.

As an example, for the schema {"properties": {"foo": {"title": "Foo", "description": "a list of strings", "type": "array", "items": {"type": "string"}}}, "required": ["foo"]}
the object {"foo": ["bar", "baz"]} is a well-formatted instance of the schema. The object {"properties": {"foo": ["bar", "baz"]}} is not well-formatted.

Here is the output schema:
`
{"properties": {"name": {"title": "Name", "description": "name of a cusine", "type": "string"}, "recipe": {"title": "Recipe", "description": "recipe to cook the cusine", "type": "string"}}, "required": ["name", "recipe"]}
`
The output should be formatted as a JSON instance that conforms to the JSON schema below.

```
# LCELL을 사용한 Chain 정의
chain = prompt | llm | jsonListParser
```

### Template를 통해서 OutPut 형태를 지정

그렇다.. 비록 OutputParser를 통해서 얻었지만, 이것은 그대로 PromptTemplate를 통해서 Prompt에도 넣을 수 있다. 

```python
instruction = "You're ~~~"
prompt = PromptTemplate.from_template(instruction, format_instructions=format_instructions)
```

Prompt의 Parameter

Format Instructin은 Prompt를 만들때 부터 지정할 수 있다. 아래는 기본 PromptTemplate에 지정하는 예시이다.





### LLM으로 질의

```python
chain = prompt | llm | json_parser
```

LLM으로 

### Output Parser의 작동 방식

Output Parser는 다음과 같은 단계로 작동합니다:

출력 텍스트 받기: 먼저 NLP 모델에서 생성된 텍스트 출력을 받습니다.
규칙 적용: 미리 정의된 규칙이나 패턴을 적용하여 텍스트에서 필요한 정보를 추출합니다.
구조화된 데이터 생성: 추출된 정보를 JSON이나 CSV 같은 구조화된 형식으로 변환합니다.
결과 반환: 최종적으로 구조화된 데이터를 반환하여 다른 시스템이나 프로세스에서 쉽게 사용할 수 있도록 합니다.

## 결론

## 참고

https://python.langchain.com/v0.1/docs/modules/model_io/output_parsers/
https://wikidocs.net/231384
https://medium.com/@larry_nguyen/langchain-101-lesson-3-output-parser-406591b094d7