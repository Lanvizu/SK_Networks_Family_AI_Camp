## API 사용하기

네이버에서 제공하는 api를 사용.

https://developers.naver.com/docs/serviceapi/search/blog/blog.md#python

네이버 개발자 페이지에서 애플리케이션을 신청

제공하는 파이썬 코드에 본인의 클라이언트 아이디와 secret 키를 입력하여 실행

``` python

# 네이버 검색 API 예제 - 블로그 검색
import os
import sys
import urllib.request
client_id = "YOUR_CLIENT_ID"
client_secret = "YOUR_CLIENT_SECRET"
encText = urllib.parse.quote("검색할 단어")
# 블로그 url
url = "https://openapi.naver.com/v1/search/blog?query=" + encText # JSON 결과
# url = "https://openapi.naver.com/v1/search/blog.xml?query=" + encText # XML 결과

request = urllib.request.Request(url)
request.add_header("X-Naver-Client-Id",client_id)
request.add_header("X-Naver-Client-Secret",client_secret)
response = urllib.request.urlopen(request)
rescode = response.getcode()
if(rescode==200):
    response_body = response.read()
    print(response_body.decode('utf-8'))
else:
    print("Error Code:" + rescode)

```

쿼리 스트링 → '?' 뒤에 쿼리 값을 직접 넣어 주는 경우
ex) … news?query = etc



```python
# 블로그 url
# url = "https://openapi.naver.com/v1/search/blog?query=" + encText # JSON 결과
# url = "https://openapi.naver.com/v1/search/blog.xml?query=" + encText # XML 결과

# 쿼리 스트링의 입력 순서가 바뀌어도 정상적으로 작동한다.
# url = "https://openapi.naver.com/v1/search/blog?display=100&query=" + encText # JSON 결과

# 뉴스 url
display = 100
params = f"?query={encText}&display={display}"
url = "https://openapi.naver.com/v1/search/news" + params # JSON 결과
# https://openapi.naver.com/v1/search/news.json

```

쿼리 스트링의 입력 순서가 바뀌어도 정상적으로 작동한다.

----

## 필수 라이브러리

``` python
# json형식의 반환값을 json 객체로 변환
import pandas as pd
import os
import sys
import urllib.request
import json
from bs4 import BeautifulSoup  # pip install bs4
import xml.etree.ElementTree as ET

def convJson(response_body):
    result = response_body.decode('utf-8')
    return json.loads(result)

# XML 데이터를 파싱하여 딕셔너리로 변환
def xml_to_dict(element):
    if len(element) == 0:  # 자식 요소가 없는 경우
        return element.text
    return {child.tag: xml_to_dict(child) for child in element}

from sqlalchemy import create_engine  # pip install sqlalchemy  필요함
# MySQL 데이터베이스 연결 생성
def insertTable(
    df,
    tablename,
    if_exists='append',
    username = 'root',
    password = '1234',
    host = 'localhost',
    database = 'mydatabase'
):  
    '''if_exists: Literal['fail', 'replace', 'append']'''
    
    # SQLAlchemy 엔진 생성
    engine = create_engine(f'mysql+pymysql://{username}:{password}@{host}/{database}')
    
    # DataFrame을 MySQL의 테이블로 삽입 (테이블이 없으면 생성)
    df.to_sql(tablename, con=engine, if_exists='append', index=False)
    
    # 데이터 삽입 후 연결 해제
    engine.dispose()    
```

## API 크롤링

