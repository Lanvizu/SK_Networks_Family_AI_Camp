# 동적 크롤링

- 실시간 데이터를 처리할 수 있으며, 자바 스크립트를 통해 완전한 HTML을 가져올 수 있음

동적 크롤링 실습
``` Python
# pip install selenium
# 웹 애플리케이션의 자동화 테스트를 위해 널리 사용되는 라이브러리
!pip install selenium

```

기본적인 설정 import

``` Python

from selenium import webdriver
# 웹 브라우저를 자동으로 제어할 수 있는 기능을 사용할 수 있게 해준다.

from bs4 import BeautifulSoup
import os
import sys
import urllib.request
import time
import re
```

```python
# webdriver 객체 생성
wd = webdriver.Chrome()

# 객체를 생성한 후 진행해줘야 한다
# wd.get('http://www.daum.net')
wd.get('https://www.coffeebeankorea.com/store/store.asp')

```

```python
# 브라우저의 자바 스크립트를 실행
wd.execute_script("storePop2('31')")

time.sleep(2)  # 필요한 경우 적절한 시간으로 조정

# 해당 스크립트에서 html을 읽어와야한다.
html = wd.page_source

soup = BeautifulSoup(html, 'html.parser')

tr_lists = soup.select('#matizCoverLayer0Content > div > div > div.store_txt > table > tbody:nth-child(1) > tr')
print(tr_lists)

# 데이터 변환
data_dict = {}
for tr in tr_lists:
    th = tr.th
    td = tr.td
    key = th.get_text(strip=True) if th is not None else None
    value = td.get_text(strip=True) if td is not None else None
    data_dict[key] = value

print(data_dict)

# {'영업시간': '평일 08:00~21:00 l 주말 09:00~21:00', '주차': '5천원이상 구매 시 , 1시간 무료주차',
# '주소': '강원특별자치도 원주시 입춘로 110 (반곡동) 파라다이스프라자 101호 일부', '전화번호': '033-746-7951'}
```

여기서부터는 수업 내용과 다르게 진행

try 문으로 에러를 스킵하는 대신 존재하는 매장 리스트의 번호를 저장하여 진행

``` python
# 매장 번호 리스트 전부 저장

wd.get('https://www.coffeebeankorea.com/store/store.asp')

time.sleep(1)  # 페이지 로딩 시간을 고려.

html = wd.page_source
soup = BeautifulSoup(html,'html.parser') # html 정보를 파싱
li_lists = soup.select('#contents2 > div.store_map > div.store_box > div.store_tab > div:nth-child(2) > ul > li')

store_numbers = []
for store in li_lists:
    # '자세히보기' 링크 찾기
    detail_link = store.find('a', class_='btn_style6')['href']
    
    match = re.search(r"storePop2\('(\d+)'\)", detail_link)
    if match:
        store_number = match.group(1)
        store_numbers.append(store_number)

print(store_numbers)

```

```python
store_dict = []
for i in store_numbers:
    print(i)
    
    wd.get('https://www.coffeebeankorea.com/store/store.asp')
    
    time.sleep(1)
    
    wd.execute_script(f"storePop2('{i}')") # get 없이 script만 가져오게될 경우 데이터를 잘 가져오지 못함.

    time.sleep(1)  # 필요한 경우 적절한 시간으로 조정

    # 해당 스크립트에서 html을 읽어와야한다.
    html = wd.page_source

    soup = BeautifulSoup(html, 'html.parser')

    # 데이터 추출
    store_name = soup.find_all('h2')[32].text.strip() # 33번째 h2에 해당 데이터 존재
    business_hours = soup.find('th', string='영업시간').find_next('td').text.strip()
    parking_info = soup.find('th', string='주차').find_next('td').text.strip()
    address = soup.find('th', string='주소').find_next('td').text.strip()
    phone_number = soup.find('th', string='전화번호').find_next('td').text.strip()
    
    # 결과를 딕셔너리 리스트로 변환
    result = {
        '가맹점': store_name,
        '영업시간': business_hours,
        '주차': parking_info,
        '주소': address,
        '전화번호': phone_number
    }
    
    store_dict.append(result)

```
``` python
df = pd.DataFrame(store_dict)
df.head(-1)
```

