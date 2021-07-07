.

Data_Preprocessing_TIL(20210707)

#### [학습자료]

패스트캠퍼스 온라인 강의 "한 번에 끝내는 파이썬 웹 개발 초격차 패키지 Online." 를 공부하고 정리한 내용입니다.

URL : https://fastcampus.co.kr/dev_online_pyweb

#### [코드편집기를 사용하는 이유]

1) 폴더 및 파일을 쉽게 정리 할 수 있다. 왜냐하면 파일탐색기가 내장되어 있기 때문이다.

2) 코드 자동완성기능

3) 디버깅을 도와주는 보조기능이 있음

4) 유용한 단축키들을 사용할 수 있음

#### [실습내용]

#### step 1) 설치파일 다운로드 및 설치(윈도우 환경 기준)

https://code.visualstudio.com/ 에 접속해서 화면 가운데에 'Download for Windows'를 클릭해서 설치파일을 다운받아 설치한다.

#### step 2) vscode의 root folder 지정

먼저 바탕화면에 'minman_vscode'라는 이름으로 폴더를 만든다.

그런 다음에 vscode 화면으로 돌아와서 좌측상단 메뉴에서 "File" --> "Open Folder" 클릭한다.

바탕화면에 'minman_vscode' 폴더를 선택한다.

그러면 좌측화면 Explorer 메뉴에 minman_vscode 폴더가 설정된 것을 확인할 수 있다.

다시 바탕화면으로 돌아가서 minman_vscode 폴더로 들어간 다음에 test.txt를 하나 만들어보자. 그리고 vscode로 돌아가면 vscode 화면에서 test.txt가 보이는 것을 알 수 있다. 이는 vscode에도 윈도우 탐색기가 내장되어 있기 때문에 가능하다. 따라서 vscode에서 test.txt를 삭제하면 바탕화면으로 돌아가서 minman_vscode 폴더로 가보면 역시 test.txt가 삭제된 것을 알 수 있다.

#### step 3) test python script 작성 및 실행

vscode에서 minman_vscode 하위에 hello.py를 만들고 print("hello world") 를 작성한 다음에 저장한다.

vscode에서 python script를 실행하려고 하면 vscode의 확장프로그램을 설치해야 한다. vscode 좌측화면에 큐브같은 아이콘에 커서를 올려놓으면 extensions라고 표시되는게 있는데 그거를 클릭해보자.

그런다음에 상단에 python 이라고 검색을 하면 마이크로소프트에서 만든 python 확장프로그램이 있는데 그거를 설치를 해준다.

그런 다음에 다시 좌측화면 최상단에 종이 아이콘 explorer를 클릭해서 hello.py 작성한 화면으로 돌아온다.

그 다음에 상단 메뉴에 "Run" --> "Run Without Debugging"을 클릭하면 hello.py를 실행할 수 있다. 또는 우측 상단에 초록색 재생버튼 아이콘이 있는데 그거를 눌러서 실행해도 된다.

#### step 4) python 가상환경 설정

venv라는 패키지를 사용해서 가상환경을 쉽게 만들 수 있다.

minman_vscode 폴더 하위에 가상환경들을 만들것이다. 아래 그림과 같이 명령 프롬프트를 띄운다.

