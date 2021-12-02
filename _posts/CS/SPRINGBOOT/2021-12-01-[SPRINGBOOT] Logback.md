---
title : "[SPRINGBOOT] Logback"
tags :
 - Springboot
---

# Logback

Java기반 Logging Framework로 가장 널리 사용되고 있다. SLF4j의 구현체이다.

순서 : log4j -> logback -> log4j2 

log4j보다 10배 빠르게 수행 및 메모리 효율성 증가

http://logback.qos.ch/manual/index.html

<br/>

#### 특징

##### Automatic Reloading Configuration File

설정 파일을 스캔하는 별도의 스레드를 두어 지정한 시간마다 설정 파일을 스캔해 프로그램의 재시작 없이 설정을 변경할 수 있다.

##### Automatic Compression / Removal

로그 파일을 생성할 때, 별도의 프로그램을 통해 압축을 진행할 필요 없이 자동 압축을 지원하며, 시간 또는 개수를 설정하여 설정한 시간이나 개수를 초과할 경우 로그 파일이 자동으로 삭제되도록 할 수 있다.

##### Graceful Recovery From I/O Failures

로그 기록 중에 장애가 발생할 경우, 서버 중지 없이 장애 발생시점으로부터의 자동 복구를 지원한다.

##### If Statement

설정 파일에 조건문을 사용하여 세밀하게 로깅 설정하여 여러 환경에 맞게 자동 구성되도록 할 수 있다.

<br/>

<br/>

## dependencies

```
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'ch.qos.logback:logback-classic:1.2.3'
implementation 'org.slf4j:jcl-over-slf4j:1.7.25'
    
compileOnly 'org.projectlombok:lombok'
```

<br/>

<br/>

## Log level

> level 5 - ⛔️ ERROR
>
> level 4 - ⚠WARN - 경고
>
> level 3 - ✅INFO - 정보
>
> level 2 - ⚙️DEBUG - 디버그
>
> level 1 - 📝TRACE - 디버그 보다 상세한 정보

<br/>

출력 레벨의 설정에 따라 설정 레벨 이상의 로그를 출력한다.

