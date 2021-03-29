---
title : "[afreecatv] 신규 입사자 게시판 "
tags :
 - afreecatv
---





# 아프리카tv 신규 입사자 프로젝트

신규 입사자 아프리카 소통 게시판 프로젝트

<br/>

## 기술스택

* php 5.6.4
* javascript
* jquery
* axios

<br/>

## 호스팅

* 게시판
  * 203.238.128.141 auth.afreecatv.com
  * 203.238.128.141 login.afreecatv.com
  * 203.238.128.141 member.afreecatv.com
  * 203.238.137.103 erwin.afreecatv.com

<br/>

* 게시판 UI
  * 203.238.150.162 www.design.afreeca.com www.design.nowcom.co.kr

<br/>

## 데이터베이스 ERD

![erd](https://user-images.githubusercontent.com/46040824/112259550-da9dc100-8cab-11eb-9e68-ce738b86fd87.PNG)



## 주요기능

* #### 게시글

* #### 댓글

* #### 답글

* #### 어드민

<br/>

## PostWrite

게시글 쓰기

<br/>

#### 구현 기능

* 게시글
  * 제목 길이 50 이하
  * 내용 3000byte 이하
  * 빈 제목, 내용 작성 불가(스페이스, 엔터)
  * html 태그 그대로 출력
  * 운영자 글 작성시 공지글 등록
  * 이모지 (X)
  * 도메인 하이퍼링크 변환(X)
  * 욕설 및 금칙어 (X)

<br/>

* 게시글 이미지

  * 확장자 바꾼 이미지 검증
  * 이미지 확장자 제한(GIF, JPEG, PNG, BMP)
  * 파일 크기 2mb 이하
  * 파일 이름 길이 30 이하
  * 최대 5개의 이미지 생성 가능
  * 업로드 이미지 미리보기 기능
  * 업로드 이미지 체크박스를 활용해 삭제 가능

<br/>

#### 데이터 flow

```js
createPost() // 게시글을 생성한다.
.then((r)=>{
   	if(preview_length == 0){ // 이미지가 없으면
        postCount(1) // post_list_tbl is_notice에 따라 게시글 or 공지 갯수를 +1 한다.
    }else { // 이미지가 있으면
        createPostImage() // 게시글 이미지를 생성한다.
        .then((r)=>{
            postCount(1) // post_list_tbl is_notice에 따라 게시글 or 공지 갯수를 +1 한다.
        })
        .catch((e)=> { // 이미지를 만들다가 에러가 발생하면
            deletePost() // 게시글을 삭제한다.
        })
    }
})
```

<br/>

## PostList

게시글 목록

<br/>

#### 구현 기능

* 20개씩 페이징 (이전, 다음)
* 공지 숨기기
* 게시글 제목 우측 댓글 개수 노출
* 삭제 버튼 (운영자 기능)
* 글쓰기 버튼 클릭시 게시글 쓰기 페이지 이동

<br/>

#### 데이터 flow

```js
if(!isDetail) { // 디테일이 아니면 == postList.js를 postDetail.php 에서 사용하기 때문
    getPostList(); // 게시글을 가져온다.
    .then((r)=> {
        createPages(); // 게시글 페이징 버튼을 만든다.
        createPost(); // 게시글 목록을 만든다.
    })
        
    getNoticeList(); // 공지 목록을 가져온다.
    .then((r)=> {
        createNotice(); // 공지 목록을 만든다.
    })
}
```

<br/>

검색

#### 구현 기능

* 닉네임 + 아이디 | 제목 | 내용 다중선택 가능
* 최대 10자 이하 검색
* 전체 버튼 클릭시 전체 리스트 노출
* Enter 또는 검색 버튼 클릭시 검색 기능
* 검색어에 대한 결과가 없을 경우 경고창 노출

<br/>

#### 데이터 flow

```js
// 이전, 다음 버튼 클릭시
// 현재 페이지 정보와 limit 정보를 활용해서 list를 가져온다.
if(!search) { // 검색중이 아니면 전체 게시글을 가져온다.
    getPostList();
}else{ // 검색중이면 검색 게시글을 가져온다.
    getSearchList()
}

// 검색을 누르면 체크박스와 input을 확인한 뒤 검색을 한다.
checkSearch() // 체크박스 확인
getSearchList() // 검색 리스트 가져오기

```

<br/>

## PostDetail

게시글 읽기

<br/>

#### 구현 기능

* BJ명 클릭시 방송국 이동
* 공지 글일 경우 제목 앞에 공지 표시
* up버튼
* 게시글 버튼
  * 작성자 = 수정, 삭제, 목록 버튼
  * 운영자 = 삭제, 목록
  * 작성자도 운영자도 아닌 유저 = 목록
* 조회수 증가
* 게시글 댓글, 답글 수 노출

<br/>

#### 데이터 flow

```js
getParam() // URL의 파라미터로 검색중인지 아닌지 파악후 검색 중이면 검색에 필요한 파라미터를 파싱한다.
setParamToData() // 파라미터로 얻은 데이터를 js 전역변수에 Set 한다. + 검색 체크박스 및 검색 input Set
udpateViewCount() // 조회수를 +1 한다.
.then((r)=> {
   	// 게시물
    getPostById() // 게시글 정보를 가져온다.
    getPostImageByPostId() // 게시물 이미지 정보를 가져온다.
   
   	// 게시물 리스트
    getNoticeList() // 공지 게시글을 가져온다.
   	// 검색 중인지 판단하여 게시글 리스트를 가져온다.
    if (search) {
        getSearchList()
    } else {
        getPostList()
    }
})
.catch((e)=> { // 게시글이 없는 경우
    alert('삭제된 게시글입니다.')
})
```

<br/>

up버튼

<br/>

#### 구현 기능

* up을 했는지 검증한 뒤 up 카운트


<br/>

#### 데이터 flow

```js
findUpByPostId() // 계정이 up을 했는지 up 테이블을 조회하여 확인한다.
.then((r)=>{
    createUpBtn() // up 버튼을 만든다.
})
    
// up 버튼을 누르면
if(is_up) { // up을 한 상태
    deleteUp() // up 데이터 삭제 및 게시글 up_cnt-=1
}else { // up을 안한 상태
    createUp() // up 데이터 생성 및 게시글 up_cnt+=1
}
```

<br/>

## PostUpdate

게시글 수정

<br/>

#### 데이터 flow

```js
var post_image_list = [] // 기존의 이미지 배열
var previews = [] // 추가 이미지 배열
var post_image_delete = [] // 기존의 이미지 중 삭제될 이미지 배열

getParam() // 파라미터로 post_no를 가져온다.
getPostById() // post를 가져와서 데이터를 뿌려준다.
getPostImageByPostId() // postImage를 가져와서 데이터를 뿌려준다.

// 게시글을 수정한뒤 확인 버튼을 누른다.
btn_ok.on("click", function () {
    
    // 기존에 있던 이미지(기존에 있던 이미지를 삭제한 경우 배열에서 제외)와 추가된 이미지 개수 5개
    if (previews.length + post_image_list.length > 5) {
        alert("이미지 개수가 5개 이상입니다.")
        return
    }
    
    // 추가될 이미지가 0개이면
	if(preview.length === 0) {
        deletePostImageById() // 기존에 있던 이미지 중 지울것을 삭제한다.
        .then((r)=>{
            updatePost() // 게시글 내용과 제목을 업데이트
        })
    }else { // 추가된 이미지가 있으면
        createPostImage() // 추가된 이미지를 생성한다.
        .then((r)=>{
            if(post_image_delete.length === 0) { // 서버 이미지 중 지울게 없으면
                updatePost() // 게시글 내용과 제목을 업데이트
            }else{
                deletePostImageById() // 기존에 있던 이미지 중 지울것을 삭제한다.
                .then((r)->{
                    updatePost() // 게시글 내용과 제목을 업데이트
            }
        })
    }
}
```

<br/>

## PostDelete

게시글 삭제

<br/>

#### 데이터 flow

```js
// 게시글 삭제 버튼
deletePostAll() {
    getCommentListByPostId() // 게시글 넘버로 댓글들 가져오기
    .then((r)=>{
        // comment_nums_all 게시글 관련 모든 댓글 넘버
        // comment_nums = 답글이 있는 댓글 넘버 
        
        // 댓글이 없는 경우 게시글 삭제
        if (comment_nums_all.length === 0) {
            // 1번
            // 게시글 삭제 1.게시글 up 2.게시글 이미지 3. 게시글 4.게시글 리스트 count -= 1
            deletePostUpById()
        } else {
            // 답글이 없는 경우
            if (comment_nums === 0) {
                // 2번
                // 댓글 삭제 1.댓글 up 2.댓글 이미지 3.댓글
                deleteCommentUpListByCommentIdList()
            } else {
                // 3번
                // 답글 삭제 1.이미지 2.답글
                deleteRepleImageListByCommentIdList()
            }
        }
        	
        // 전체 데이터 흐름
        deleteRepleImageListByCommentIdList() // 댓글 번호들로 답글들 이미지 삭제 -> *****3번*****
        .then((r)=> {
            deleteRepleListByCommentIdList() // 댓글 번호들로 답글 삭제
            .then((r)=> {
                deleteCommentUpListByCommentIdList() // 댓글들 up 삭제 -> *****2번*****
                .then((r)=>{
                    deleteCommentImageListByCommentIdList() // 댓글들 번호로 댓글 이미지 삭제
                    .then((r)=>{
                        deleteCommentListByPostId() // 게시글 번호로 댓글들 삭제
                        .then((r)=>{
                            deletePostUpById() // 게시글 up 삭제 -> *****1번*****
                            .then((r)=>{
                                deletePostImageByPostId() // 게시글 이미지 삭제
                                .then((r)=> {
                                    deletePostById() // 게시글 삭제
                                    .then((r)=>{
                                        postCount() // 게시글 리스트 카운트 -1
                                    })
                                })
                            })
                        })
                    })
                })
            })
        })
    })
}
```

<br/>

## CommentWrite

댓글 쓰기

<br/>

#### 구현 기능

* 내용 최대 500자 작성 가능
* 댓글 내용 수정 시 현재 내용 길이 출력
* 댓글 내용 html 태그 가능
* 이미지 최대 1개 업로드
* 댓글 up (검증)
* 운영자 댓글 작성 기능
* 이모지 (X)
* 도메인 하이퍼링크 변환 (X)

<br/>

#### 데이터 flow

```js
// 댓글 등록 버튼을 누르면
createComment() // 댓글 등록
.then((r)=>{
    if(exist_preview) { // 이미지가 있으면
        createCommentImage() // 댓글 이미지를 생성한다.
        .then((r)=>{
            commentCount(1) // cnt_comment += 1 댓글 수를 증가시킨다.
        })
        .catch((e)=>{ // 이미지를 생성하다 에러가 발생하면
            deleteCommentById() // 댓글을 지운다.
        })
    }else { // 이미지가 없으면
        commentCount(1) //  cnt_comment += 1 댓글 수를 증가시킨다.
    }
})
```

<br/>

## CommentList

댓글 리스트

<br/>

#### 구현기능

* 댓글 5개씩 노출 (댓글 더보기 기능)
* 댓글 버튼
  * 댓글 작성자 = 수정, 삭제 노출
  * 운영자 = 삭제 노출
  * 댓글 작성자와 운영자가 아닌 유저 = 노출 X

<br/>

#### 데이터 flow

```js
getCommentListByLimit() // 댓글을 page*5개씩 가져온다.
.then((r)=>{
    if(comment_list.length !== 0){ // 댓글이 존재하면
    	getCommentImageListById() // 댓글 이미지 가져오기
        .then((r)=>{
            createImageMap() // 이미지 데이터 댓글 번호로 map 생성
            findCommentUpByCommentId() // 댓글 리스트의 up 가져오기
            .then((r)=>{
                createUpMap() // up 데이터 댓글 번호로 map 생성
                getRepleByCommentId() // 답글 데이터 댓글 번호로 가져오기
                .then((r)=>{
                    createRepleMap() // 답글 데이터 댓글 번호로 map 생성
                    getRepleImageByCommentId() // 답글 이미지 답글 번호로 가져오기
                    .then((r)=>{
                        // 댓글 리스트 생성
                        createCommentList()
                    })
                })
            })
        })
    }
})
```

<br/>

## CommentUpdate

댓글 수정

<br/>

#### 데이터 flow

```js
updateComment() { // 수정 버튼을 클릭하면
    createCommentUpdate(); // 수정폼을 생성한다.
}

clickUpdateBtn() { // 수정폼의 등록 버튼을 클릭하면 
    updateCommentById() // 댓글 내용 변경
    .then((r)=>{
        if(update_comment_preview.file !== undefined) { // 새로운 이미지가 있으면
            updateCommentImage() // 댓글 이미지 업데이트
            .then((r)=>{
                if(delete_comment_image !== -1) {
                    deleteExistCommentImageById() // 기존의 이미지 지운다.
                    .then((r)=>{ // ***새로운 이미지 추가, 기존의 이미지 삭제***
                        getCommentListByLimit() // 댓글 리스트 생성
                    })
                }else{ // ***새로운 이미지만 추가***
                    getCommentListByLimit() // 댓글 리스트 생성
                }
            })
        }else { // 새로운 이미지가 없으면 *** 기존의 이미지만 삭제 ***
            if(delete_comment_image !== -1) { // 기존의 이미지를 지워야 한다면
                deleteExistCommentImageById() // 기존의 이미지 지운다.
                .then((r)=>{
                    getCommentListByLimit() // 댓글 리스트 생성    
                })
            }else { // 댓글 내용만 바꾼 경우 *** 댓글 내용만 수정 ***
                getCommentListByLimit() // 댓글 리스트 생성  
            }
        }
    })
}
```

<br/>

## CommentDelete

댓글 삭제

<br/>

#### 데이터 flow

```js
clickDeleteComment() { // 댓글 삭제 버튼 클릭
    deleteRepleImageListByCommentId() // 답글 이미지 리스트 지우기
    .then((r)=>{
        deleteRepleListByCommentId() // 답글들 지우기
        .then((r)=>{
            if(image_exist) { // 댓글 이미지가 존재하는지 확인
                deleteCommentImageByCommentId() // 댓글 이미지 삭제
                .then((r)=>{
                    deleteCommentUpByCommentId() // 댓글 up 삭제       
                    .then((r)=> {
                        deleteCommentById() // 댓글 삭제
                        .then((r)=> {
                            deleteCommentCount() // 답글 n, 댓글 1 게시글 댓글 수 update
                        })
                    })	
                })
            } else{ // 댓글 이미지가 존재하지 않으면
                deleteCommentUpByCommentId() // 댓글 up 삭제
                .then((r)=>{
                    deleteCommentById() // 댓글 삭제
                    .then((r)=> {
                        deleteCommentCount() // 답글 n, 댓글 1 게시글 댓글 수 update
                    })
                })
            }
       	})
    })
}
```

<br/>

## RepleWrite

답글 쓰기

<br/>

#### 구현 기능

* 답글 클릭시 답글 리스트 노출
* 답글 최대 500자
* 이미지 최대 1개 (이미지 검증 적용)
* 답글 내용 수정 시 현재 내용 길이 출력
* 답글 열어 놓은 것 그대로 유지 (답글 쓰기, 삭제시 열어 놓은 답글 유지 but 내용 및 이미지는 초기화됌)
* 답글 내용  html 태그 작성 가능
* 운영자 답글 작성 기능
* 이모지(X)
* 도메인 하이퍼링크변환(X)

<br/>

<br/>

#### 데이터 flow

```js
clickCreateReple() { // 답글 등록 버튼 클릭
    createReple() // 답글 생성
    .then((r)=>{
        // 이미지가 있으면 이미지 생성
        if(reple_image) {
            createRepleImage() // 리플 이미지 생성
            .then((r)=>{
                commentAndRepleCount(1) // 게시글 댓글+=1, 댓글 리플+=1
            })
            .catch((e)=>{
                deleteRepleById() // 리플 삭제    
            })
        }else{
            commentAndRepleCount(1) // 게시글 댓글+=1, 댓글 리플+=1
        }
    })
}
```

<br/>

# RepleUpdate

답글 수정

<br/>

#### 데이터 flow

```js
updateReple() { // 수정을 누르면
    createUpdateReple() // 수정폼을 만든다.
}

clickUpdateRepleBtn() {
    updateRepleById(comment_no) // 댓글 내용 변경
    .then((r)=>{
        if (update_reple_preview.file !== undefined) { // 새로운 이미지가 있으면
            updateRepleImage(comment_no) // 답글 이미지를 바꾼다.
            .then((r)=>{
                if (delete_reple_image !== -1) { // 기존의 지워야할 답글 이미지가 있으면
                    deleteExistRepleImageById(update_reple_no) // 기존의 이미지를 지운다.
                    .then((r)=>{ // ***새로운 이미지 추가, 기존의 이미지 삭제***
                        getCommentListByLimit() // 댓글 리스트를 만든다.
                    })
                }else{ // ***새로운 이미지만 추가***
                    getCommentListByLimit() // 댓글 리스트를 만든다.
                }
            })
        } else { // *** 기존의 이미지만 삭제 ***
            if (delete_reple_image !== -1) { // 기존의 지워야할 답글 이미지가 있으면
                deleteExistRepleImageById(update_reple_no) // 기존의 이미지를 지운다.
                .then((r)=>{
                    getCommentListByLimit() // 댓글 리스트를 만든다.    
                })
            }else{ // *** 댓글 내용만 수정 ***
                getCommentListByLimit() // 댓글 리스트를 만든다.
            }
        }
    })
}
```

<br/>

<br/>

## RepleDelete

답글 삭제

<br/>

#### 데이터 flow

```js
clickDeleteReple() { // 답글 삭제 버튼을 누르면
    if (exist_image) { // 이미지가 있으면
        deleteOneRepleImageByRepleId() // 이미지 삭제
        .then((r)=>{
            deleteOneRepleById() // 답글 삭제
            .then((r)=> {
                commentAndRepleCount(-1) // 댓글의 답글 cnt -= 1, 게시글의 댓글 cnt -= 1
            })
        })
    } else {
        deleteOneRepleById() // 답글 삭제
        .then((r)=>{
            commentAndRepleCount(-1) // 댓글의 답글 cnt -= 1, 게시글의 댓글 cnt -= 1
        })
    }
}
```

<br/>

## AdminCreate

어드민 생성

<br/>

#### 구현기능

* station_tbl을 통해서 실제 있는 유저인지 검증한다.
* 이미 어드민으로 등록되어 있는지 검증한다.
* 입력값이 비어있는지 검증한다.

<br/>

#### 데이터 flow

```js
clickAdminCreate() { // 추가 버튼을 누르면
	createAdmin() // admin을 생성한다.
	.then((r)=>{
		getAdminListAll() // admin 리스트를 만든다.
	})
}
```

<br/>

## AdminUpdate

어드민 수정

<br/>

#### 데이터 flow

```js
clickAdminUpdate() { // 수정 버튼을 누르면
	updateAdmin()
	.then((r)=>{
		getAdminListAll() // admin 리스트를 만든다
	})
}
```

<br/>

<br/>

## AdminDelete

어드민 삭제

<br/>

#### 데이터 flow

```js
clickAdminDelete() { // 삭제 버튼을 누르면
	deleteAdmin() // admin을 삭제한다.
	.then((r)=>{
		getAdminListAll() // admin 리스트를 만든다.
	})
}
```

<br/>



# 느낀점

1. #### API에 대한 잘못된 생각

   > 나는 지금까지 API는 어떤 Request를 받으면 명확하게 명시된 한가지 일을 해야한다고 생각했다. 왜냐하면 나눠진 기능을 조합하여 더욱 많은 기능과 서비스를 구현할 수 있기 때문이다. 그래서 API를 구현하면서 데이터 처리 및 DB조작을 한가지 요청에 한가지 작업만 했다. 하지만 이렇게 구현하게 된다면 어떤 작업을 해야할 때 서버에 너무 많은 요청을 하게 된다. 
   
   
   
   > 예를 들어 게시글을 삭제하는 데이터 플로우를 본다.
   >
   > 이걸 보면 서버에 게시글을 삭제하면서 순차적으로 여러가지 삭제 및 수정 요청을 보내는걸 확인할 수 있다. 하지만 이렇게 하면 서버 트래픽이 증가하고 프론트가 요청을 받고 다음 요청을 하므로 느려지게 된다.  그리고 모바일의 경우 서버에 많은 요청을 하기 때문에 많은 데이터를 소비하게 된다. 이런 이유로 api는 여러 기능을 구현하되 요청은 하나를 받아서 백엔드 내부에서 작업을 하는것이 좀 더 서버 트래픽을 줄 일 수 있는 방법이라고 생각한다. 그리고 프론트에서 데이터 처리에 대한것들을 해야 서버에서의 부하를 줄일 수 있다. 프론트는 각 유저의 컴퓨터로 처리하지만 서버는 서버 컴퓨터 하나가 처리를 하기 때문이다.

   <br/>

2. #### Validation(검증)

   > 서버 Validator를 통해서 Controller에서 받은 데이터를 검증하고 그에 맞게 응답 헤더와 데이터를 응답하여 프론트가 데이터를 해석해 대응하는것을 설계했다. 하지만 이는 어쨋든 서버로 요청을 보내고 이것도 트래픽 문제로 직결된다. 그렇기 때문에 프론트는 최대한 서버에 요청을 적게 보내야하기 때문에 프론트에서도 검증 로직이 필요하다. 프론트의 검증로직으로 서버에 보내는 요청을 최소화 하여 서버 트래픽을 줄여야 한다.

   <br/>

3. #### DB 설계

   > DB를 설계하면서 역시 명확성이 중요하다고 생각했다. 예를 들면 답글과 댓글은 이미지가 한개만 존재하기 때문에 굳이 이미지 DB를 나눌 필요가 없었다. 하지만 image라는 명확한 데이터가 있어야 한다고 생각을 해서 분리를 했다. 이로 인해서 mysql은 댓글과 관련된 crud를 할 때 이미지에 query를 하나 더 보내야하는 이슈가 발생했다.

   <br/>

4. #### IE11 es6

   > IE는 es6를 지원하지 않는다. 아프리카tv는 IE를 지원해야 하기 때문에 es6문법을 사용하는것을 피해야 한다. 이점을 고려하지 않고 게시판을 구현하면서 let과 애로우 function을 사용함으로써 IE에서는 게시판이 동작하지 않는 이슈가 발생했다. 앞으로 개발을 하면서 es6를 고려하거나 babel(es6문법을 수정해주는 라이브러리)를 사용하는것을 고려해야 겠다.

   <br/>

5. #### 네이밍

   > 파일명, 변수명등에 사용하는 네이밍 규칙이 일정하지 않았다. 카멜케이스를 사용하려고 했으나 두단어가 되지 않는것은 소문자가 되었고 afreecatv의 클래스는 앞에 c를 붙임으로서 다른 파일은 cPostService.php인데 반해 Controller는 클래스가 아니기 때문에 postController.php가 되어 일관성이 없다고 생각해서 대문자로 했으나 잘못된 생각이였다. 프론트의 소스에서도 함수가 많아지면서 네이밍이 복잡하고 길어져서 가시성이 떨어졌다.

   <br/>

6. #### Axios의 Promise 객체

   > 이번 프로젝트를 하면서 Promise객체에 대해 알게되었다. 코타님께서 피드백해주신걸 보면서 axios와 ajax의 차이점은 무엇인지 찾아보게 되었다. ajax의 콜백함수는 ajax요청을 보내면서 쓰는데 반해 axios는 객체를 반환하여 .then()이나 .catch()로 요청후의 응답을 명시할 수 있고 따로 사용할 수 있다. 이로 인해서 가시성이 확보되는데 개발 당시에는 몰라서 axios에 콜백함수처럼 .then()을 사용해서 비슷한 여러 함수가 생성되었다. 이번 기회에 Axios에 대해 좀 더 알아봐야겠다.

