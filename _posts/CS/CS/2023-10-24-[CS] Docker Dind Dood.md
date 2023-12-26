---
title : "[Docker] Dind Dood"
tags :
 - Cs
---



# Dind vs Dood

gitlab ci와 gitlab runner로 ci 진행과정에 이미지를 올려서 하는데 테스트 과정에서 docker가 없어서 에러가 발생했다.

테스트를 하면서 testContainer를 사용했는데 그러면 이미지에 docker가 있어야 하는데 default 이미지를 gradle:jdk17을 사용하고 있었기 때문에 docker가 없었다.

방법을 찾아보던 도중에 docker dind와 dood 방법이 있어 기록한다.

<br/>

#### Dind (Docker in Docker)



<br/>

#### Dood (Docker Out Of Docker)