---
title : "[ISSUE] TestContainer Gitlab CI"
tags :
 - issue

---

# TestContainer Gitlab CI

gitlab ci로 테스트를 진행하는데 테스트 컨테이너를 사용하면 docker가 필요함

-> docker dind (docker in docker)로 해결

gitlab ci로 컨테이너를 띄워서 테스트 컨테이너 진행시 호스트가 없어서 에러가 발생

-> 서버 호스트 이름을 ip로 바꿔서 해결

다른 서버 db 컨테이너 접속 불가

-> gitlab runner network_mode brige -> host 변경

-> docker dind link와 충돌나서 안됨

-> docker dood 완료

동시에 돌리면 컨테이너가 올라가고 git pull 받고 gradle 초기화 과정에서 gitlab volumes에 cache가 들어가있어 충동

-> garadle 위치를 변경해줌

- export GRADLE_USER_HOME=`pwd`.gradle
- export GRADLE_USER_HOME=`pwd`/bj/.gradle



gitlab ci => 쿠버 변경 이슈 종료
