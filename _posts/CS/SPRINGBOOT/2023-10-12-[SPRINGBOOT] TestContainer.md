---
title : "[SPRINGBOOT] TestContainer"
tags :
 - Springboot

---

# TestContainer

Testcontainers는 Docker를 기반으로 Junit 수행 시 테스트를 도와주는 Java Library

**도커 설치 필수!**

##### Testcontainers 특징

- Java로 Container를 동작시킬 수 있다.

- Dockerfile, docker-compose, docker hub로 Container를 동작시킬 수 있다.

- Container 환경을 자체적으로 구축하고 start, stop으로 **멱등성** 유지가 가능

  **멱등성**(idempotent)은 수학이나 전산학에서 연산의 한성질을 나타내는 것으로, 연산을 여러 번 적용하더라도 결과가 달라지지 않는 성질을 의미한다.

- Restarted Lifecycle를 이용해 독립적인 테스트 환경이 많아질수록 시간이 오래 걸릴 수도 있다.

<br/>

간단하게 말해서 테스트를 위한 환경을 도커 컨테이너로 띄워서 데이터 멱등성을 유지한다는 라이브러리다.

테스트를 위한 3개의 컨테이너를 구성했다. 테스트 컨테이너는 method 또는 class마다 restart가 되는데 시간이 너무 많이 걸려서 sigletone으로 구성을 했다.

싱글톤 참고 : https://java.testcontainers.org/test_framework_integration/manual_lifecycle_control/#singleton-containers

<br/>

**@DynamicPropertySource**

Spring Boot 애플리케이션에서 동적으로 환경 속성을 제어하기 위한 방법을 제공하는 인터페이스이다.

<br/>

### Elasticsearch

```java
public abstract class ElasticsearchContainerConfig {
    private static final String ELASTICSEARCH_IMAGE = "docker.elastic.co/elasticsearch/elasticsearch:7.17.4";
    public static final ElasticsearchContainer ELASTICSEARCH_CONTAINER;

    static {
        ELASTICSEARCH_CONTAINER = new ElasticsearchContainer(ELASTICSEARCH_IMAGE)
                .withAccessToHost(true)
                .withExposedPorts(9200)
                .withCreateContainerCmdModifier(cmd -> cmd.withName("test-elasticsearch"))
                .withReuse(true);

        ELASTICSEARCH_CONTAINER.start();

        Testcontainers.exposeHostPorts(ELASTICSEARCH_CONTAINER.getFirstMappedPort());
    }

    @DynamicPropertySource
    static void setElasticsearchContainerConfig(DynamicPropertyRegistry registry) {
        registry.add("elasticsearch.servers", () -> List.of(new ElasticsearchConfig.Config(ELASTICSEARCH_CONTAINER.getHost(), ELASTICSEARCH_CONTAINER.getFirstMappedPort(), "http")));
    }
}
```

<br/>

### Kafka

```java
public abstract class KafkaContainerConfig {

    private static final String KAFKA_IMAGE = "confluentinc/cp-kafka:7.3.2";
    private static final Network network = Network.newNetwork();
    public static final KafkaContainer KAFKA_CONTAINER;
    public static final GenericContainer KAFKA_UI;

    static {
        KAFKA_CONTAINER = new KafkaContainer(DockerImageName.parse(KAFKA_IMAGE))
                .withExposedPorts(9092, 9093)
                .withNetwork(network)
                .withNetworkAliases("test-kafka")
                .withAccessToHost(true)
                .withCreateContainerCmdModifier(cmd -> cmd.withName("test-kafka"))
                .withReuse(true);
        KAFKA_CONTAINER.start();

        Testcontainers.exposeHostPorts(KAFKA_CONTAINER.getMappedPort(9092));

        Map<String, String> kafkaUiEnv = new HashMap<>();
        kafkaUiEnv.put("KAFKA_CLUSTERS_0_BOOTSTRAPSERVERS", "PLAINTEXT://host.testcontainers.internal:" + KAFKA_CONTAINER.getMappedPort(9092));
        kafkaUiEnv.put("KAFKA_CLUSTERS_0_NAME", "test");

        KAFKA_UI = new GenericContainer<>(DockerImageName.parse("provectuslabs/kafka-ui:latest"))
                .withExposedPorts(8080)
                .withNetwork(network)
                .withNetworkAliases("test-kafka-ui")
                .withCreateContainerCmdModifier(cmd -> {
                    cmd.withName("test-kafka-ui");
                })
                .withReuse(true)
                .withEnv(kafkaUiEnv)
                .dependsOn(KAFKA_CONTAINER);
        KAFKA_UI.start();
    }

    @DynamicPropertySource
    static void setKafkaContainerConfig(DynamicPropertyRegistry registry) {
        registry.add("spring.kafka.bootstrap-servers", KAFKA_CONTAINER::getBootstrapServers);
    }
```

<br/>

#### Redis

```java
public abstract class RedisContainerConfig {
    public static final GenericContainer REDIS_CONTAINER;
    public static final RedisTemplate<String, String> REDIS_TEMPLATE;
    private static final String REDIS_IMAGE = "redis:6.0.9";

    static {
        REDIS_CONTAINER = new GenericContainer<>(DockerImageName.parse(REDIS_IMAGE))
                .withExposedPorts(6379)
                .withAccessToHost(true)
                .withNetworkAliases("test-redis")
                .withCreateContainerCmdModifier(cmd -> {
                    cmd.withName("test-redis");
                })
                .withReuse(true);
        REDIS_CONTAINER.start();

        Testcontainers.exposeHostPorts(REDIS_CONTAINER.getMappedPort(6379));

        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(REDIS_CONTAINER.getHost());
        redisStandaloneConfiguration.setPort(REDIS_CONTAINER.getFirstMappedPort());

        REDIS_TEMPLATE = new RedisTemplate<>();
        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisStandaloneConfiguration);
        lettuceConnectionFactory.afterPropertiesSet();
        REDIS_TEMPLATE.setConnectionFactory(lettuceConnectionFactory);
        REDIS_TEMPLATE.setKeySerializer(new StringRedisSerializer());
        REDIS_TEMPLATE.setValueSerializer(new StringRedisSerializer());
        REDIS_TEMPLATE.afterPropertiesSet();
    }
}
```

참고 : https://dealicious-inc.github.io/2022/01/10/test-containers.html
