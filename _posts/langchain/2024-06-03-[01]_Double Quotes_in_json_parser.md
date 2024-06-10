---
permalink: /blog/About-Jekyll-240523/
title: "Jekyll란 무엇인가"
excerpt: "Jekyll에 대해서 쉽게 설명합니다."
author: "Jinsu Kim"
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

### 목표

## 본론


### 문제점

```json
langchain_core.exceptions.OutputParserException: Invalid json output: ```json
[
    {
        "index": 51618,
        "title": "이철우 <b>경북</b>도지사 "경주가 APEC 정상회의 최적지""
    }
]
```

## 결론

## 참고

- https://github.com/langchain-ai/langchain/discussions/19204
- https://community.openai.com/t/any-idea-on-how-to-prevent-double-quotes-inside-of-paragraphs/299915/14

### Output Fixing Parser

To handle JSON parsing issues in LangChain, especially those caused by double quotes, you can use the following strategies:

Output Fixing Parser: LangChain offers an "output-fixing parser" which wraps another output parser. When the initial parser fails, this output-fixing parser can call another language model to correct the formatting errors. For example, if your initial JSON output is not properly formatted, this parser can adjust it to meet the expected schema​ (Introduction | 🦜️🔗 LangChain)​​ (Introduction | 🦜️🔗 LangChain)​.

Custom Output Parser: If the built-in parsers don't meet your needs, you can create a custom output parser. This allows you to define exactly how the output should be formatted and handle any specific edge cases or errors in the JSON output. This involves defining a custom class that inherits from LangChain’s base output parser classes and implementing the parsing logic​ (Introduction | 🦜️🔗 LangChain)​.

Prompt Engineering: Improve the robustness of the JSON output by refining your prompt. Ensure the prompt includes clear instructions for the model to output valid JSON. You can explicitly mention the use of double quotes and proper JSON formatting in your prompt. Additionally, using a Pydantic model in your prompt can help the language model understand the required schema more effectively​ (Introduction | 🦜️🔗 LangChain)​.

Here is an example of how you might set up an output-fixing parser in LangChain:

python
Copy code
from langchain.output_parsers import PydanticOutputParser, OutputFixingParser
from langchain_core.pydantic_v1 import BaseModel, Field
from langchain_openai import ChatOpenAI

class Actor(BaseModel):
    name: str = Field(description="Name of the actor")
    film_names: List[str] = Field(description="List of film names")

actor_query = "Generate the filmography for a random actor."
initial_parser = PydanticOutputParser(pydantic_object=Actor)
fixing_parser = OutputFixingParser(base_parser=initial_parser, llm=ChatOpenAI())

# Example misformatted output
misformatted_output = "{'name': 'Tom Hanks', 'film_names': ['Forrest Gump']}"

# Parsing the output
fixed_output = fixing_parser.parse(misformatted_output)
print(fixed_output)
This example demonstrates how to set up an output-fixing parser that attempts to correct any misformatted JSON by calling another language model to fix the errors.

By using these strategies, you can effectively handle JSON parsing errors caused by improper use of double quotes or other formatting issues in LangChain.