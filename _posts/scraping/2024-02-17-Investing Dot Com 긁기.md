---
excerpt: "Investing.com을 긁기 위한 방법 생각해보기"
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

## `Investing.com`은 만만하지 않다.

  2022년 12월 부터 `Cloudflare`가 지키고 있다. [참고-invespy가 안되요!](https://github.com/alvarobartt/investiny/issues/73)

  만약 Request에 Response로 `403 Forbidden`이 돌아온다면 그건 걸린거다.
  
  오늘은 이 `Cloudflare`에서 효과적으로 벗어날 방법을 고민해보자.

### 걸린 건지 확인

  아래의 코드로 시험을 해보면

```python
import requests

url = 'https://www.investing.com/currencies/usd-krw-news'

headers = {
  'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'
}
response = requests.get(url, headers=headers)
print(response.status_code)
```

 정상은 `200`을 차단된 애는 `403`을 Response한다. 같은 Code이지만 다른 결과이니 Black으로 처리된 IP는 그냥 검증없이 차단하는 듯 하다.

### 어떻게 Bot이라고 아는가?

Cloudflare는 아래와 같은 방법으로 Bot을 인지한다고 한다.

- IP 데이터베이스: 악성 또는 의심스러운 활동과 관련된 IP 주소를 식별하고 차단
- 사용자 행동 분석: 마우스 움직임, 키보드 입력, 스크롤 등 매번 같은 패턴을 하는지 확인
- Human 인증 페이지: 의심스러운 트래픽을 감지했을 때, CAPTCHA 같은 챌린지를 제시
- 속도 제한: 짧은 시간 내에 지나치게 많은 요청을 보내는 IP 주소를 자동으로 감지, 제한
- HTTP 헤더 검사: HTTP를 통해 확인, Ex) User-agent
- 업계끼리 공유 : Cloudflare는 전 세계의 다른 네트워크와 위협 정보를 공유
- 머신 러닝: 머신 러닝 모델을 훈련시켜 봇 트래픽을 식별
- 브라우저 검증: JavaScript 챌린지 같은 것으로 브라우저가 실제 브라우저인지를 검증

나는 주로 `reqeusts`을 주로 쓰는데 초기 접속시 `javascript`를 챌린지를 실패하고 이후 Blocked IP로 관리되어 403 Response로 돌아온 듯 하다.

## 어떻게 풀지?

단순하게는 그냥 IP를 바꾸면 얼마간 접속이 가능하다. 하지만 동일 패턴은 또 차단당할 수 있다.

가장 현실적인 방법은 `Chrome Driver`와 `Selenium`의 조합 이였다. 그냥 사람이 하는 브라우징과 같은 방식으로 하는 것. 하지만 Chrome을 Full로 돌리기엔 내 HW 인프라가 부족하기에 속도에서 불리하였고 `Chrome` 구동 중 Loading이 늦거나 `Selemium`에서 예외가 발생하면 여지없이 수집이 실패하고 만다.

 결국 내가 원하는 건 `속도`와 `안정성` 그리고 저렴이 Infra에서도 돌아가는 `용이성`이다.

 더 나은 방법이 있는지 알아볼 때이다.

### requests vs Chrome+Selenium

아래의 기능 비교 처럼 reqeusts는 속도가 빠르고 쉽게 구현이 가능하다. 하지만 `Investing.com`은 이게 안먹힌다.

