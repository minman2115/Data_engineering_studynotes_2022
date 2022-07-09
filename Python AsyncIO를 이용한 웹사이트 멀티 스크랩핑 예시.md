.

Data_Engineering_TIL(20220709)

[참고자료]

인프런 "우리를 위한 프로그래밍 : 파이썬 중급 (Inflearn Original)" 강의내용

URL : https://www.inflearn.com/course/프로그래밍-파이썬-중급-인프런-오리지널/dashboard

[학습내용]

먼저 터미널을 열고 테스트할 파이썬 가상환경을 아래와 같은 명령어를 실행해서 구성해준다.

 ```console
$ mkdir asyncio_test

$ cd asyncio_test

$ python3 -m venv asyncio_test

$ cd asyncio_test 

$ ll
total 8
drwxr-xr-x  12 minman  staff   384B  7  9 13:24 bin
drwxr-xr-x   2 minman  staff    64B  7  9 13:24 include
drwxr-xr-x   3 minman  staff    96B  7  9 13:24 lib
-rw-r--r--   1 minman  staff    94B  7  9 13:24 pyvenv.cfg

$ cd bin

$ ll
total 72
-rw-r--r--  1 minman  staff   8.6K  7  9 13:24 Activate.ps1
-rw-r--r--  1 minman  staff   1.9K  7  9 13:24 activate
-rw-r--r--  1 minman  staff   884B  7  9 13:24 activate.csh
-rw-r--r--  1 minman  staff   2.0K  7  9 13:24 activate.fish
-rwxr-xr-x  1 minman  staff   271B  7  9 13:24 pip
-rwxr-xr-x  1 minman  staff   271B  7  9 13:24 pip3
-rwxr-xr-x  1 minman  staff   271B  7  9 13:24 pip3.9
lrwxr-xr-x  1 minman  staff     9B  7  9 13:24 python -> python3.9
lrwxr-xr-x  1 minman  staff     9B  7  9 13:24 python3 -> python3.9
lrwxr-xr-x  1 minman  staff    42B  7  9 13:24 python3.9 -> /opt/homebrew/opt/python@3.9/bin/python3.9

$ source activate
# 비활성화 명령어 : deactivate

$ pip list
Package    Version
---------- -------
pip        22.0.4
setuptools 60.10.0
WARNING: You are using pip version 22.0.4; however, version 22.1.2 is available.
You should consider upgrading via the '/Users/minman/minman_study/asyncio_test/asyncio_test/bin/python3.9 -m pip install --upgrade pip' command.

$ pip install asyncio
Collecting asyncio
  Downloading asyncio-3.4.3-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 KB 1.1 MB/s eta 0:00:00
Installing collected packages: asyncio
Successfully installed asyncio-3.4.3
WARNING: You are using pip version 22.0.4; however, version 22.1.2 is available.
You should consider upgrading via the '/Users/minman/minman_study/asyncio_test/asyncio_test/bin/python3.9 -m pip install --upgrade pip' command.

$ pip install beautifulsoup4
Collecting beautifulsoup4
  Downloading beautifulsoup4-4.11.1-py3-none-any.whl (128 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 128.2/128.2 KB 1.4 MB/s eta 0:00:00
Collecting soupsieve>1.2
  Downloading soupsieve-2.3.2.post1-py3-none-any.whl (37 kB)
Installing collected packages: soupsieve, beautifulsoup4
Successfully installed beautifulsoup4-4.11.1 soupsieve-2.3.2.post1
WARNING: You are using pip version 22.0.4; however, version 22.1.2 is available.
You should consider upgrading via the '/Users/minman/minman_study/asyncio_test/asyncio_test/bin/python3.9 -m pip install --upgrade pip' command.

$ pip list
Package        Version
-------------- -----------
asyncio        3.4.3
beautifulsoup4 4.11.1
pip            22.0.4
setuptools     60.10.0
soupsieve      2.3.2.post1
WARNING: You are using pip version 22.0.4; however, version 22.1.2 is available.
You should consider upgrading via the '/Users/minman/minman_study/asyncio_test/asyncio_test/bin/python3.9 -m pip install --upgrade pip' command.
```

AsyncIO를 이용해서 여러 웹사이트를 동시에 스크래핑하는 프로그램을 구현할 것이다.

AsyncIO는 비동기 input/output 작업을 코루틴을 쉽게할수록 지원해주는 패키지임

None 블러킹 비동기 처리를 지원해준다.

