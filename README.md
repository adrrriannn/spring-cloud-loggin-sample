# spring-cloud-logging-sample

This application sample purpose is show this application freezes on startup when there is a problem to flush logs to a 
remote server and `spring-cloud-context` is in the classpath.

For this example, [FluencyLogbackAppender](https://github.com/sndyuk/logback-more-appenders/blob/master/src/main/java/ch/qos/logback/more/appenders/FluencyLogbackAppender.java) has been used.
This appender uses [Fluency](https://github.com/komamitsu/fluency) library under the hood to send logs via TCP to a remote server.

In order to reproduce this problem, a **fake** remote server has been set in the `logback-spring.xml` configuration file:
```xml
<appender name="FLUENCY" class="ch.qos.logback.more.appenders.FluencyLogbackAppender">
  <tag>debug</tag>
  <remoteHost>fake.host</remoteHost>
  <port>24224</port>
</appender>
```

## Usage

### Build an executable jar:
```sh
./gradlew clean build -x test
```
Tests are skipped in order to make faster the build process.

### Run the app
```sh
  java -jar build/libs/spring-cloud-logging-sample-0.0.1-SNAPSHOT.jar
```

By following the previous steps, we should see the startup freezes for some time:
```sh
 ~/p/spring-cloud-logging-sample> java -jar build/libs/spring-cloud-logging-sample-0.0.1-SNAPSHOT.jar                                                                                                            Mon Sep 30 11:42:42 2019
2019-09-30 11:42:45.570  INFO [,,,] 1422 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$bef36ca5] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
```

However, if the following code block is uncommented in the `build.gradle` file to exclude `spring-cloud-context` dependency, 
the application continues startup normally. It does print error logs, of course, for not being able to communicate 
to the fake remote server, but this is expected, and desirable, behaviour.

- Build.gradle
```gradle
	{
		exclude group: 'org.springframework.cloud', module: 'spring-cloud-context'
	}
```

- Startup log
```sh
 ~/p/spring-cloud-logging-sample> java -jar build/libs/spring-cloud-logging-sample-0.0.1-SNAPSHOT.jar                                                                                                 1542ms  Mon Sep 30 11:38:51 2019

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.1.8.RELEASE)

2019-09-30 11:38:54.061  INFO [,,,] 1368 --- [           main] c.a.s.SampleAppApplication               : Starting SampleAppApplication on ip-192-168-10-13.eu-west-1.compute.internal with PID 1368 (/Users/adrian/projects/spring-cloud-logging-sample/build/libs/spring-cloud-logging-sample-0.0.1-SNAPSHOT.jar started by adrian in /Users/adrian/projects/spring-cloud-logging-sample)
2019-09-30 11:38:54.093  INFO [,,,] 1368 --- [           main] c.a.s.SampleAppApplication               : No active profile set, falling back to default profiles: default
2019-09-30 11:38:54.602 ERROR [,,,] 1368 --- [pool-2-thread-1] o.k.f.f.ingester.sender.NetworkSender    : Failed to send 929 bytes data
2019-09-30 11:38:54.605 ERROR [,,,] 1368 --- [pool-2-thread-1] o.k.f.f.ingester.sender.MultiSender      : Failed to send: sender=TCPSender{config=Config{host='fake.host', port=24224, connectionTimeoutMilli=5000, readTimeoutMilli=5000, waitBeforeCloseMilli=1000} Config{senderErrorHandler=null}} NetworkSender{config=Config{host='fake.host', port=24224, connectionTimeoutMilli=5000, readTimeoutMilli=5000, waitBeforeCloseMilli=1000} Config{senderErrorHandler=null}, failureDetector=FailureDetector{failureDetectStrategy=PhiAccrualFailureDetectStrategy{failureDetector=org.komamitsu.failuredetector.PhiAccuralFailureDetector@4f3e0bc} org.komamitsu.fluency.fluentd.ingester.sender.failuredetect.PhiAccrualFailureDetectStrategy@41adfde0, heartbeater=TCPHeartbeater{config=Config{host='fake.host', port=24224, intervalMillis=1000}} Heartbeater{config=Config{host='fake.host', port=24224, intervalMillis=1000}}, lastFailureTimestampMillis=1569836334604, config=Config{failureIntervalMillis=3000}}} org.komamitsu.fluency.fluentd.ingester.sender.TCPSender@5eb2e86f. Trying to use next sender...

java.net.UnknownHostException: null
	at sun.nio.ch.Net.translateException(Net.java:155)
	at sun.nio.ch.SocketAdaptor.connect(SocketAdaptor.java:127)
	at org.komamitsu.fluency.fluentd.ingester.sender.TCPSender.getOrCreateSocketInternal(TCPSender.java:66)
	at org.komamitsu.fluency.fluentd.ingester.sender.TCPSender.getOrCreateSocketInternal(TCPSender.java:31)
	at org.komamitsu.fluency.fluentd.ingester.sender.NetworkSender.getOrCreateSocket(NetworkSender.java:76)
	at org.komamitsu.fluency.fluentd.ingester.sender.NetworkSender.sendInternal(NetworkSender.java:101)
	at org.komamitsu.fluency.fluentd.ingester.sender.FluentdSender.sendInternalWithRestoreBufferPositions(FluentdSender.java:74)
	at org.komamitsu.fluency.fluentd.ingester.sender.FluentdSender.send(FluentdSender.java:56)
	at org.komamitsu.fluency.fluentd.ingester.sender.MultiSender.sendInternal(MultiSender.java:60)
	at org.komamitsu.fluency.fluentd.ingester.sender.FluentdSender.sendInternalWithRestoreBufferPositions(FluentdSender.java:74)
	at org.komamitsu.fluency.fluentd.ingester.sender.FluentdSender.send(FluentdSender.java:56)
	at org.komamitsu.fluency.fluentd.ingester.sender.RetryableSender.sendInternal(RetryableSender.java:77)
	at org.komamitsu.fluency.fluentd.ingester.sender.FluentdSender.sendInternalWithRestoreBufferPositions(FluentdSender.java:74)
	at org.komamitsu.fluency.fluentd.ingester.sender.FluentdSender.send(FluentdSender.java:56)
	at org.komamitsu.fluency.fluentd.ingester.FluentdIngester.ingest(FluentdIngester.java:87)
	at org.komamitsu.fluency.buffer.Buffer.flushInternal(Buffer.java:358)
	at org.komamitsu.fluency.buffer.Buffer.flush(Buffer.java:112)
	at org.komamitsu.fluency.flusher.Flusher.runLoop(Flusher.java:62)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)

2019-09-30 11:38:54.607  WARN [,,,] 1368 --- [pool-2-thread-1] o.k.f.f.ingester.sender.RetryableSender  : Sender failed to send data. sender=RetryableSender{baseSender=MultiSender{senders=[TCPSender{config=Config{host='fake.host', port=24224, connectionTimeoutMilli=5000, readTimeoutMilli=5000, waitBeforeCloseMilli=1000} Config{senderErrorHandler=null}} NetworkSender{config=Config{host='fake.host', port=24224, connectionTimeoutMilli=5000, readTimeoutMilli=5000, waitBeforeCloseMilli=1000} Config{senderErrorHandler=null}, failureDetector=FailureDetector{failureDetectStrategy=PhiAccrualFailureDetectStrategy{failureDetector=org.komamitsu.failuredetector.PhiAccuralFailureDetector@4f3e0bc} org.komamitsu.fluency.fluentd.ingester.sender.failuredetect.PhiAccrualFailureDetectStrategy@41adfde0, heartbeater=TCPHeartbeater{config=Config{host='fake.host', port=24224, intervalMillis=1000}} Heartbeater{config=Config{host='fake.host', port=24224, intervalMillis=1000}}, lastFailureTimestampMillis=1569836334604, config=Config{failureIntervalMillis=3000}}} org.komamitsu.fluency.fluentd.ingester.sender.TCPSender@5eb2e86f]} org.komamitsu.fluency.fluentd.ingester.sender.MultiSender@46664821, retryStrategy=ExponentialBackOffRetryStrategy{config=Config{baseIntervalMillis=400, maxIntervalMillis=30000} Config{maxRetryCount=7}} RetryStrategy{config=Config{baseIntervalMillis=400, maxIntervalMillis=30000} Config{maxRetryCount=7}}, isClosed=false} org.komamitsu.fluency.fluentd.ingester.sender.RetryableSender@5d0bb651, retry=0
```


