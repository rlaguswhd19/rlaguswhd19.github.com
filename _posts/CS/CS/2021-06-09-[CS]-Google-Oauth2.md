---
title: "[CS] Google Oauth2"
tags : 
 - Cs
---



# Google Oauth2

아프리카TV에서 Google API를 사용하게 되어 정리합니다. Oauth2를 사용해 Access Token을 발급받고 Indexing API를 요청하는 과정입니다.

<br/>

## GCP

먼저 Google API를 사용하려면 GCP 계정이 필요합니다. 이 GCP는 제가 지금까지 보고 느낀바로는 Google API를 사용할 수 있는 플랫폼입니다. 그리고 API를 사용하기 위해서는 Oauth2를 사용해서 인증하고 access_token을 발급받아야 사용할 수 있습니다.

![API 및 서비스](https://user-images.githubusercontent.com/46040824/119591037-df4c3600-be10-11eb-96c9-1d8b01a08b3d.png)

<br/>

사용자 인증 정보에서 OAuth 2.0 클라이언트 ID를 생성합니다.

![OAuth2 클라이언트 ID](https://user-images.githubusercontent.com/46040824/119591040-e1ae9000-be10-11eb-9a67-4c2c109450a4.png)

##### 승인된 자바스크립트 원본 

> 웹 애플리케이션을 호스팅하는 HTTP 원본입니다. 이 값에는 와일드 카드나 경로를 포함할 수 없습니다. 80 외의 포트를 사용하는 경우 포트를 지정해야 합니다. 예: https://example.com:8080

##### 승인된 리디렉션 URI

> 사용자가 Google에서 인증을 받은 후 이 경로로 리디렉션됩니다. 이 경로는 뒤에 액세스용 승인 코드가 추가되며 프로토콜이 있어야 합니다. URL 조각, 상대 경로, 와일드 카드는 포함할 수 없으며 공개 IP 주소는 사용할 수 없습니다.

여기서 승인된 리디렉션 URI 설정을 해줘야합니다. 나중에 OAuth2 인증을 받은뒤 성공한 뒤 redirect할 URI을 설정하지 않으면 redirect_url 미스매치 에러가 발생합니다 . 다르게 보면 로그인 인증을 한뒤 이동하는 url이라고 생각하면 됩니다.

<br/>

![client_secret](https://user-images.githubusercontent.com/46040824/120449466-66c61600-c3ca-11eb-8d13-79704157156b.png)

생성하고 보면 Oauth 인증을 받기위한 정보가 있습니다. 클라이언트 ID, 클라이언트 보안 비밀 등이 존재하는데 위에 JSON 다운로드를 통해서 json파일로 다운받을 수 있습니다.

gcp는 이정도면 준비가 된 상태입니다. 이제 composer를 사용해서 Google Api Client 라이브러리를 다운받아보겠습니다.

<br/>

## Composer

#### Composer란?

node.js에는 npm, 파이썬에는 pip, 루비에는 bundle이 있습니다. 이것들은 모두 의존성 도구로 php에서도 대표적인 의존성 도구가 바로 이 composer입니다.

<br/>

#### Composer 설치

##### window

* composer 공식 홈페이지 다운로드하여 간단히 설치 할 수 있습니다.

##### 리눅스

1. curl 요청으로 다운로드 받습니다.

   ```
   curl -sS https://getcomposer.org/installer | php
   ```

2. 설치하면 composer.phar 파일이 생성되는데 이것을 프로젝트뿐만 아니라 global로 사용하기 위해서는 composer.phar을 옮겨줍니다.

   옮기는 경우 `composer`로 간단히 명령어를 실행할 수 있지만 프로젝트 내부에서만 사용할 경우 `php composer.phar` 명령어를 사용해야 합니다.

   ```
   mv composer.phar /usr/local/bin/composer
   ```

<br/>

#### composer 사용

composer 설치가 끝나면 composer.json파일을 생성해서 필요한 라이브러리를 추가하면 됩니다. (https://packagist.org/ 사이트 참고)

Google의 ApiClient를 사용하기 위해서 composer.json을 구성했습니다.

google-api-php-client git : https://github.com/googleapis/google-api-php-client

##### composer.json

```json
{
    "require": {
        "google/apiclient": "^2.7"
    },
    "scripts": {
        //post-update-cmd: update 커맨드가 실행 된 후에 발생
        //경로는 vendor/google/task/composer의 cleanup 함수인듯
        "post-update-cmd": "Google\\Task\\Composer::cleanup"
    },
    "extra": { 
        "google/apiclient-services": [ // 필요한 라이브러리 정의
            "Drive",
            "Oauth2",
            "Indexing"
        ]
    }
}
```

composer.json을 만들고 composer install 명령어를 실행하면 composer.json에 명시된 라이브러리를 설치합니다. 

최초의 다운로드 후 다른 라이브러리를 다운로드 받고 싶으면 지우고 update를 통해 수정된 json으로 다운받습니다.

```
rm -r vendor/google/apiclient-services
composer update
```

<br/>

`composer install`하면 composer.lock 파일이 생성되는데 이 파일의 내용을 열어보면 현재 설치된 라이브러리를 이용하기 위해서 필요한 선행 라이브러리들의 항목과 정확한 버전이 기술되어 있습니다. `composer install`을 실행했을 때 이 파일이 존재한다면 composer는 이 파일에 기술된 라이브러리와 다른 버전의 라이브러리만을 설치할 것입니다.

만약 라이브러리를 최신버전으로 갱신하고 싶다면 아래와 같이 update 명령을 사용하면 됩니다.

```
composer update
```

<br/>

install 후 vendor에 composer의 기본 파일과 라이브러리가 추가됩니다.

google api client의 경우는 vendor/google/apiclient-services에 composer.json에 정의한 라이브러리가 존재할 것입니다.

![image-20210602173655502](https://user-images.githubusercontent.com/46040824/120449319-4ac27480-c3ca-11eb-8130-cb73c4e73971.png)

<br/>

# OAuth2

git : https://github.com/googleapis/google-api-php-client/blob/master/docs/oauth-web.md

OAuth2를 사용해서 access_token을 받아보도록 합니다.



##### login.php

```php
<?php
    
require __DIR__ . '/vendor/autoload.php';

session_start();

$client = new Google\Client();

// gcp 계정 인증 정보를 받아 client에 Set
$client->setAuthConfigFile('client_secrets.json');
$client->setScopes(Google_Service_Indexing::INDEXING);
$client->setAccessType('offline'); // offline access
$client->setIncludeGrantedScopes(true); // incremental auth
$client->setApprovalPrompt('force'); // 계속 refresh_token 발급하기 위한 속성

if (isset($_SESSION['access_token']) && $_SESSION['access_token']) {
    var_dump($_SESSION['access_token']);
} else {
 	// 세션에 access_token이 존재하지 않으면
    $redirect_uri = 'https://' . $_SERVER['HTTP_HOST'] . '/oauth2callback.php';
    // redirect url 설정 oauth2callback.php
    $client->setRedirectUri($redirect_uri);
    // 인증 url 생성
    $auth_url = $client->createAuthUrl();
    // 인증 redirect
    header('Location: ' . filter_var($auth_url, FILTER_SANITIZE_URL));
}
```

Client에 필요한 정보들을 Set한뒤 인증서버로 요청을 보냅니다.

![로그인](https://user-images.githubusercontent.com/46040824/121287215-17c53700-c91c-11eb-80d7-655839d64955.png)

![색인](https://user-images.githubusercontent.com/46040824/121287217-18f66400-c91c-11eb-9db3-1c04c7d8d26a.png)

정상적으로 로그인을 하면 redirect_url로 요청을 보냅니다.

url : https://www.example.com/oauth2callback.php?code=4/0AY0e-g5lvQQtBQNjf6Xvy4lqc8WPuUylYgwaPieMkVUrm_ng4VtE87V24pVcFw1d_4k8Ag&scope=https://www.googleapis.com/auth/indexing

여기서 code가 있는데 oauth2callback.php에서 code를 활용해 access_token을 set한 client를 가져옵니다.



##### oauth2callback.php

```php
<?php
require __DIR__ . '/vendor/autoload.php';

session_start();

$client = new Google\Client();
$client->setAuthConfigFile('client_secrets.json');
$client->setRedirectUri('https://' . $_SERVER['HTTP_HOST'] . '/oauth2callback.php');
$client->addScope(Google_Service_Indexing::INDEXING);

// 인증 후 code를 받는다.
if (!isset($_GET['code'])) { // code가 없으면 다시 인증 요청을 한다.
    $auth_url = $client->createAuthUrl();
    header('Location: ' . filter_var($auth_url, FILTER_SANITIZE_URL));
} else {
    $client->authenticate($_GET['code']); // code로 client에 인증 정보를 set한다.
    $_SESSION['access_token'] = $client->getAccessToken(); // 세션에 저장한다.
    $redirect_uri = 'https://' . $_SERVER['HTTP_HOST'] . '/';
    header('Location: ' . filter_var($redirect_uri, FILTER_SANITIZE_URL));
}
```



이제 access_token을 세션에 저장하고 client를 활용해 API를 요청할 수 있습니다.
