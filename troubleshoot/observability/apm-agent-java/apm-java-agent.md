---
navigation_title: APM Java Agent
mapped_pages:
  - https://www.elastic.co/guide/en/apm/agent/java/current/trouble-shooting.html
applies_to:
  stack: all
  serverless:
    observability: all
products:
  - id: apm-agent
---

# Troubleshoot APM Java Agent [trouble-shooting]

Something isn’t quite working as expected? Here are some guidelines how to find out what’s going wrong.

As a first step, check if your stack is compatible with the currently [supported technologies](apm-agent-java://reference/set-up-apm-java-agent.md#supported-technologies).

Don’t worry if you can’t figure out what the problem is. Open a topic in the [APM discuss forum](https://discuss.elastic.co/c/apm) and we will help you out.

::::{important}
If you do so, **attach your debug logs** so that we can analyze the problem. Upload the **complete** logs to a service like [https://gist.github.com](https://gist.github.com). The logs should include everything from the application startup up until the first request has been executed. In addition to agent and application logs, look for `[elastic-apm-agent]` entries in all of your service’s std out and std error logs, where we sometimes print useful information when logging is unavailable.
::::



## Updating to latest agent version [trouble-shooting-use-latest-agent]

Agent is updated frequently and releases are not strongly tied to other components in the stack.

Therefore, trying to update to the most recently released agent version is often the recommended first troubleshooting step. If possible, trying the [latest-snapshot](https://github.com/elastic/apm-agent-java/blob/main/README.md#snapshots) is even preferable as it would include fixes that are not yet released.

See [upgrading documentation](apm-agent-java://reference/upgrading.md) for more details.


## Running alongside other agents [trouble-shooting-additional-agent]

Like many other Java agents, our agent instruments classes by injecting bytecode into them on runtime. Our bytecode instrumentation guarantees to produce only valid bytecode and to never change the class’s "schema". If you are using other Java agents in addition to ours that have the same guarantees, they should not interfere with each other. However, some Java agents do not conform to these guarantees, in which case there may be issues with using them in parallel to ours. If you encounter errors, one of the first things to do is to remove any additional agent in order to check whether this is indeed a case of agent mismatch. We may still find a way to make them work together, but this information would be crucial for our ability to assist, so make sure to include it when reporting such issues.


## Logging [trouble-shooting-logging]

There are several [logging related configuration options](apm-agent-java://reference/config-logging.md). The most important one is [`log_level`](apm-agent-java://reference/config-logging.md#config-log-level).

Set the log level to `DEBUG` or even `TRACE` to get more information about the behavior of the agent.

* `DEBUG` shows:

    * Agent configuration as read by the agent
    * Transaction and spans creation, activation and deactivation events

* `TRACE` is even more verbose than `DEBUG`:

    * A stack trace is printed every time a transaction or span is activated or deactivated
    * All data sent to apm-server is included in JSON format


Always post the whole content of your log files when asking for help. Use the [procedure](#trouble-shooting-logging-procedure) to ensure consistent logs when reporting potential issues.

When the agent starts up, you should see logs similar to these:

```
[main] INFO co.elastic.apm.agent.configuration.StartupInfo - Starting Elastic APM (unknown version) on Java 10 (Oracle Corporation) Mac OS X 10.13.6
[apm-server-healthcheck] INFO co.elastic.apm.agent.report.ApmServerHealthChecker - Elastic APM server is available: {"build_date":"2018-11-05T07:58:08Z","build_sha":"dffb98a72a262ca22adad0152f0245ea743ea904","version":"7.0.0-alpha1"}
[main] DEBUG co.elastic.apm.agent.configuration.StartupInfo - service_name: 'elastic-apm-test' (source: Java System Properties)
[main] DEBUG co.elastic.apm.agent.configuration.StartupInfo - log_level: 'DEBUG' (source: Java System Properties)
```

Make sure to execute some requests to your application before posting your log files. Each request should at least add some lines similar to these in the logs:

```
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - startTransaction '' 00-2a82cbe3df7a0208f7be6da65be260d1-05e72d045206587a-01 {
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - Activating '' 00-2a82cbe3df7a0208f7be6da65be260d1-05e72d045206587a-01 on thread 66
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.transaction.SpanImpl - startSpan '' 00-2a82cbe3df7a0208f7be6da65be260d1-b2ffa0401105e3d8-01 {
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - Activating 'APIRestController#products' 00-2a82cbe3df7a0208f7be6da65be260d1-b2ffa0401105e3d8-01 on thread 66
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.transaction.SpanImpl - startSpan '' 00-2a82cbe3df7a0208f7be6da65be260d1-49b9d805eca42ec6-01 {
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - Activating '' 00-2a82cbe3df7a0208f7be6da65be260d1-49b9d805eca42ec6-01 on thread 66
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - Deactivating 'SELECT' 00-2a82cbe3df7a0208f7be6da65be260d1-49b9d805eca42ec6-01 on thread 66
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.transaction.SpanImpl - endSpan 'SELECT' 00-2a82cbe3df7a0208f7be6da65be260d1-49b9d805eca42ec6-01
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Receiving SPAN event (sequence 23)
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Starting new request to http://127.0.0.1:8200/intake/v2/events
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - Deactivating 'APIRestController#products' 00-2a82cbe3df7a0208f7be6da65be260d1-b2ffa0401105e3d8-01 on thread 66
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.transaction.SpanImpl - endSpan 'APIRestController#products' 00-2a82cbe3df7a0208f7be6da65be260d1-b2ffa0401105e3d8-01
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Scheduling request timeout in 10s
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Receiving SPAN event (sequence 24)
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - Deactivating 'APIRestController#products' 00-2a82cbe3df7a0208f7be6da65be260d1-05e72d045206587a-01 on thread 66
[http-nio-8080-exec-10] DEBUG co.elastic.apm.agent.impl.ElasticApmTracer - endTransaction 'APIRestController#products' 00-2a82cbe3df7a0208f7be6da65be260d1-05e72d045206587a-01
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Receiving TRANSACTION event (sequence 25)
```

If you don’t see anything in your logs, the technology stack you are using is probably not [supported](apm-agent-java://reference/supported-technologies.md).

After that, you should see logs indicating that the agent has successfully sent data to the APM server:

```
[apm-request-timeout-timer] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Request flush because the request timeout occurred
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Receiving FLUSH event (sequence 26)
[apm-reporter] DEBUG co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Flushing 10912 uncompressed 2667 compressed bytes
```

If the APM server responds with a 400, it could indicate JSON validation errors. The log would then contain the actual documents which failed to validate:

```
[apm-reporter] INFO co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Backing off for 0 seconds (±10%)
[apm-reporter] WARN co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - Server returned HTTP response code: 400 for URL: http://127.0.0.1:8200/intake/v2/events
[apm-reporter] WARN co.elastic.apm.agent.report.IntakeV2ReportingEventHandler - {"accepted":13,"errors":[{"message":"Problem validating JSON document against schema: I[#] S[#] doesn't validate with \"span#\"\n  I[#] S[#/allOf/2] allOf failed\n    I[#] S[#/allOf/2/required] missing properties: \"transaction_id\"","document":"{\"span\":{\"name\":\"OpenTracing product span\",\"timestamp\":29352159207,\"id\":\"aeaa7e0ac95acad6\",\"trace_id\":\"d88b5cbfc4536f9a700cd114a53bfeae\",\"parent_id\":\"082fd71ce7e4089a\",\"duration\":17.992,\"context\":{\"tags\":{\"productId\":\"1\"}},\"type\":\"unknown\"}}"}]}
```


### Capturing logs procedure [trouble-shooting-logging-procedure]

Ideally this should be done outside of production.

* It requires restarting the application
* With log level set to `DEBUG` logs can be verbose, with `TRACE` they are even more verbose.
* Having only a few transactions in the log makes investigations easier, production traffic creates noise.
* If not possible, we can still use it for a few minutes in production, when using an external configuration file ([doc](apm-agent-java://reference/configuration.md)) the `log_level` can be changed at runtime.

Most investigations require using `DEBUG` log level, thus use `DEBUG` unless asked for `TRACE`.

Here, we will refer to the agent log file as `/tmp/agent.log`, but any other location could be used.

1. configure the agent with `log_level=debug` or `log_level=trace` ([doc](apm-agent-java://reference/config-logging.md#config-log-level)) and `log_file=/tmp/agent.log` ([doc](apm-agent-java://reference/config-logging.md))
2. truncate the `/tmp/agent.log` file and restart the application
3. execute a few transactions which aren’t properly captured by the agent
4. copy the `/tmp/agent.log` file and send it back for investigation


## Agent matching heuristics [trouble-shooting-matching-heuristics]

The agent relies on heuristics to define which classes needs to be instrumented or not efficiently to prevent instrumentation overhead.

Those heuristics are based on the packages and class names and are influenced by `application_packages` and `jms_listener_packages` configurations. However, if instrumentation is not applied as expected or if you want to investigate creating configurations without proper knowledge of the application internals, it could be relevant to disable those heuristics for investigation:

1. disable name heuristics by setting `enable_type_matching_name_pre_filtering=false` and enable the [agent logs](#trouble-shooting-logging-procedure).
2. restart the application, which will be slower than usual due to the extra overhead
3. analyze the agent logs to identify which classes/methods are instrumented by filtering lines with `Method match` string.
4. properly configure `application_packages` or `jms_listener_packages` with values that apply


## Debugging [trouble-shooting-debugging]

Sometimes reading the logs is just not enough to debug a problem. As the agent is OpenSource and released on Maven Central, debugging the agent code is really easy.

In order for your IDE to download the sources, first declare a dependency to the agent.

::::{note}
The agent is added via the `-javaagent` flag. So you have to make sure that you declare the dependency in a way that the agent is not on the classpath twice. For example, when you are developing a web application, make sure that the agent is not packaged in your war in `WEB-INF/lib`. In the debug window of your IDE, make sure that `apm-agent` does not appear in the `-classpath`.
::::


```xml
<dependency>
    <groupId>co.elastic.apm</groupId>
    <artifactId>apm-agent</artifactId>
    <version>${elastic-apm.version}</version>
    <scope>provided</scope>
</dependency>
```

::::{note}
Even when setting the `scope` to `provided`, IntelliJ sometimes adds the agent on the classpath anyway. One workaround is to set the `scope` to `test`.
::::


```groovy
compileOnly "co.elastic.apm:apm-agent:$elasticApmVersion"
```

::::{note}
In versions prior to 1.26.0, you need to declare a dependency on the `elastic-apm-agent` module instead of `apm-agent`
::::



## Common problems [trouble-shooting-common-issues]


### There is no data in the Kibana APM app [trouble-shooting-no-data]

The most common source of this problem are connection issues between the agent and the APM server.

If the APM server does not receive data from the agent, check if the agent is able to establish a connection to the server. In the agent logs, look out for logs containing `Elastic APM server is available` and `Elastic APM server is not available`.

If you see the message `Elastic APM server is not available`, the agent has problems connecting to the APM server. Check the setting of [`server_url`](apm-agent-java://reference/config-reporter.md#config-server-url) and make sure the agent is able to connect to the server. Try to execute `curl -v <apm-server-url>` from the machine the agent is running on. The server should respond with a 200 status code.

If the APM server does not respond successfully, have a look at the APM server logs to verify that the server is actually running. Also make sure to configure your firewalls so that the host the agent runs on can open HTTP connections to the APM server.


### Kibana APM app shows "unknown route" [trouble-shooting-unknown-route]

By default, transactions are named with the Servlet name that handled the request. Thus, if request does not reach a servlet, the Agent defaults to naming the transaction "unknown route"

There are two reasons why this might happen:

1. Requests reach a servlet, but the Agent is not properly able to detect them.
2. Requests do not reach a servlet. It might’ve been handled by filter, static resources, etc.

    Requests reach a servlet
    :   The Agent has a *pre-filter* heuristic to only consider classes whose names end with *Servlet*. This heuristic can be disabled by setting the internal configuration `enable_type_matching_name_pre_filtering=false`.

        Note that this has an impact on all plugins. A small increase of overhead during application startup time is expected.


    Requests do not reach a servlet
    :   It’s possible to change the default transaction naming to use the URL path instead. See [`use_path_as_transaction_name` ([1.0.0])](apm-agent-java://reference/config-http.md#config-use-path-as-transaction-name) for more information.

        Unfortunately, this may create a lot of duplicate transactions if they have similar paths. For example, in `/usr/{{id}}`, where `{{id}}` is the user ID, you can end up with as many transactions as there are users. You can mitigate this by using [`url_groups` (deprecated)](apm-agent-java://reference/config-http.md#config-url-groups), which will allow the use of wildcards in transaction URLs.


If the proposed fixes do not solve the problem, or if a custom name is required, transaction names can be set manually throughout the request handling flow using our API:

* [`Transaction currentTransaction()`](apm-agent-java://reference/public-api.md#api-current-transaction) gets the current transaction.
* [`Transaction setName(String name)`](apm-agent-java://reference/public-api.md#api-set-name) sets the transaction name.


### Libraries compiled against old Java versions [trouble-shooting-old-jdbc-drivers]

If you are seeing warning like these in your application, it means that you are using a library which has been compiled for a very old version of Java:

```
org.apache.commons.dbcp.DelegatingStatement uses an unsupported class file version (pre Java 5) and can't be instrumented.
Consider updating to a newer version of that library.
```

That mostly concerns JDBC drivers. Updating them to a more recent version should resolve the problem.


### Failed to find Premain-Class manifest attribute [trouble-shooting-incorrect-manual-jar-file]

If you are using a manual setup with a `-javaagent` flag against an application server and are seeing the `Failed to find Premain-Class manifest attribute` error and a failure to start, then you might be pointing at the incorrect jar file.

The correct jar file to be pointing at should be in the form of `elastic-apm-agent-<version>.jar` and further information about how to download this file can be found [in the manual setup instructions.](apm-agent-java://reference/setup-javaagent.md)


### Communication with APM Server [trouble-shooting-communication]

`unable to find valid certification path to requested target` - server authentication fails. Check out [APM Server certificate authentication](apm-agent-java://reference/ssl-configuration.md#ssl-server-authentication).

`java.net.SocketException: Broken pipe` - one option is that client authentication fails. Check out [Agent certificate authentication](apm-agent-java://reference/ssl-configuration.md#ssl-client-authentication).

With Oracle Weblogic application server wildcard TLS certificates are not allowed by default, when this happens a message error like this is visible in agent logs: `Hostname verification failed: HostnameVerifier=weblogic.security.utils.SSLWLSHostnameVerifier`. Disabling this extra check can be done with `-Dweblogic.security.SSL.ignoreHostnameVerification=true`.

For other SSL/TLS related problems, - check out [the JSSE troubleshooting section](https://docs.oracle.com/javase/8/docs/technotes/guides/security/jsse/JSSERefGuide.md#Troubleshooting). You can add `-Djavax.net.debug=all` to the JVM cmd line to get more details about your problem.


## Uncommon problems [trouble-shooting-uncommon-issues]


### JVM Crashes [trouble-shooting-jvm-crashes]

More often than not, JVM crashes indicate a JVM bug being surfaced by the installation of the Java agent within the specific configuration of the traced application and it’s dependencies. Therefore, the first thing to try is upgrade the JVM to the latest minor version.

Known issues:

* Early Java 8 versions before update 40 are **not supported** because they have several bugs that might result in JVM crashes when a java agent is active, thus agent **will not start** on those versions.
* Similarly, Java 7 versions before update 60 are not supported as they are buggy in regard to `invokedynamic`.
* Later Java 7 versions (> update 60) and early Java 8 versions (< update 40) are known to crash with agent versions 1.18.0-1.20.0 at some random point after (sometimes long after) startup due to [a bug](https://bugs.openjdk.java.net/browse/JDK-8041920) causing the creation of faulty native code by the C2 compiler. Symptoms of such crashes are non-deterministic. In order to prevent such crashes, we added a built-in delay for agent initialization in agent version 1.21.0, that will be automatically applied on these Java versions. If crashes still occur with agent version > 1.20.0, try one of the followings:

    1. Add `-XX:CompileCommand=exclude,java.lang.invoke.LambdaForm*::*` to the command line to avoid the problematic JIT compilation
    2. Increase the delay from the default (3000ms) by setting the `elastic.apm.delay_agent_premain_ms` System property to indicate the number of milliseconds to delay, through the command line, for example: `-Delastic.apm.delay_agent_premain_ms=10000`.

* When [`profiling_inferred_spans_enabled` ([1.15.0] experimental)](apm-agent-java://reference/config-profiling.md#config-profiling-inferred-spans-enabled) is set to `true`, it uses a native library that collects low-level information from the JVM. All known issues so far had been fixed. Try to disable it if you think the crash may be related. We continuously upgrade to the latest async profiler version, so upgrading your agent to the latest version may already contain a fix.

Whenever you encounter a JVM crash, report through [our forum](https://discuss.elastic.co/c/observability/apm/58) or by opening an issue on our [GitHub repository](https://github.com/elastic/apm-agent-java). Look for the crash log (e.g. an `hs_err_pid<PID>.log`) and provide it when reporting, as well as all factors describing you setup and scenario.


### JVM Hangs [trouble-shooting-jvm-hangs]

If your JVM gets hang when attaching the Java agent, create a thread dump (e.g. through `jstack`) and report through [our forum](https://discuss.elastic.co/c/observability/apm/58) or by opening an issue on our [GitHub repository](https://github.com/elastic/apm-agent-java).


### Custom Java runtimes using `jlink` [trouble-shooting-jlink]

If you use `jlink` to create a custom runtime, make sure the following modules are added: `--add-modules java.base,java.logging,java,jdk.zipfs,java.management,jdk.management`


## Disable the Agent [disable-agent]

In the unlikely event the agent causes disruptions to a production application, you can disable the agent while you troubleshoot.

Using [dynamic configuration](apm-agent-java://reference/configuration.md#configuration-dynamic), you can disable the recording of events by setting [`recording`](apm-agent-java://reference/config-core.md#config-recording) to `false`.

If that doesn’t work, you can completely disable the agent by setting [`enabled`](apm-agent-java://reference/config-core.md#config-enabled) to `false`. You’ll need to restart your application for this change to take effect.


## Unsupported framework versions [trouble-shooting-unsupported-framework-versions]

* JSF - myfaces some 2.2.x versions are not supported on JDK 15 - see [related bug](https://github.com/raphw/byte-buddy/issues/979).