(ex INFO로 설정할 경우 DEBUG와 TRACE는 무시한다.

<br/>

<br/>

## 설정 파일

path = resources/logback-spring.xml

<br/>

1. logback-spring.xml

2. logback-spring.xml이 없으면 [.yml, .properites]을 읽는다.
3. logback-spring.xml와 [.yml, .properites]이 동시에 있으면 [.yml, .properties]를 먼저 읽은후 logback-spring.xml을 적용한다.

<br/>

spring.profiles.active를 활용하여 원하는 logback설정을 적용할 수 있다.

* spring.profiles.active=local
  * application-local.properties
  * logback-local.properties

<br/>

Springboot는 간단하게 application.propreties에서 설정이 가능하다.

````
logging.level.root=info
logging.level.com.search.api=info // 패키지별로 처리 가능
````

<br/>

#### logback-spring.xml

* `springProfile` : spring.profile.active 설정에 따른 properties를 가져오는것을 의미한다.

* `property` : 상수를 설정하는데 값은 propreties에서 가져올 수 있다. (ex ${log.file_path}

* `appender` : log 출력 방법을 설정한다. 출력위치, 출력내용에 대한 패턴을 설정할 수 있다.

  > ch.qos.logback.core.ConsoleAppender : 콘솔 로그
  >
  > ch.qos.logback.core.FileAppender : 파일 로그
  >
  > ch.qos.logback.core.rolling.RollingFileAppender : 여러개 파일 순회 로그
  >
  > ch.qos.logback.classic.net.SMTPAppender : 메일 로그
  >
  > ch.qos.logback.classic.db.DBAppender : DB 로그

* `encoder` : `appender`에 포함되어 사용자가 지정한 형식으로 표현 될 로그메시지를 변환하는 역할을 담당한다.

* `Pattern` : 로그 출력 패턴을 정의한다.

  > %Logger{length} : Logger name을 축약할 수 있다. {length}는 최대 자리 수, ex)logger{35}
  >
  > %-5level : 로그 레벨, -5는 출력의 고정폭 값(5글자)
  >
  > %msg : - 로그 메시지 (=%message)
  >
  > ${PID:-} : 프로세스 아이디
  >
  > %d : 로그 기록시간
  >
  > %p : 로깅 레벨
  >
  > %F : 로깅이 발생한 프로그램 파일명
  >
  > %M : 로깅일 발생한 메소드의 명
  >
  > %l : 로깅이 발생한 호출지의 정보
  >
  > %L : 로깅이 발생한 호출지의 라인 수
  >
  > %thread : 현재 Thread 명
  >
  > %t : 로깅이 발생한 Thread 명
  >
  > %c : 로깅이 발생한 카테고리
  >
  > %C : 로깅이 발생한 클래스 명
  >
  > %m : 로그 메시지
  >
  > %n : 줄바꿈(new line)
  >
  > %% : %를 출력
  >
  > %r : 애플리케이션 시작 이후부터 로깅이 발생한 시점까지의 시간(ms)

* `file` : 기록할 파일 경로를 지정한다.

* `rollingPolicy` : 순회 정책

  * ch.qos.logback.core.rolling.TimeBasedRollingPolicy  - 일자별 적용
  * ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP - 일자별 + 크기별 적용

* `fileNamePattern` : 파일 쓰기가 종료된 파일의 패턴을 지정한다.

* `maxFileSize` : 한 파일당 최대 파일 용량을 지정한다. log 내용의 크기도 IO성능에 영향을 미치기 때문에 되도록이면 너무 크지 않은 사이즈로 지정하는 것이 좋다.(최대 10MB 내외 권장) 용량의 단위는 KB, MB, GB 3가지를 지정할 수 있다.

* `maxHistory` : 최대 파일 생성 갯수를 지정한다. 예를들어 maxHistory가 30이고 rollover 일 단위로 하면 30일동안만 저장되고, 월 단위로 하면 30개월간 저장된다.

  rollover는 `fileNamePattrrn`의 최소 단위에 따라 지정된다. (ex $d{yyyy-MM} => 월단위

* `filter` : 로그 필터

  * #### Regular Filters

    추상클래스 Filter를 상속하며 ILoggingEvent event를 파라미터로 받아오기 위한 decide() 메소드를 구현. 등록된 Filter는 차례대로 decide(ILoggingEvent event) 메소드를 호출하여, enum 객체인 FilterReply를 반환.

    | FilterReply | Description                                                  |
    | ----------- | ------------------------------------------------------------ |
    | DENY        | 로그 이벤트 동작을 취소하며, 남아있는 Filter들에 대하여 검증 하지 않는다. |
    | NEUTRAL     | 남아있는 다음 Filter에게 검증을 넘긴다. 만약 남아있는 Filter가 없으면 로그 이벤트가 정상적으로 동작한다. |
    | ACCEPT      | 로그 이벤트를 정상적으로 동작시키며, 남아있는 Filter들에 대하여 검증하지 않는다. |

    <br/>

    ##### Custom Filter

    ```java
    import ch.qos.logback.classic.spi.ILoggingEvent;
    import ch.qos.logback.core.filter.Filter;
    import ch.qos.logback.core.spi.FilterReply;
    
    public class LogAccessFilter extends Filter<ILoggingEvent> {
        @Override
        public FilterReply decide(ILoggingEvent event) {
            if(event.getMessage().contains("Access Log")) {
                return FilterReply.ACCEPT;
            }else{
                return FilterReply.DENY;
            }
        }
    }
    ```

    Filter는 appender 태그 하위에 넣어 추가한다.

    ```xml
    <filter class="com.search.api.common.log.LogAccessFilter"/>
    ```

    <br/>

    ##### AbstractMatcherFilter

    AbstractMatcherFilter는 onMatch, onMismatch 두 가지의 속성으로 일치/불일치에 대한 적절한 응답을 지정할 수 있는 골격을 제공. 대부분 regular filter들은 이 필터를 상속하여 만들어짐.

    ```java
    package ch.qos.logback.core.filter;
    
    import ch.qos.logback.core.spi.FilterReply;
    
    public abstract class AbstractMatcherFilter<E> extends Filter<E> {
    
        protected FilterReply onMatch = FilterReply.NEUTRAL;
        protected FilterReply onMismatch = FilterReply.NEUTRAL;
    
        final public void setOnMatch(FilterReply reply) {
            this.onMatch = reply;
        }
    
        final public void setOnMismatch(FilterReply reply) {
            this.onMismatch = reply;
        }
    
        final public FilterReply getOnMatch() {
            return onMatch;
        }
    
        final public FilterReply getOnMismatch() {
            return onMismatch;
        }
    }
    ```

    <br/>

    ##### Level Filter

    지정한 level과 같은 level의 로그 이벤트를 필터링한다. 지정한 level과 같다면 onMatch, 같지 않다면 onMismatch에 따른 FilterReply로 처리됨. AbstrachMatcherFilter를 상속했기 때문에 onMatch와 onMismatch를 지정해 주어야 한다.

    ```xml
    <filter class="ch.qos.logback.classic.filter.LevelFilter">
    	<level>ERROR</level>
        <onMatch>ACCEPT</onMatch>
        <onMismatch>DENY</onMismatch>
    </filter>
    ```

    <br/>

    ##### ThresholdFilter

    지정된 level 임계점에 대해 로그 이벤트를 필터링한다. 이 필터는 AbstractFilter를 상속하지 않았기에 onMatch, onMismatch를 지정하지 않음을 확인할 수 있다.

    ```xml
    <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
    	<level>INFO</level>
    </filter>
    ```

    <br/>

  * #### Turbo Filters

    TurboFilter 객체들은 모두 추상 클래스 TurboFilter를 상속한다.

    TurboFilter는 다른 Filter들과 같이 decide() 메소드를 구현하여 FilterReply를 반환하며 동작하지만 차이점이 존재한다.

    TurboFilter는 기존 Filter보다 더 넓은 Scope를 가지고 있다. logging context와 연결되어 있어 appender-attached가 아닌 전체 로그 요청에 대해 필터링한다.

    TurboFilter는 LoggingEvent객체가 생성하기 이전에 호출된다. 즉 TurboFilter 객체가 로깅 요청을 필터링하기 위해 LoggingEvent를 인스턴스화 할 필요가 없다. 이는 TurboFilter가 로그 이벤트 필터링을 더욱 빠르게 처리할 수 있도록 만들어졌기 때문이다.

    <br/>

    ##### Custom Filter

    TurboFilter 추상 클래스를 상속해서 만들 수 있다.

<br/>

##### logback-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="10 seconds">
    <springProfile name="local">
        <property resource="application-local.properties"/>
    </springProfile>

    <springProfile name="prod">
        <property resource="application-prod.properties"/>
    </springProfile>

    <springProfile name="default">
        <property resource="application.properties"/>
    </springProfile>

    <property name="LOG_PATTERN" value="%d{yy-MM-dd HH:mm:ss} [%-5level] %logger{36}[line: %L] - %msg%n"/>
    <property name="LOG_FILE_PATH" value="${log.file_path}"/>
    <property name="LOG_FILE_NAME_INFO" value="${log.file_name_info}"/>
    <property name="LOG_FILE_NAME_ERROR" value="${log.file_name_error}"/>
    <property name="LOG_FILE_NAME_ACCESS" value="${log.file_name_access}"/>
    <property name="LOG_FILE_MAX_HISTORY" value="${log.file_max_history}"/>

    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <Pattern>${LOG_PATTERN}</Pattern>
        </encoder>
    </appender>

    <appender name="INFO" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>

        <file>${LOG_FILE_PATH}/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${LOG_FILE_NAME_INFO}</fileNamePattern>
            <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
        </rollingPolicy>

        <encoder>
            <Pattern>${LOG_PATTERN}</Pattern>
        </encoder>
    </appender>

    <appender name="ERROR" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>

        <file>${LOG_FILE_PATH}/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${LOG_FILE_NAME_ERROR}</fileNamePattern>
            <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
        </rollingPolicy>

        <encoder>
            <Pattern>${LOG_PATTERN}</Pattern>
        </encoder>
    </appender>

    <appender name="ACCESS" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="com.search.api.common.log.LogAccessFilter"/>

        <file>${LOG_FILE_PATH}/access/access.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/access/${LOG_FILE_NAME_ACCESS}</fileNamePattern>
            <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
        </rollingPolicy>

        <encoder>
            <Pattern>${LOG_PATTERN}</Pattern>
        </encoder>
    </appender>

    <logger name="com.search.api" level="info">
        <appender-ref ref="ERROR"/>
        <appender-ref ref="INFO"/>
    </logger>


    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ACCESS"/>
    </root>

</configuration>
```

<br/>

<br/>

출처

https://goddaehee.tistory.com/206

filter : https://ckddn9496.tistory.com/89