Blocking I/O : 호출된 함수가 자신의 작업이 완료될때까지 제어권을 가지고 있는것을 말한다. 그래서 타 함수는 대기해야함.

None Blocking I/O : 호출된 함수(서브 루틴)가 리턴후 호출한 함수 (메인 루틴)에 제어권 전달하는 것을 말한다. 그래서 타 함수는 일을 계속해서 할 수 있다.

참고로 우리가 실습때 사용할 request 라이브러리는 블러킹 I/O이다. (구글링 하면 나옴)

블러킹 함수는 AsyncIO를 사용하는 것보다 멀티쓰레딩을 사용하는게 더 낫다.

즉 내가 사용할 함수도 제네테레이터 같이 비동기 함수로 구현되어 있어야지 AsyncIO를 사용하는게 효과적이다.

멀티쓰레드를 사용할 경우 단점 : 디버깅에 어려움, 레이스 컨디션(경쟁상태), 데드락을 고려하고 코딩해야 한다.

코루틴의 장점 : 하나의 단일쓰레드에서 실행되는 루틴만 실행해도 락관리가 필요가 없다. 

코루틴의 단점 :코루틴을 사용하려면 사용할 함수가 비동기로 구현이 되어 있어야 하거나 또는 직접 비동기로 구현해야 한다.

아래와 같이 test.py 를 작성해보자.

```python
import asyncio
import timeit
from urllib.request import urlopen
from concurrent.futures import ThreadPoolExecutor
import threading
from bs4 import BeautifulSoup
# urlopen 함수는 블럭 함수임 따라서 asyncio로 urlopen 함수를 여러개 동시에 실행해도 블러킹 되버린다. 즉 asyncio가 비효과적이다.
# 결론적으로 그래서 https://pymotw.com/3/asyncio/executors.html 와 같은 패턴을 사용하면 블러킹 함수를 사용해도 asyncio를 효과적으로 사용할 수 있다.

# 실행시작 시간
start = timeit.default_timer()

urls = ['https://naver.com', 'https://google.com','https://tstory.com','https://minman2115.github.io','https://pymotw.com/3/asyncio/executors.html']

async def fetch(url,executor):
  # 쓰레드 시작상태 출력
  print('Thread Name : ', threading.current_thread().getName(),'Start', url)

  # 실행
  response = await loop.run_in_executor(executor, urlopen, url)
  soup = BeautifulSoup(response.read(), 'html.parser')

  # 전체 페이지 소스 확인
  #print(soup.prettify())

  # 웹사이트의 html 타이틀만 가져오기
  result_data = soup.title

  # 쓰레드 종료상태 출력
  print('Thread Name : ', threading.current_thread().getName(),'Done', url)

  # 결과 반환
  return result_data

# 함수선언 할때 async를 붙여주는 거는 이 함수가 비동기 함수라는 것을 선언하는 것임.
async def main():
  # 쓰레드풀 생성
  executor = ThreadPoolExecutor(max_workers=10)

  # future 객체 모아서 gather해서 실행
  futures = [asyncio.ensure_future(fetch(url,executor)) for url in urls]

  # 결과 취합
  # futures 객체가 리스트 타입이기 때문 *futures 로 언패킹을 한것임
  result = await asyncio.gather(*futures)

  print("Result : ", result)


if __name__ == '__main__':
  # 루프 초기화
  loop = asyncio.get_event_loop()
  # 메인함수가 끝날때까지 루프를 계속 돌린다.
  loop.run_until_complete(main())
  # 수행시간 계산
  duration = timeit.default_timer() - start
  # 프로그램 러닝타임 출력
  print('Running time : ', duration)
```

- test.py를 실행하면 실행결과는 아래와 같음.

```console
$ python3 test.py
Thread Name :  MainThread Start https://naver.com
Thread Name :  MainThread Start https://google.com
Thread Name :  MainThread Start https://tstory.com
Thread Name :  MainThread Start https://minman2115.github.io
Thread Name :  MainThread Start https://pymotw.com/3/asyncio/executors.html
Thread Name :  MainThread Done https://minman2115.github.io
Thread Name :  MainThread Done https://pymotw.com/3/asyncio/executors.html
Thread Name :  MainThread Done https://naver.com
Thread Name :  MainThread Done https://google.com
Thread Name :  MainThread Done https://tstory.com
Result :  [<title>NAVER</title>, <title>Google</title>, None, <title>Whiplash – Data Science Study Notes</title>, <title>Combining Coroutines with Threads and Processes — PyMOTW 3</title>]
Running time :  4.363094542
```