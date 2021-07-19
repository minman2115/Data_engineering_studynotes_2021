.

Data_Engineering_TIL(20210719)

[학습자료]

블로그글 "깃허브에 소스 리뷰어 설정하는 방법 2가지"을 공부하고 정리한 내용입니다.

URL : https://lookingfor.tistory.com/entry/github-auto-assign2


[학습내용]


- 개요

마스터 브랜치에 merge를 할때 일반적으로 pull request를 날려서 reviewer에게 승인을 받고 merge가 되는 경우가 많다. 사용자의 pull request를 승인하는 사람은 일반적으로 고정된 사람일 것이다. 디폴트로 깃허브를 사용하는 경우 pull request를 날릴때마다 매번 reviewer를 지정해줘야 하는데 이게 번거로울 수 있다. 이를 자동으로 해주는 기능이 있다.


- 방법 

방법 1) .github/CODEOWNERS를 사용하여 특정 사용자를 리뷰어로 지정하도록 설정

참고 URL : https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-on-github/about-code-owners

방법 2)  auto-assign 이라는 깃허브 application으로 설정

참고 URL : https://github.com/kentaro-m/auto-assign/

- 상세내용

방법 1) .github/CODEOWNERS를 사용하여 특정 사용자를 리뷰어로 지정하도록 설정

아래와 같이 작성해서 마스터 브랜치 최상단 위치에 파일을 생성한다 그런 다음에 사용자별로 master branch에 pull requests를 날리면 minman2115라는 아이디가 reviewers로 자동 설정되는 것을 확인할 수 있다.

.github/CODEOWNERS 예시


```bash
# Lines starting with '#' are comments.
# Each line is a file pattern followed by one or more owners.

# More details are here: https://help.github.com/articles/about-codeowners/

# The '*' pattern is global owners.

# Order is important. The last matching pattern has the most precedence.
# The folders are ordered as follows:

# In each subsection folders are ordered first by depth, then alphabetically.
# This should make it easy to add new rules without breaking existing ones.

# Global rule:
*           @minman2115
```

방법 2)  auto-assign 이라는 깃허브 application으로 설정

아래의 링크를 통해서 auto-assingn 어플리케이션을 설치한뒤에, 깃 레포를 선택하고 '.github/auto_assign.yml'을 만들어서 아래와 같이 내용을 작성하면 된다.

https://github.com/apps/auto-assign

[install]

STEP 1) Go to auto-assign app page

STEP 2) Add to GitHub

STEP 3) Please choose a repository

STEP 4) Create .github/auto_assign.yml in your repository

.github/auto_assign.yml 예시


```yml
# Set to true to add reviewers to pull requests
addReviewers: true

# Set to true to add assignees to pull requests
addAssignees: true

# A list of reviewers to be added to pull requests (GitHub user name)
reviewers: 
  - reviewerA
  - reviewerB
  - reviewerC

# A list of keywords to be skipped the process that add reviewers if pull requests include it 
skipKeywords:
  - wip

# A number of reviewers added to the pull request
# Set 0 to add all the reviewers (default: 0)
numberOfReviewers: 0
```
