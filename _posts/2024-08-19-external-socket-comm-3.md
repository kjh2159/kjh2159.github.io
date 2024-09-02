---
title: (python) 외부 IP 사이의 소켓 통신 GET/POST 요청 3편
date: 2024-08-19 11:00:00 +0900
categories: [python, socket communication]
tags: [socket, socket communication, python, ip]
---
# (python) 외부 IP 사이의 소켓 통신 GET/POST 요청 3편

## 들어가며

### 이전 내용

[2편 포스트](../external-socket-comm-2/)에서 다룬 내용은 방화벽과 관련해 인/아웃바운드의 개념을 알아보고 인/아웃바운드 규칙을 설정했다. 만약 이 부분이 설정되어 있지 않다면, [2편 포스트](../external-socket-comm-2/)를 통해 먼저 설정하고 오도록하자.

## 서버 열기

### 서버 열기란?
서로 다른 두 기기가 네트워크를 통해 통신하기 위해서는 내부망과 외부망의 여부와 상관없이 이는 한 쪽이 서버가 될 수 있도록 IP접속을 허용하도록 본인 IP와 기기에서 소켓을 열어두어야 한다. 간단한 비유를 들자면, 콘센트(socket)에 플러그를 꼽기위해서는 콘센트 보호 캡이 없어야 낄 수 있는데, 이 보호캡을 열어둘 수 있도록 해야한다. 이러한 프로세스를 실행시키며 플러그 연결을 받을 준비를 하는 기기를 ***서버***라고 칭할 수 있다. 일반적인 기기에서 이를 막는 이유는 보안상의 이유가 굉장히 크다. 
> 물론, 리눅스 간의 내부망 통신에서는 서버를 열지 않아도 파일을 주고받을 수 있긴 하다. 속도는 느리지만 말이다.
{: .prompt-info }

서버를 여는 과정은 크게 2가지 단계로 나눠진다. 첫 번째는 ***로컬 서버 열기***, 두 번째는 ***인터넷 접근 허용***이다. 먼저, 로컬 서버를 열기 위해서 서버 운용을 위한 코드가 필요하다. 드디어 코딩을 하게 됐다. 

### 서버 파이썬 코드

파이썬 코드는 로컬 서버를 위한 파일 하나, 외부 클라이언트가 서버에 접근하기 위한 파일 하나가 필요하다. 

***Server Side: `server.py`*** 
```python
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/post', methods=['POST'])
def handle_post():
    data = request.json
    print(f"Received data: {data}")
    return jsonify({"status": "success", "data": data}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000) # Allow all ip accesses
```

***Client Side: `client.py`*** 
```python
import requests

url = 'https://YOUR_EXTERNAL_IP:5000/post' # or link
data = {'key': 'value'}

response = requests.post(url, json=data)
print(response.json())
```
이렇게 되면 기본적인 준비는 끝났다. 물론, 파이썬 기본 모듈인 socket을 사용할 수야 있겠지만, Flask가 굉장히 가독성도 괜찮고 utility가 좋아서 Flask와 requests를 사용했다.

## Ngrok 설정
서버를 외부의 도메인으로 찾기위해서는 IP와 도메인을 연결해주는 프로그램 또한 실행시켜주어야 한다. 이는 Ngrok을 통해서 진행할 수 있으며, 나는 테스트용이므로 Ngrok이 기본적으로 제공해주는 Ngrok자체의 도메인을 쓰겠다. 하지만, 잘 찾아보면 도메인 커스텀이 있다고 알기에 본인이 자체적으로 도메인을 갖고 있다면, 해당 도메인을 적용시켜도 괜찮아보인다. 

### Ngrok 시작
***계정 생성***
[Ngrok 사이트](https://ngrok.com)에 먼저 접속해 가입해준다(공공기관에서는 접속 안 될 수 있는데 안 된다면, VPN이나 IP 우회 프로그램 사용).  

***인증 토큰 받기***
로그인 후, 대시보드(Dashboard)로 이동합니다. 로그인시 저는 MS Authenticator와 연동해 보안 수준을 높였다.  메뉴 > Your Authtoken으로 들어가 인증 토큰을 발급받는다.

***ngrok 다운로드***
Ngrok 사이트에서 ngrok 실행 프로그램을 다운로드 받고 압축을 풀어준다. 그리고 터미널을 열어 해당 디렉토리로 이동한다.
```bash
cd ngrok
```

***ngrok 터널 시작***
대시보드에서 AuthToken을 복사해 아래의 명령어의 `YOUR_AUTH_TOKEN`에 대치합니다.
```bash
ngrok authtoken YOUR_AUTH_TOKEN
```
그리고 성공적으로 인증이 됐다면, 파이썬 서버 코드와 함께 실행해준다.
```bash
python server.py # or python3
```
```bash
ngrok http 5000
```
이 명령어는 5000번 포트에서 ngrok 터미널 시작한다는 의미이다.


## 통신
### URL 대치
이제 마지막이다. ngrok을 실행시키면 다음과 같은 domain (url)이 뜰 것이다.
```
https://abc1-111-222-33-123.ngrok-free.app
```
이 링크를 `client.py`의 url에 대치시켜준다.
```python
import requests

#url = 'https://YOUR_EXTERNAL_IP:5000/post' # or link
url = 'https://abc1-111-222-33-123.ngrok-free.app'
data = {'key': 'value'}

response = requests.post(url, json=data)
print(response.json())
```
이렇게하면, 도메인 주소를 통해서도 접근이 가능하다(도메인 연결은 또 다른 문제..). 

### 클라이언트 실행
이렇게 했다면, 클라이언트 단에서 클라이언트의 코드를 실행하면 된다. 
```bash
python client.py # or python 3
```
연결이 되었다면, json 파일을 주고 받는다.


## 마무리
솔직히 말해서 NAT 사용하는 공유기를 통과하는 경우에 포트포워딩과 NAT 설정을 잘 만져주면 된다고 하는데 외부 IP간의 통신은 아직 못 만들고 테스트도 못 했다. 그런데 앞으로 많이 바빠질 것 같아서, 이 부분은 못 만지기에 잠시 넣어두려고 한다. 추후 성공하면 공유기 cover 버전으로 가져오도록 하겠다. 그럼 이만.