```python
# 네이버 검색 API 예제 - 블로그 검색
import os
import sys
import urllib.request
client_id = "dPvUD5AhQ_AnjwC6EAn4"
client_secret = "ntmEoi5Gih"
encText = urllib.parse.quote("크롤링")
# url = "https://openapi.naver.com/v1/search/blog?query=" + encText # JSON 결과
# url = "https://openapi.naver.com/v1/search/blog.xml?query=" + encText # XML 결과
display = 100
params = f"?query={encText}&display={display}"
url = "https://openapi.naver.com/v1/search/news.json"+params

request = urllib.request.Request(url)
request.add_header("X-Naver-Client-Id",client_id)
request.add_header("X-Naver-Client-Secret",client_secret)
response = urllib.request.urlopen(request)
rescode = response.getcode()
result = None
if(rescode==200):  # 성공시 반환하는 코드
    response_body = response.read()    
else:
    print("Error Code:" + rescode)

convJson(response_body)  # 객체로 사용가능


# {'lastBuildDate': 'Tue, 03 Sep 2024 16:18:34 +0900',
#  'total': 5358,
#  'start': 1,
#  'display': 100,
#  'items': [{'title': '[소부장 인사이트] 디지털 경제 생태계로 변화하는 로봇 산업',
#    'originallink': 'https://www.etnews.com/20240903000077',
#    'link': 'https://n.news.naver.com/mnews/article/030/0003236970?sid=105',
#    'description': '공공 데이터, 정보 웹 <b>크롤링</b> 기술 등을 이용해 수집하거나, 온라인 플랫폼 등으로부터 정보 트래킹 획득, 데이터 웨어하우스 구축을 통한 데이터 분석 및 예측 모델 등을 거래하고 있다. 중국은 44개의 데이터 거래소를... ',
#    'pubDate': 'Tue, 03 Sep 2024 16:01:00 +0900'},
# ...
#   {'title': '2024학년도 수시 합격생 인터뷰 - 성균관대 컴퓨터교육과 장은비(금옥여...',
#    'originallink': 'https://www.naeil.com/news/read/516626?ref=naver',
#    'link': 'https://www.naeil.com/news/read/516626?ref=naver',
#    'description': '전공 적합 활동-인공지능 전문가 체험과 동아리 활동 &lt;은비 학생의 진로 적합 활동&gt; *인공지능 전문가 진로 체험 -자연어 처리, 사람 얼굴 인식 등 인공지능 전문가가 수행하는 업무를 실습 -특히 웹 <b>크롤링</b> 기술을 이용해... ',
#    'pubDate': 'Fri, 12 Jul 2024 17:38:00 +0900'}]}
```

## 공공기관 API

```python
import requests
url = 'http://openapi.tour.go.kr/openapi/service/EdrcntTourismStatsService/getEdrcntTourismStatsList'
skey = 'U5i63xpIWM48raIRQBYpUXbA/wKd0iO6n/n1+JISyoEW1gWcEiz2No2fHPeid5TZoQN0HV85WyGv6LPwaQ8n4w=='
params ={'serviceKey' : skey, 'YM' : '202005', 'NAT_CD' : '112', 'ED_CD' : 'E' }
response = requests.get(url, params=params)

# 바이트 데이터를 문자열로 디코딩
xml_string = response.content.decode('utf-8')
# XML 파싱
root = ET.fromstring(xml_string)
xml_to_dict(root)

# {'header': {'resultCode': '0000', 'resultMsg': 'OK'},
#  'body': {'items': {'item': {'ed': '방한외래관광객',
#     'edCd': 'E',
#     'natCd': '112',
#     'natKorNm': '중  국',
#     'num': '5124',
#     'rnum': '1',
#     'ym': '202005'}},
#   'numOfRows': '10',
#   'pageNo': '1',
#   'totalCount': '1'}}
```

## 정적 웹 크롤링

```python
def makeRow(row):
    return [row[0].text, row[1].text, row[2].text, row[3].text, ','.join([ data.attrs['alt'] for data in row[4].select('img') ]), row[5].text]
```

```python
pagenum = 1
url = f'https://www.hollys.co.kr/store/korea/korStore2.do?pageNo={pagenum}&sido=&gugun=&store='
html = urllib.request.urlopen(url)
soup = BeautifulSoup(html,'html.parser')
tr_lists = soup.select('#contents > div.content > fieldset > fieldset > div.tableType01 > table > tbody > tr')
result = [makeRow(tr.select('td')) for tr in tr_lists]
result
```

