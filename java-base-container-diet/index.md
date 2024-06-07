# java base docker image 경량화 시키기


## 서론

컨테이너의 운영환경에서는 어플리케이션 docker image가 작을 수록 빠르게 실행하고 확장할 수 있다.

특히, container orchestration 환경(추후)에서는 image 크기는 중요한데 Java의 경우 다른 언어에 비해 image가 매우 크다.

`운영환경에서 1G 이상의 docker container 크기를 가지고 사용한 경우를 많이 봐왔음..`

<!--more-->

## 단계별 Dockerfile

### 1. build

```dockerfile
##############################
# STEP 1 build
##############################
FROM gradle-jdk17 AS build
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY gradle.properties .
COPY settings.gradle .
COPY src src
RUN chmod +x ./gradlew
RUN ./gradlew bootJar
```

gradle 빌드 진행한다. 

### 2. jdeps, jlink를 사용하여 경량화한다. 

```dockerfile
##############################
# STEP 2 Analyze jar file and extract only what is needed
##############################
FROM corretto-jdk17 AS corretto-jdk
COPY --from=build /home/gradle/build/libs/*.jar /app/app.jar
RUN mkdir /app/unpacked && \
    cd /app/unpacked && \
    unzip ../app.jar && \
    cd .. && \
    $JAVA_HOME/bin/jdeps \
    --ignore-missing-deps \
    --print-module-deps \
    -q \
    --recursive \
    --multi-release 17 \
    --class-path="./unpacked/BOOT-INF/lib/*" \
    --module-path="./unpacked/BOOT-INF/lib/*" \
    ./app.jar > /deps.info
RUN apk add --no-cache binutils
# Build small JRE image
RUN $JAVA_HOME/bin/jlink \
    --verbose \ # 상세한 추적을 활성화 하여 로깅
    --add-modules $(cat /deps.info) \
    --strip-debug \ # 디버그 정보를 제거
    --no-man-pages \ # 리소스의 man page를 제거
    --no-header-files \
    --compress=2 \ # 리소스를 압축
    --output /customjre
```

1. jdeps를 사용해 자바 런타임 모듈 의존성 추출
2. jlink를 사용하여 사용자 정의 JRE 만들기

### 3. alpine base 이미지 사용하여 이미지 경량화

```dockerfile
##############################
# STEP 3 Build a small image
##############################
FROM alpine:3.18.2
ENV JAVA_HOME=/jre
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ENV WORKDIR=/app
COPY --from=corretto-jdk /customjre $JAVA_HOME
RUN mkdir /app
COPY --from=build /home/gradle/build/libs/*.jar $WORKDIR/app.jar
WORKDIR $WORKDIR
EXPOSE 8080
COPY distribution/docker/java_options.sh /app/java_options.sh
RUN chmod 755 $WORKDIR/java_options.sh
RUN mkdir -p $WORKDIR/plugin && wget -O $WORKDIR/plugin/dd-java-agent.jar 'https://dtdg.co/latest-java-tracer'
RUN sed -i 's/^#networkaddress\.cache\.ttl=-1/networkaddress\.cache\.ttl=10/' $JAVA_HOME/conf/security/java.security
ENTRYPOINT source /app/java_options.sh $ENV_PROFILE $ENV_DOCKER_TAG && /jre/bin/java $JAVA_OPTS -jar /app/app.jar $*
```

1. step1 단계의 jar파일, step2 단계의 JRE를 COPY
2. JAVA_OPTS을 위한 java_options.sh COPY
3. `networkaddress` cache ttl 변경을 위한 sed 명령어 수행
    * container내부 설정을 변경하기 위함, ttl뿐아니라 다른 것도 이런식으로 변경 
4. java_options.sh이라는 스크립트에 jvm 옵션들 넣어두고 환경변수를 가지고 jar실행하도록 함

{{< admonition tip "완성된 Dockerfile" false >}}
```dockerfile
FROM gradle:8.5.0-jdk17 AS gradle-jdk17
FROM amazoncorretto:17-alpine3.18-jdk AS corretto-jdk17
##############################
# STEP 1 build
##############################
FROM gradle-jdk17 AS build
COPY gradlew .
COPY gradle gradle
COPY build.gradle .
COPY gradle.properties .
COPY settings.gradle .
COPY src src
RUN chmod +x ./gradlew
RUN ./gradlew bootJar
##############################
# STEP 2 Analyze jar file and extract only what is needed
##############################
FROM corretto-jdk17 AS corretto-jdk
COPY --from=build /home/gradle/build/libs/*.jar /app/app.jar
RUN mkdir /app/unpacked && \
    cd /app/unpacked && \
    unzip ../app.jar && \
    cd .. && \
    $JAVA_HOME/bin/jdeps \
    --ignore-missing-deps \
    --print-module-deps \
    -q \
    --recursive \
    --multi-release 17 \
    --class-path="./unpacked/BOOT-INF/lib/*" \
    --module-path="./unpacked/BOOT-INF/lib/*" \
    ./app.jar > /deps.info
RUN apk add --no-cache binutils
# Build small JRE image
RUN $JAVA_HOME/bin/jlink \
    --verbose \
    --add-modules $(cat /deps.info) \
    --strip-debug \
    --no-man-pages \
    --no-header-files \
    --compress=2 \
    --output /customjre
##############################
# STEP 3 Build a small image
##############################
FROM alpine:3.18.2
ENV JAVA_HOME=/jre
ENV PATH="${JAVA_HOME}/bin:${PATH}"
ENV WORKDIR=/app
COPY --from=corretto-jdk /customjre $JAVA_HOME
RUN mkdir /app
COPY --from=build /home/gradle/build/libs/*.jar $WORKDIR/app.jar
WORKDIR $WORKDIR
EXPOSE 8080
COPY distribution/docker/java_options.sh /app/java_options.sh
RUN chmod 755 $WORKDIR/java_options.sh
RUN mkdir -p $WORKDIR/plugin && wget -O $WORKDIR/plugin/dd-java-agent.jar 'https://dtdg.co/latest-java-tracer'
RUN sed -i 's/^#networkaddress\.cache\.ttl=-1/networkaddress\.cache\.ttl=10/' $JAVA_HOME/conf/security/java.security
ENTRYPOINT source /app/java_options.sh $ENV_PROFILE $ENV_DOCKER_TAG && /jre/bin/java $JAVA_OPTS -jar /app/app.jar $*
```
{{< /admonition >}}

## 결과

기존 알려진 dockerfile들 대비 20% 크기인 200MB도 안되게 경량화 되어 운영이 가능하다.

---

## 참고
* https://aws.amazon.com/ko/blogs/tech/amazon-corretto-base-container-diet/
