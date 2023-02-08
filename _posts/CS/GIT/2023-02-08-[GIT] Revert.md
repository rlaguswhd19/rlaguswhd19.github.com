---
title: "[GIT] RESET, REVERT"
tags : 
 - Git
---



# Reset vs Revert

둘다 이전 커밋으로 되돌린다는 점에서는 동일하나 revert는 git log(tag) 기록이 남는다.

local에서 사용하는 경우는 `reset`을 사용해도 된다. 하지만 origin의 경우는 git log가 공유되어 있어 다른 사람과 log가 달라질 수 있기 때문에 `revert`을 사용해야 한다.

### Reset options

* --hard

  돌아간 커밋 이후의 변경 이력(log)를 모두 삭제한다.

* --mixed (default)

  돌아간 커밋 이후의 변경 이력은 삭제되지만 변경 내용(unStage)은 남아있다.

  add 이후에 commit이 가능하다.

* --soft

  변경 이력은 모두 삭제하지만 변경 내용(Stage)은 남아있다.

  add 하지 않아도 commit이 가능하다.

<br/>

### origin에 올린 상태에서 reset하고 push

로컬은 origin에 있는 commit을 삭제한 채로 origin에 덮으려고 하면 error가 발생

이럴땐 --force 옵션을 주어 강제로 로컬 commit history를 origin commit history로 덮어쓴다. 다른 사람과 공유하고 있다면 하면 안된다. 다른사람 작업이 날라감

### reset

* 혼자만 사용하는 브랜치의 경우

* origin에 있지만 아무도 이 브랜치를 사용하지 않는 경우

그 이외의 경우는 revert를 사용한다.

<br/>

### revert

revert는 reset과 다르게 커밋을 삭제하지 않고 추가한다.
