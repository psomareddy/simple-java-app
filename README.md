This is a really simple sample java app used to demonstrate the ability to add custom tags to spans delivered through opentelemetry auto instrumentation extension.

## Running simple-java-app locally
simple-java-app is a java application built using gradle. You can build a jar file and run it from the command line (it should work just as well with Java 11 or newer):


```
git clone https://github.com/psomareddy/simple-java-app.git
cd simple-java-app
./gradlew clean build
java -jar build/libs/*.jar
```

## Instrument the app with OpenTelemetry Java Auto Instrumentation

Check out the documentation for [Instrumenting a Java application for Splunk Observability Cloud](https://docs.splunk.com/Observability/gdi/get-data-in/application/java/instrumentation/instrument-java-application.html#nav-Instrument-a-Java-application)

The gist of it for a simple app is to follow these few commands below

0. Make sure the Splunk Distribution of opentelemetry collector is installed. If not, you can just install it or start one up with docker

```
docker run --rm -e SPLUNK_ACCESS_TOKEN=12345 -e SPLUNK_REALM=us0 \
    -p 13133:13133 -p 14250:14250 -p 14268:14268 -p 4317:4317 -p 6060:6060 \
    -p 7276:7276 -p 8888:8888 -p 9080:9080 -p 9411:9411 -p 9943:9943 \
    --name otelcol quay.io/signalfx/splunk-otel-collector:latest
```

1. Download the Splunk Distribution of opentelemetry Java agent

```
curl -L https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent.jar \
-o splunk-otel-javaagent.jar
```

2. Set the OTEL_SERVICE_NAME and OTEL_RESOURCE_ATTRIBUTES environment variables

```
OTEL_SERVICE_NAME=simple-java-app
OTEL_RESOURCE_ATTRIBUTES=deployment.environment=simple-env
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317
```

3. Set the -javaagent argument to the path of the Java agent. I also include the otel.instrumentation.methods.include directive to instrument our getGreeting method in the App class.

```
java  -javaagent:/opt/otel/splunk-otel-javaagent.jar  -Dotel.instrumentation.methods.include=com.simple.app.App[getGreeting] -jar build/libs/*.jar 
```


## Usage
Once started, the application should run for about 3 minutes invoking the App.getGreeting() every two seconds method which prints "Hello World! {iteration}". The instrumentation will pick up on these invocations and create a span for each exectution.