![image](https://github.com/user-attachments/assets/f832b3e5-5a4f-4741-9c24-119388d19041)

----

## 두 번째 실습 - 지역 검색

```python
wd = webdriver.Chrome()
wd.get('https://www.coffeebeankorea.com/store/store.asp')
html = wd.page_source
```

링크를 클릭하여 변하는 화면 정보를 가져옴

```python
#contents2 > div.store_map > div.store_box > div.search_tab > h3.region_srh
# 지역 검색에 대한 링크 클릭

from selenium import webdriver
from selenium.webdriver.common.by import By
soup = BeautifulSoup(html, 'html.parser') # html 정보 파싱

#contents2 > div.store_map > div.store_box > div.search_tab > h3.region_srh
selenium_el = wd.find_element(By.ID, 'region_srh') # ID가 region_srh인 정보를 검색
selenium_el.click()
```


```python
# dropdown 메뉴를 선택하고 그 메뉴중에서 특정 인덱스 번호에 해당하는 메뉴를 실행
#contents2 > div.store_map > div.store_box > div.search_tab > div:nth-child(4) > div:nth-child(1)

dropdown = wd.find_element(By.ID,'localTitle')
dropdown.click()
time.sleep(1)
dropdown = wd.find_element(By.XPATH,'//*[@id="storeLocal"]/li[1]/a')  # 1 서울 ~ 11
dropdown.click()
```

![image](https://github.com/user-attachments/assets/dbd36164-5cac-4c69-b06c-7006aef00603)

![image](https://github.com/user-attachments/assets/29d927b2-a8ca-4188-b848-7a7e6e680c2a)

```python
store_dict = []
for i in store_numbers:
    print(i)
    
    wd.get('https://www.coffeebeankorea.com/store/store.asp')
    
    time.sleep(1)
    
    wd.execute_script(f"storePop2('{i}')") # get 없이 script만 가져오게될 경우 데이터를 잘 가져오지 못함.

    time.sleep(1)  # 필요한 경우 적절한 시간으로 조정

    # 해당 스크립트에서 html을 읽어와야한다.
    html = wd.page_source

    soup = BeautifulSoup(html, 'html.parser')

    # 데이터 추출
    store_name = soup.find_all('h2')[32].text.strip() # 33번째 h2에 해당 데이터 존재
    business_hours = soup.find('th', string='영업시간').find_next('td').text.strip()
    parking_info = soup.find('th', string='주차').find_next('td').text.strip()
    address = soup.find('th', string='주소').find_next('td').text.strip()
    phone_number = soup.find('th', string='전화번호').find_next('td').text.strip()
    
    # 결과를 딕셔너리 리스트로 변환
    result = {
        '가맹점': store_name,
        '영업시간': business_hours,
        '주차': parking_info,
        '주소': address,
        '전화번호': phone_number
    }
    
    store_dict.append(result)

```

```python
df = pd.DataFrame(store_dict)
df.head(-1)
```

![image](https://github.com/user-attachments/assets/373829fb-e050-433d-ba9b-607c7252fb3e)


-----

## 팀프로젝트

- 프로젝트 주제 : 전국 자동차 등록 현황 및 기업 FAQ 조회시스템

본격적으로 시작하기 전 간단하게 다나와에서 웹 크롤링하는 코드를 작성

```python
!pip install selenium

from selenium import webdriver
from selenium.webdriver.common.by import By
from bs4 import BeautifulSoup
import sys
import urllib.request
import os

import time
import pandas as pd

```

### 국산 브랜드 TOP 5

```python
wd = webdriver.Chrome()
```

```python
wd.get('https://auto.danawa.com/auto/?Work=record')
time.sleep(2)
element_A = wd.find_element(By.CSS_SELECTOR, '#autodanawa_gridC > div.gridMain > article > main > div > div:nth-child(3) > div.left > table')
element_B = wd.find_element(By.CSS_SELECTOR, '#autodanawa_gridC > div.gridMain > article > main > div > div:nth-child(3) > div.right > table')

# 국내 브랜드 TOP5
#autodanawa_gridC > div.gridMain > article > main > div > div:nth-child(3) > div.left > table

# 해외 브랜드 TOP5
#autodanawa_gridC > div.gridMain > article > main > div > div:nth-child(3) > div.right > table

def extract_brand_data(table):
    data_list = []
    for row in table.find_elements(By.TAG_NAME, 'tr'):
        cols = row.find_elements(By.TAG_NAME, 'td')
        if cols:  
            data_dict = {
                'rank': cols[0].text.strip(),
                'title': cols[1].text.strip(),
                'num': cols[2].text.strip(),
                'share': cols[3].text.strip()
            }
            data_list.append(data_dict)
    return data_list
    
data_list_A = extract_brand_data(element_A)
data_list_B = extract_brand_data(element_B)

print(data_list_A)
print(data_list_B)

df = pd.DataFrame(data_list_A)
df.head()
```

![image](https://github.com/user-attachments/assets/8e98d6a1-b92f-4610-ab4a-91e40b43f30f)


``` python
df = pd.DataFrame(data_list_B)
df.head()
```

![image](https://github.com/user-attachments/assets/7d16d145-203b-4fe4-a3fb-e377a9ae61f8)


### 기아 2024 07월 크롤링

```python
wd.get('https://auto.danawa.com/auto/?Work=record&Tab=Grand&Brand=307&Month=2024-07-00&MonthTo=')
time.sleep(2)

element = wd.find_element(By.CSS_SELECTOR, '#autodanawa_gridC > div.gridMain > article > main > div > table.recordTable.model > tbody')
rows = element.find_elements(By.TAG_NAME, 'tr')

data_list = []
for row in rows:
    
    # 다른 class 속성이 있는 경우 건너뜀 - <tr>에 대해서만 작동
    class_attribute = row.get_attribute('class')
    if class_attribute and ('sub' in class_attribute or 'model_' in class_attribute):
        continue  

    # 판매 순위
    rank = row.find_element(By.CLASS_NAME, 'rank').text
    
    # 모델명
    model_element = row.find_element(By.CSS_SELECTOR, 'td.title a')
    model_name = model_element.text.strip()  # 모델명
    
    # 판매량
    sales = row.find_element(By.CLASS_NAME, 'num').text.split(' ')[0].replace(',', '')
    
    # 점유율
    market_share = row.find_element(By.CLASS_NAME, 'rate').text
    
    # 전월 대비 판매량 추출
    format_change = lambda text: '-' + text.split('▼')[0].strip() if '▼' in text else text.split('▲')[0].strip()
    last_month_values = row.find_elements(By.CLASS_NAME, 'right')[1].text.split('\n')
    
    last_month_sales = last_month_values[0].replace(',', '') # 전월 판매량
    last_month_change = format_change(last_month_values[1]) # 전월 대비 증감
    
    data_dict = {
        '판매 순위': rank,
        '모델명': model_name,
        '판매량': sales,
        '점유율': market_share,
        '전월 판매량': last_month_sales,
        '전월 대비 증감': last_month_change
    }
    print(data_dict)
    data_list.append(data_dict)
```

```python
df = pd.DataFrame(data_list)
df.head(-1)
```

![image](https://github.com/user-attachments/assets/c8bda4e0-0cc6-4d6f-82ae-422d24575b74)

![image](https://github.com/user-attachments/assets/a15eda90-a1af-4c7c-be3e-201c94f00828)


