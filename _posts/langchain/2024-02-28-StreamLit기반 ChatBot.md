---
toc: true
toc_sticky: true
toc_label: "StreamLit 활용 ChatBot"
categories:
  - langchain
tags:
  - 자동화Bot
  - langchain
---

## 배경과 목표

 StreamLit을 활용해서 ChatBot을 만들어보자.

## 설치 및 기본 정보

- `StreamLit`은 `pip`를 이용해서 설치하면 된다.
- Raspberry Pi Docker위에 BookWorm Container를 만들고 Virtual Environment생성, `pip install streamlit`을 실행하였다.
- 만약 Rasberry Pi OS가 32bit일 경우 `PyArrow`가 설치되지 않았다. 64 Bit 이어야 한다.
- StreamLit은 다음의 명령어 실행한다 `streamlit run ######.py`
- StreamLit은 `8051` Port로 Web 서비스를 연다.

나는 Host OS에 ssh 접속 및 Tunneling을 통해서 StreamLit에 접속하였다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/20240228153308.png)

## Full Code

- OpenAI의 API Key는 `dotenv` Module을 활용하여 전달 하였다. `.env` 파일에 `OPENAI_API_KEY = sk-*******` 형식으로 API를 전달하면 된다.
- 코드는 그냥 StreamLit 홈페이지에서 그대로 복사 붙이기를 하였다. 
  참고: [build-a-chatgpt-like-app](https://docs.streamlit.io/knowledge-base/tutorials/build-conversational-apps#build-a-chatgpt-like-app)

```python
from openai import OpenAI
import streamlit as st
## Load Config
from dotenv import load_dotenv, find_dotenv
load_dotenv(find_dotenv(), override=True)

## Log Setting
import coloredlogs, logging
logger = logging.getLogger(__name__)
coloredlogs.install(level='DEBUG')
logging.basicConfig(
    level=logging.DEBUG, 
    format='%(name)s - %(levelname)s - %(message)s'
)

st.title("ChatGPT Clone 001")

client = OpenAI()

if "openai_model" not in st.session_state:
    st.session_state["openai_model"] = "gpt-3.5-turbo"

if "messages" not in st.session_state:
    st.session_state.messages = []

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

if prompt := st.chat_input("What is up?"):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    with st.chat_message("assistant"):
        stream = client.chat.completions.create(
            model=st.session_state["openai_model"],
            messages=[
                {"role": m["role"], "content": m["content"]}
                for m in st.session_state.messages
            ],
            stream=True,
        )
        response = st.write_stream(stream)

        
    st.session_state.messages.append({"role": "assistant", "content": response})
```

## Code별 설명

Code별로 살펴보자.

`st.title("ChatGPT Clone 001")` 는 상단 타이틀

`client = OpenAI()`는 LLM을 선언하는 것 

```
if "openai_model" not in st.session_state:
    st.session_state["openai_model"] = "gpt-3.5-turbo"

if "messages" not in st.session_state:
    st.session_state.messages = []

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
```

`st.session_state`가 중요하다. 여기 밑에 아래의 애들이 있다.

- st.session_state.openai_model
- st.session_state.messages
- st.session_state.chat_messages

### st.session_state