![image](https://github.com/user-attachments/assets/87722d07-7c29-400f-b0a9-efde4756df56)


```python
df = pd.DataFrame(result,columns=['지역',	'매장명',	'현황'	,'주소'	,'매장 서비스'	,'전화번호'])
df.head(2)
```

![image](https://github.com/user-attachments/assets/42522914-de06-4c10-9367-14937ec45cc3)


```python
df2 = df.reset_index().rename(columns={'index':'store_id'})
df2.head()
```

![image](https://github.com/user-attachments/assets/404b9c6d-4b51-447a-ab1f-0ed910b32459)


## 데이터베이스 저장

```python
insertTable(df2,'newtable01')
# index 이름이 pk가 되므로 alter 명령어를 이용해서 테이블 변경한다.
# alter table mydatabase.car add constraint car_id primary key(car_id);
```

```python
import pymysql

# MySQL 데이터베이스에 연결
connection = pymysql.connect(
    host='localhost',  # MySQL 서버 주소
    user='root',  # MySQL 사용자 이름
    password='1234',  # MySQL 비밀번호
    database='mydatabase'  # MySQL 데이터베이스 이름
)

try:
    with connection.cursor() as cursor:
        # ALTER TABLE 쿼리 실행: car_id 필드에 PRIMARY KEY 제약 조건 추가
        alter_table_query = """
        ALTER TABLE newtable01
        ADD CONSTRAINT store_id PRIMARY KEY (store_id);
        """
        cursor.execute(alter_table_query)
        connection.commit()  # 변경 사항을 커밋
    print("Primary key added successfully.")
    
except Exception as e:
    print(f"An error occurred: {e}")
    print(e.with_traceback())
    
finally:
    connection.close()  # 연결 해제

# Primary key added successfully.
```

-----

## 크롤링 활용 연습

### 구글 드라이브 마운트를 진행한 후 필요한 라이브러리를 다운

``` python
from google.colab import drive
drive.mount('/content/drive')

!pip install selenium
!apt-get update
!apt install chromium-chromedriver
# !cp /usr/lib/chromium-browser/chromedriver '/content/drive/MyDrive/Colab Notebooks' # (최초 1회)
!pip install chromedriver-autoinstaller
```

### 버전 확인 후 필요한 라이브러니 임포트

```python
# selenium 설치 확인
!python --version

import selenium
print(selenium.__version__)

from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
import sys
from selenium.webdriver.common.keys import Keys
import urllib.request
import os
from urllib.request import urlretrieve

import time
import pandas as pd
import chromedriver_autoinstaller  # setup chrome options
```

### 필요한 데이터 크롤링해오기

![image](https://github.com/user-attachments/assets/6fb89be0-98b9-4412-b59e-16094581ec46)

- https://auto.danawa.com/auto/?Work=record 의 자동차 표


```python

chrome_path = "/content/drive/MyDrive/Colab Notebooks/chromedriver"

sys.path.insert(0,chrome_path)
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--headless') # ensure GUI is off : cloab은 새창을 지원하지않기 때문에 창 없는 모드
chrome_options.add_argument('--no-sandbox')
chrome_options.add_argument('--disable-dev-shm-usage')  # set path to chromedriver as per your configuration
chrome_options.add_argument('lang=ko_KR') # 한국어

chromedriver_autoinstaller.install()  # set the target URL

url = "https://auto.danawa.com/auto/?Work=record"  # set up the webdriver

driver = webdriver.Chrome(options=chrome_options)
driver.get(url)
time.sleep(1)  # 필요한 경우 적절한 시간으로 조정

# 데이터 가져오기
element = driver.find_element(By.CSS_SELECTOR, '#autodanawa_gridC > div.gridMain > article > main > div > div:nth-child(3) > div.left > table')

# 테이블의 각 행을 딕셔너리 형태로 변환
rows = element.find_elements(By.TAG_NAME, 'tr')  # 모든 행 가져오기
data_list = []

for row in rows:
    cols = row.find_elements(By.TAG_NAME, 'td')  # 각 행의 열 가져오기
    if cols:  # 열이 있는 경우에만 
        rank = cols[0].text
        title = cols[1].text  # 여기서 title에는 브랜드 이름이 포함됨
        num = cols[2].text
        share = cols[3].text
        
        data_dict = {
            'rank': rank,
            'title': title,
            'num': num,
            'share': share
        }
        data_list.append(data_dict)

print(data_list)  # 최종적으로 딕셔너리 리스트 출력

# [{'rank': '1', 'title': '현대', 'num': '47,764', 'share': '45.2%'}, {'rank': '2', 'title': '기아', 'num': '40,685', 'share': '38.5%'}, {'rank': '3', 'title': '제네시스', 'num': '10,323', 'share': '9.8%'}, {'rank': '4', 'title': 'KGM', 'num': '3,943', 'share': '3.7%'}, {'rank': '5', 'title': '쉐보레', 'num': '1,587', 'share': '1.5%'}]
```

### pandas를 통해 확인하기

```python
import pandas as pd

df = pd.DataFrame(data_list)
df.head()
```

![image](https://github.com/user-attachments/assets/11999ecf-530e-4014-8c29-94aeb32e0126)


동일하게 크롤링하려는 elements의 정보 위치를 수정하여 해외 브랜드 자동차 정보를 가져오기

'#autodanawa_gridC > div.gridMain > article > main > div > div:nth-child(3) > div.right > table'

![image](https://github.com/user-attachments/assets/462cea26-06a6-408e-8be6-d380a4f90c85)

![image](https://github.com/user-attachments/assets/a8b70954-31dd-46b1-a608-609eb98d760e)

성공적으로 크롤링.