| 기능/특징 | `requests` 모듈 | `Chrome Driver + Selenium` |
|-----------|----------------|-----------------------------|
| 설치 용이성 | 매우 쉬움. 파이썬 패키지 관리자(pip)를 통해 간단하게 설치 가능 | 복잡함. Selenium과 Chrome Driver 설치 필요. 브라우저 호환성 확인 필요 |
| 사용 용이성 | 간단한 HTTP 요청에 최적화. 코드가 간결하고 이해하기 쉬움 | 복잡한 웹 애플리케이션과 상호작용하기 위해 설계됨. 초기 설정이 복잡할 수 있으나, 동적 페이지 처리에 유리 |
| 실행 속도 | 빠름. 실제 브라우저를 실행하지 않기 때문에 리소스 사용이 적음 | 느림. 실제 브라우저를 실행하므로 시작 시간과 리소스 사용이 더 많음 |
| 동적 콘텐츠 처리 | 제한적. JavaScript로 렌더링된 콘텐츠를 직접 처리할 수 없음 | 우수함. 실제 브라우저를 사용하기 때문에 JavaScript 동적 콘텐츠 처리 가능 |
| 헤드리스 모드 지원 | 해당 없음 | 지원함. GUI 없이 백그라운드에서 브라우저를 실행할 수 있음 |
| CAPTCHA 우회 | 어려움. CAPTCHA를 자동으로 해결할 수 있는 기능이 없음 | 가능성 있음. 사용자 상호작용을 모방하는 방식으로 CAPTCHA 우회 시도 가능 |
| 클라우드플레어 같은 보안 조치 우회 | 어려움. 추가적인 처리 없이는 보안 조치에 막힐 가능성 높음 | 가능함. 실제 사용자와 유사한 웹 브라우징 행위를 모방하여 보안 조치 우회 가능 |
| API 사용 | 간단한 REST API 호출에 최적화 | 복잡한 웹 애플리케이션의 API를 통한 상호작용도 가능하지만, 주로 웹 UI를 통한 상호작용에 초점 |

## 새로운 방법의 연구

### 남들도 같은 고민을 한다

 [How-to-bypass-cloudflare](https://scrapeops.io/web-scraping-playbook/how-to-bypass-cloudflare/)

 위 문서에선 총 6가지 방법을 제시한다. 그 4번 `Scrape With Fortified Headless Browsers`은 현재 쓰고 있는 방법이고, 3번 `Cloudflare Solvers`를 알아보고 지금의 방식과 비교 해보려 한다.

### Insight를 얻자

 현재 가장 쓸만하다는 [FlareSolverr](https://github.com/FlareSolverr/FlareSolverr)의 설명에서 보면 처음 Chrome을 이용해서 WEB을 열어 `Cloudflare`를 통과한 후 거기에서 생성된 Cookie를 활용해서 다음에는 바로 쓸수 있게 해준다고 한다.

 > FlareSolverr starts a proxy server, and it waits for user requests in an idle state using few resources. When some request arrives, it uses Selenium with the undetected-chromedriver to create a web browser (Chrome). It opens the URL with user parameters and waits until the Cloudflare challenge is solved (or timeout). The HTML code and the cookies are sent back to the user, and those cookies can be used to bypass Cloudflare using other HTTP clients.

라즈베리파이에 올려서 써보는 건 어떨까? 괜찮다면 Main Server로 Porting 해서 지금의 부하를 줄여주는 것이 좋을 것이다. 

## Next Step

### Action Item

- Mission#1: `FlareSolverr`를 라즈베리파이에 올려보자. [완료:flaresolverr on RaspberryPi]({{ site.url }}{{ site.baseurl }}/scraping/flaresolverr-on-RaspberryPi/)
  - Investing.com 전용으로 쓸 수도 있을 것이다.
- Mission#2: `#1`이 된다면 지금의 CASE와 비교해서 Time 측면에서 잇점이 있는지 분석하자
  - Chrome 구동시간을 줄이기 위해서 Session을 재활용하는 방법을 연구하자
- Mission#3: 종목의 News Scraping에 활용하자.

## 참고 문헌

- [How To Solve 403 Forbidden](https://scrapeops.io/web-scraping-playbook/403-forbidden-error-web-scraping/)
- [how-to-bypass-cloudflare in 2024](https://scrapeops.io/web-scraping-playbook/how-to-bypass-cloudflare/)
- [How to Bypass Cloudflare in 2024: The 8 Best Methods](https://www.zenrows.com/blog/bypass-cloudflare)