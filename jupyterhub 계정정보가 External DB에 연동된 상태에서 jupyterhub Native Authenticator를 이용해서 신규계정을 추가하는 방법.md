Data_Engineering_TIL(20210417)

[사전 참고자료]

- Native Authenticator 를 이용한 EMR jupyterhub 계정관리

https://minman2115.github.io/DE_TIL178/

- EMR jupyterhub External DB 설정 테스트 예시

https://minman2115.github.io/DE_TIL181/

[학습내용]

#### jupyterhub 계정정보가 External DB에 연동된 상태에서 jupyterhub Native Authenticator를 이용해서 신규계정을 추가하는 방법

step 1) jupyterhub Web UI에서 Create User를 클릭해서 계정을 생성한다.

step 2) admin 화면에 접속해서 Add Users를 클릭한 후 step 1)에서 Create한 user 정보를 추가해준다.

https://[jupyterhub_IP_address]:9443/hub/admin

step 3) admin 권한으로 접속해서 step2)에서 추가한 내용을 승인해준다.

https://[jupyterhub_IP_address]:9443/hub/authorize

step 4) 회원가입된 사용자는 최초 로그인 후 아래의 링크에서 비번을 바꿔준다.

https://[jupyterhub_IP_address]:9443/hub/change-password