![1](https://user-images.githubusercontent.com/41605276/124765550-09952580-df71-11eb-9e9a-60ce118e7524.png)

그런 다음에 터미널 창에서 아래와 같이 명령어를 날려서 실행한다. 이 명령어가 뭐냐면 minman_venv이라는 python 가상환경을 하나 만들라는 것이다.

`C:\Users\user\Desktop\minman_vscode>python -m venv minman_venv`

그러면 minman_vscode 폴더 아래에 minman_venv라는 폴더가 새로 생긴것을 확인할 수 있다. 그 안에 스크팁트 폴더라는거 안에 있는 내용들이 중요한데 activate라는 것은 해당 가상환경을 활성화하고 그 가상환경 안으로 들어가겠다라는 명령어가 있는 스크립트인 것이고, deactive라는 것은 반대로 비활성화하고 해당 가상환경을 빠져나오겠다라는 명령어가 있는 스크립트이다.

![2](https://user-images.githubusercontent.com/41605276/124766077-8cb67b80-df71-11eb-93e8-815dd68efda3.PNG)

다시 터미널로 돌아가서 `C:\Users\user\Desktop\minman_vscode>minman_venv\Scripts\activate` 를 실행해본다.

그러면 터미널 화면이 갱신되면서 `(minman_venv) C:\Users\user\Desktop\minman_vscode>`로 바뀌게 된다.

가상환경에 어떤 파이썬 패키지가 설치되어 있는지 아래와 같이 터미널에서 커맨드를 날려서 확인해보자. 그리고 requests라는 패키지를 하나 설치해보자.


```console
(minman_venv) C:\Users\user\Desktop\minman_vscode>python --version
Python 3.7.4

(minman_venv) C:\Users\user\Desktop\minman_vscode>pip list
Package    Version
---------- -------
pip        19.0.3 
setuptools 40.8.0 
You are using pip version 19.0.3, however version 21.1.3 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.

(minman_venv) C:\Users\user\Desktop\minman_vscode>pip install requests
Collecting requests
  Downloading https://files.pythonhosted.org/packages/29/c1/24814557f1d22c56d50280771a17307e6bf87b70727d975fd6b2ce6b014a/requests-2.25.1-py2.py3-none-any.whl (61kB)
    100% |████████████████████████████████| 61kB 1.1MB/s
Collecting urllib3<1.27,>=1.21.1 (from requests)
  Downloading https://files.pythonhosted.org/packages/5f/64/43575537846896abac0b15c3e5ac678d787a4021e906703f1766bfb8ea11/urllib3-1.26.6-py2.py3-none-any.whl (138kB)
    100% |████████████████████████████████| 143kB 2.0MB/s
Collecting certifi>=2017.4.17 (from requests)
  Downloading https://files.pythonhosted.org/packages/05/1b/0a0dece0e8aa492a6ec9e4ad2fe366b511558cdc73fd3abc82ba7348e875/certifi-2021.5.30-py2.py3-none-any.whl (145kB)
    100% |████████████████████████████████| 153kB 2.0MB/s
Collecting chardet<5,>=3.0.2 (from requests)
  Downloading https://files.pythonhosted.org/packages/19/c7/fa589626997dd07bd87d9269342ccb74b1720384a4d739a1872bd84fbe68/chardet-4.0.0-py2.py3-none-any.whl (178kB)
    100% |████████████████████████████████| 184kB 3.4MB/s
Collecting idna<3,>=2.5 (from requests)
  Downloading https://files.pythonhosted.org/packages/a2/38/928ddce2273eaa564f6f50de919327bf3a00f091b5baba8dfa9460f3a8a8/idna-2.10-py2.py3-none-any.whl (58kB)
    100% |████████████████████████████████| 61kB 2.2MB/s
Installing collected packages: urllib3, certifi, chardet, idna, requests
Successfully installed certifi-2021.5.30 chardet-4.0.0 idna-2.10 requests-2.25.1 urllib3-1.26.6
You are using pip version 19.0.3, however version 21.1.3 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.

(minman_venv) C:\Users\user\Desktop\minman_vscode>pip list
Package    Version  
---------- ---------
certifi    2021.5.30
chardet    4.0.0
idna       2.10
pip        19.0.3
requests   2.25.1
setuptools 40.8.0
urllib3    1.26.6
You are using pip version 19.0.3, however version 21.1.3 is available.
You should consider upgrading via the 'python -m pip install --upgrade pip' command.
```

그리고 아래 그림과 같이 minman_vscode 폴더 아래에 minman_venv 아래에 venv_test라는 폴더를 만들고 그 안에 test python 스크립트를 하나 만들고 잘 실행되는지 테스트해본다. 

![3](https://user-images.githubusercontent.com/41605276/124768615-c7b9ae80-df73-11eb-9d85-cc43467a5d43.PNG)

vscode를 종료하고 다시 접속하면 터미널에 `C:\Users\user\Desktop\minman_vscode>` 와 같이 가상환경에서 나와있는 형태로 되어 있는데 아래와 같은 명령어로 가상환경을 활성화해서 들어가고 나오고 하면 된다.


```console
C:\Users\user\Desktop\minman_vscode>minman_venv\Scripts\activate

(minman_venv) C:\Users\user\Desktop\minman_vscode>deactivate
    
C:\Users\user\Desktop\minman_vscode>
```
