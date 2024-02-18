---
toc: true
toc_sticky: true
toc_label: "Scraping investing.com"
categories:
  - scraping
tags:
  - Scraping
  - 투자
  - 자동화Bot
---

## 배경과 목표

- 나를 자꾸 차단하는 `investing.com`의 Cloudflare를 우회하자. [이전 글 참고]({{ site.url }}{{ site.baseurl }}/scraping/Investing-Dot-Com-%EA%B8%81%EA%B8%B0/)
- 여러 방법 중 `FlareSolverr`를 라즈베리파이에 설치하고 동작을 시험한다.
  - [FlareSolverr HomePage](https://github.com/FlareSolverr/FlareSolverr)

## 설치

설치 환경은 다음과 같다.

> Raspberry Pi 4, 2G RAM, 64G SDD on USB3.0, Docker with Portainer

### Docker-compose 셋팅

Portainer의 Stack용으로 2개의 Container를 셋팅하였다. 하나는 `FlareSolverr`이고 하나는 시험을 위한 `BookWorm` OS 이다.

Bookworm은 Host OS에 `/home/docker/invesintdotcom`를 만들고 `/home`과 마운트 하였다.

```yaml
version: '3.8'

services:
  flaresolverr:
    image: ghcr.io/flaresolverr/flaresolverr:latest
    container_name: flaresolverr

  investing:
    image: dtcooper/raspberrypi-os:bookworm
    container_name: invesintdotcom
    command: bash -c "while true; do sleep 30; done;" # Keep container running
    volumes:
      - /home/docker/invesintdotcom:/home # Optional: mount a directory from your Raspberry Pi into the container 
```

설치상의 문제는 없었다.

## 시험

`FlareSolverr` Site에서 언급된 Test를  `Bookworm`에서 `curl`을 이용하여 실시하였다. 

### curl 기반

```sh
curl -L -X POST 'http://flaresolverr:8191/v1' \
-H 'Content-Type: application/json' \
--data-raw '{
  "cmd": "request.get",
  "url":"https://www.investing.com/currencies/usd-krw-news",
  "maxTimeout": 120000
}'
```

timeout을 60초로 하면 TimeOut이 난다. 아무래도 `Investing.com`은 초기에 뭔가 많은 것을 로딩하는 듯 하다.

결과로 많은 것을 보여주는데 아직 `HOST OS`의 IP가 Black으로 처리되진 않은 듯 하다.

![]({{ site.url }}{{ site.baseurl }}/assets/images/20240218212754.png)

## Cookie의 재사용

가장 큰 잇점은 한번 생성된 `Cookie`를 계속해서 이용할 수 있다는 것이다. 이를 위해서
`sessions.create`, `sessions.list`, `sessions.destroy`의 Session 관리 명령어를 제공한다. 또한 http get과 post 전송을 위한 
`request.get`과 `request.post`를 제공한다.

자세한 건 공식문서를 참고하자. [공식문서](https://github.com/FlareSolverr/FlareSolverr?tab=readme-ov-file#commands)

아래의 코드는 `httpx`를 활용해서 session을 만들고 request를 전송하고 마지막으로 Session을 지우는 예시이다.

ChatGPT가 만들었다.

```python
import httpx
import json

def create_session(url: str) -> str:
    """
    Create a FlareSolverr session and return the session ID.
    """
    flaresolverr_url = "http://flaresolverr:8191/v1"
    headers = {"Content-Type": "application/json"}
    payload = {
        "cmd": "sessions.create",
        "url": url,
        "maxTimeout": 60000
    }
    response = httpx.post(flaresolverr_url, headers=headers, json=payload)
    session_id = json.loads(response.text)["session"]
    return session_id

def send_request_with_session(url: str, session_id: str):
    """
    Send a GET request using FlareSolverr with an existing session.
    """
    flaresolverr_url = "http://flaresolverr:8191/v1"
    headers = {"Content-Type": "application/json"}
    payload = {
        "cmd": "request.get",
        "session": session_id,
        "url": url,
        "maxTimeout": 60000
    }
    response = httpx.post(flaresolverr_url, headers=headers, json=payload)
    return response.text

def delete_session(session_id: str):
    """
    Destroy a FlareSolverr session.
    """
    flaresolverr_url = "http://flaresolverr:8191/v1"
    headers = {"Content-Type": "application/json"}
    payload = {
        "cmd": "sessions.destroy",
        "session": session_id
    }
    response = httpx.post(flaresolverr_url, headers=headers, json=payload)
    return response.text

# Example usage
if __name__ == "__main__":
    target_url = "https://example.com"  # Replace with the target URL
    session_id = create_session(target_url)
    response = send_request_with_session(target_url, session_id)
    print(response)
    delete_response = delete_session(session_id)
    print(delete_response)
```

### SessionID의 관리

위 코드를 보면 항상 `session_id`가 중요한 것을 알 수 있는데, 얘는 저장하고 관리하는 것이 좋다.

```python
def save_session_id_to_file(session_id: str, file_path: str):
    """Save the session ID to a file."""
    with open(file_path, 'w') as file:
        file.write(session_id)

def read_session_id_from_file(file_path: str) -> str:
    """Read the session ID from a file, if it exists."""
    try:
        with open(file_path, 'r') as file:
            session_id = file.read().strip()
            return session_id
    except FileNotFoundError:
        return None

if __name__ == "__main__":
    target_url = "https://example.com"  # Replace with the target URL
    session_file_path = 'session_id.txt'  # File to store the session ID
    
    # Attempt to read an existing session ID
    session_id = read_session_id_from_file(session_file_path)
    
    if not session_id:
        # No session ID found, create a new session
        session_id = create_session(target_url)
        save_session_id_to_file(session_id, session_file_path)
    
    # Use the session ID for requests
    response = send_request_with_session(target_url, session_id)
    print(response)
    
    # Optional: Delete session if it's no longer needed
    # delete_response = delete_session(session_id)
    # print(delete_response)
    # Note: If you delete the session, consider also removing the session ID file.
```

여러개의 `session_id`를 만들어서 돌려서 쓰거나 동시에 다른 Session으로 공략하는 것도 가능하다.

## 결론 및 Next Step

- 생각보다 아주 쉽게 설치가 가능했다. 그리고 쓸만한 듯 하다.
- Session을 재활용한다는 잇점이 있어 매번 여는 방식보단 시간이 획기적으로 줄어들 듯하다.
- 위의 Function을 이용해서 `Investing.com`의 데이터를 긁어보자.
- `네이버`에서도 활용할 수 있을지 고민
  - 이상 행동을 발견되면 네이버도 IP차단하는데 이 방식으로 우회할 수 있지 않을까?