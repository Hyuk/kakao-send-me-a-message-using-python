# 파이썬을 사용해서 카카오톡 메시지 보내는 방법

1. [카카오 디벨로퍼스 웹사이트](https://developers.kakao.com/)로 이동한다.

1. '내 애플리케이션'을 클릭한다.

1. '애플리케이션 추가하기'를 클릭한다.

1. '앱아이콘'(선택)과 '앱 이름'(필수), '사업자명'(필수)을 기입하고 저장을 클릭한다.

1. '앱 설정 > 요약 정보'에서 REST API 키를 가져온다.

1. '제품 설정 > 카카오 로그인'에서 '활성화 설정의 상태'를 ON으로 변경하고, 'Redirect URI'를 임의로 "https://example.com/oauth"로 지정한다.

1. '제품 설정 > 카카오 로그인 > 동의항목'에서 '닉네임'(profile_nickname)를 필수 동의로 변경하고, '카카오톡 메시지 전송'(talk_message)을 선택 동의로 변경한다. 

1. REST API값과 Redirect URI값을 사용하여 다음과 같은 URL을 만들어 브라우저에 붙여넣어 auth_token 값을 가져온다. 

```
https://kauth.kakao.com/oauth/authorize?client_id={YOUR_REST_API}&redirect_uri={YOUR_REDIRECT_URI}&response_type=code
```

9. Python 파일 kakao-send-me-a-message-token.py를 생성하고 다음과 같이 작성한다.

```python
import requests
import json

with open("keys.json","r") as fp: # REST API 키를 keys.json 파일에 넣고 불러와서 사용한다.
  keys = json.load(fp)

url = 'https://kauth.kakao.com/oauth/token'
rest_api_key = keys["restApi"] # REST API 키를 불러와서 rest_api_key 값으로 저장한다.
redirect_uri = [YOUR_REDIRECT_URI] # REDIRCT URI를 입력한다. 'https://example.com/oauth'
authorize_code = [YOUR_AUTH_TOKEN] # auth_token 값을 입력한다.

data = {
  'grant_type': 'authorization_code',
  'client_id': rest_api_key,
  'redirect_uri': redirect_uri,
  'code': authorize_code,
}

# Token 값들을 가져온다.
response = requests.post(url, data=data)
tokens = response.json() 

# "kakao_token" 파일을 만들어 Token 값들을 저장한다.
with open("kakao_token.json","w") as fp:
    json.dump(tokens, fp)
```

10. Python 파일 kakao-send-me-a-message.py를 만들어 다음과 같이 작성한다.
```python
import requests
import json

with open("kakao_token.json","r") as fp: # kakao_token 파일에서 token 정보를 가져온다.
  tokens = json.load(fp)

# 카카오톡 메시지를 나에게 보내기 url은 다음과 같다.
url="https://kapi.kakao.com/v2/api/talk/memo/default/send"

# headers 값을 bearer token 값으로 지정해준다. bearer 다음에 스페이스 한칸이 들어가는 것을 주의해야한다.
headers={
  "Authorization" : "bearer " + tokens["access_token"]
}

# 텍스트를 기본 포맷으로 메시지를 보낼 경우 다음과 같이 작성한다.
data={
  "template_object": json.dumps({
    "object_type":"text",
    "text":"할로우",
    "link":{
        "web_url":"https://www.naver.com",
        "mobile_web_url":"https://www.naver.com"
    },
    "button_title": "바로 확인"
  })
}

# 메시지 템플릿 작성 방법은 아래 페이지 참고한다.
# https://developers.kakao.com/docs/latest/ko/message/message-template
# https://developers.kakao.com/docs/latest/ko/message/rest-api

response = requests.post(url, headers=headers, data=data)
```

### 실행하는 방법 (How to run this program)
* Auth token 값을 받은 후, kakao-send-me-a-message-token.py 파일의 [YOUR_AUTH_TOKEN]값을 수동으로 업데이트 해준다. 
```bash
$ python kakao-send-me-a-message-token.py
$ python kakao-send-me-a-message.py
```

### 주의 사항
* access_token은 12시간 유효하다.
* refresh_token은 30일간 유효하다.

## 이슈 (Issues to be fixed)
* 바로가기 링크 및 링크 이동이 안된다.

### 기능 추가 할 것 (Features to be added)
* auth token 값을 자동으로 받아와서 업데이트 해주기
* refresh_token을 사용해서 access_token 지속적으로 업데이트 해주기
* 주기적으로 주식, 코인, 뉴스 정보 보내주는 기능 업데이트 해주기

