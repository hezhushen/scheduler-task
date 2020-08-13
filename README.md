# scheduler-task
# 接口文档v1

date:2020/6/24

文档涉及azkaban 提交任务及yarn 提交任务2种模式，远程执行第三方waterdrop 大数据开源集成框架，启动spark.sh及flink.sh 

waterdrop地址：https://github.com/InterestingLab/waterdrop

如果不是自行开发任务调度平台，建议使用dolphin scheduler

## token 认证

1. token 生成规则

   密文类型： fssFSS

   字段：redis中存储该用户信息的登录的key 

2. eg:     

   ```
   {"token":"eyJhbGciOiJIUzI1NiJ9.eyJqdGkiOiI3MmU3NzhhYi0wZWRjLTRlNTAtOWFjZS1kNzQwMTIxMzA5M2UiLCJpYXQiOjE1OTI1NDMzMTgsImlzcyI6IndkanMiLCJzdWIiOiJ6aGFuZ3NhbiIsImV4cCI6MTU5MjU0NjkxOH0.K1IN4BwJKQjwu0uZ9cAN68TVmQist8N4z8FGBdk-y5w"}
   ```

   

### Azkaban接口

```java
public class Azkaban {
        //登录id
        private String sessionId;
        //yarn 任务id
        private String applicationId;
        //azkanban 执行id
        private String execId;
        //azkanban 用户
        private String username;
        //azkanban 密码
        private String password;
        //azkanban 工程
        private String project;
        //azkanban 流程
        private String flow;
        //azkanban 任务id
        private String jobId;
        //azkanban 偏移量
        private int offset;
        //azkanban 长度
        private int length;
}
```

*Azkaban url: https://192.168.2.13:8443    user:admin ,password:admin*

1. 登录azkaban 

   - url

     *http://localhost:8080/azkaban/login*

   - 请求方式：

     post

   - body:

     ```
     {
         "username":"admin",
         "password":"admin"
     }
     ```

     

   - response:

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\n  \"session.id\" : \"082acbdb-34d2-40a8-b1dd-748dcecf7193\",\n                        \"status\"    : \"success\"\n
                    }"
     }
     ```

     

   - error:

     ```
     {
         "statusCode": 154,
         "message": "登录azkaban失败",
         "data": "{\"error\":\"Incorrect Login. Username/Password not found.\"}"
     }
     ```

2. 启动任务

   - 接口

     *http://localhost:8080/azkaban/start*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "project":"test",
         "flow":"test"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\n  \"project\" : \"test\",\n  
                  \"message\" : \"Execution submitted          successfully with exec                id 9\",\n  
                  \"flow\" : \"test\",\n  
                  \"execid\" : 9\n}"
     }
     
     ```

     

   - error

     ```
     {
         "statusCode": 150,
         "message": "启动任务失败",
         "data": "{\"project\":\"test\",\"error\":\"Flow 'tes' cannot be found in project azkaban.project.Project@224c6745\",\"flow\":\"tes\"}"
     }
     ```

3. 获取applicationId

   - url

     *http://localhost:8080/azkaban/getAppId*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "execId":"9",
         "jobId":"test"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": “application_1592537240488_0075”
     }
     ```

     

   - error

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": null   //启动任务后请及时获取applicationId ,azkanban日志会覆盖
     }
     ```

     

4. 获取任务

   - url

     *http://localhost:8080/azkaban/logger*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "execId":"9",
         "jobId":"test",
         "offest":0,
         "length":"1000"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\n  \"offset\" : 0,\n  \"data\" : \"24-06-2020 16:11:19 CST test ERROR - \\tat java.lang.Thread.run(Thread.java:748)\\n24-06-2020 16:11:19 CST test ERROR - \\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO scheduler.JobScheduler: Added jobs for time 1592986280000 ms\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO scheduler.JobScheduler: Starting job streaming job 1592986280000 ms.0 from job set of time 1592986280000 ms\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO spark.ContextCleaner: Cleaned accumulator 1353\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO spark.ContextCleaner: Cleaned accumulator 1351\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO spark.ContextCleaner: Cleaned accumulator 1354\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO spark.ContextCleaner: Cleaned accumulator 1355\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO spark.ContextCleaner: Cleaned accumulator 1352\\n24-06-2020 16:11:20 CST test ERROR - 20/06/24 16:11:20 INFO\",\n  \"length\" : 1000\n}"
     }
     ```

     

   - error

     ```
     {
         "statusCode": 152,
         "message": "获取日志失败",
         "data": "{\"error\":\"Job tes doesn't exist in 9\"}"
     }
     ```

5. 获取当前运行的任务

   - url

     *http://localhost:8080/azkaban/status*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "execId":"9"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\n  \"project\" : \"test\",\n  \"updateTime\" : 1592983043907,\n  \"type\" : null,\n  \"attempt\" : 0,\n  \"execid\" : 9,\n  \"submitTime\" : 1592983042595,\n  \"nodes\" : [ {\n    \"nestedId\" : \"test\",\n    \"startTime\" : 1592983043847,\n    \"updateTime\" : 1592983043897,\n    \"id\" : \"test\",\n    \"endTime\" : -1,\n    \"type\" : \"command\",\n    \"attempt\" : 0,\n    \"status\" : \"RUNNING\"\n  } ],\n  \"nestedId\" : \"test\",\n  \"submitUser\" : \"admin\",\n  \"startTime\" : 1592983043674,\n  \"id\" : \"test\",\n  \"endTime\" : -1,\n  \"projectId\" : 2,\n  \"flowId\" : \"test\",\n  \"flow\" : \"test\",\n  \"status\" : \"RUNNING\"\n}"
     }
     ```

     

   - error

     ```
     {
         "statusCode": 153,
         "message": "获取状态失败",
         "data": "<html>\n<head>\n<meta http-equiv=\"Content-Type\" content=\"text/html; charset=ISO-8859-1\"/>\n<title>Error 500 Index: 0</title>\n</head>\n<body><h2>HTTP ERROR 500</h2>\n<p>Problem accessing /executor. Reason:\n<pre>    Index: 0</pre></p><h3>Caused by:</h3><pre>java.lang.IndexOutOfBoundsException: Index: 0\n\tat java.util.Collections$EmptyList.get(Collections.java:4454)\n\tat azkaban.executor.JdbcExecutorLoader.fetchExecutableFlow(JdbcExecutorLoader.java:188)\n\tat azkaban.executor.ExecutorManager.getExecutableFlow(ExecutorManager.java:174)\n\tat azkaban.webapp.servlet.ExecutorServlet.handleAJAXAction(ExecutorServlet.java:100)\n\tat azkaban.webapp.servlet.ExecutorServlet.handlePost(ExecutorServlet.java:174)\n\tat azkaban.webapp.servlet.LoginAbstractAzkabanServlet.doPost(LoginAbstractAzkabanServlet.java:270)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:707)\n\tat javax.servlet.http.HttpServlet.service(HttpServlet.java:790)\n\tat org.mortbay.jetty.servlet.ServletHolder.handle(ServletHolder.java:511)\n\tat org.mortbay.jetty.servlet.ServletHandler.handle(ServletHandler.java:401)\n\tat org.mortbay.jetty.servlet.SessionHandler.handle(SessionHandler.java:182)\n\tat org.mortbay.jetty.handler.ContextHandler.handle(ContextHandler.java:766)\n\tat org.mortbay.jetty.handler.HandlerWrapper.handle(HandlerWrapper.java:152)\n\tat org.mortbay.jetty.Server.handle(Server.java:326)\n\tat org.mortbay.jetty.HttpConnection.handleRequest(HttpConnection.java:542)\n\tat org.mortbay.jetty.HttpConnection$RequestHandler.content(HttpConnection.java:945)\n\tat org.mortbay.jetty.HttpParser.parseNext(HttpParser.java:756)\n\tat org.mortbay.jetty.HttpParser.parseAvailable(HttpParser.java:218)\n\tat org.mortbay.jetty.HttpConnection.handle(HttpConnection.java:404)\n\tat org.mortbay.jetty.bio.SocketConnector$Connection.run(SocketConnector.java:228)\n\tat org.mortbay.jetty.security.SslSocketConnector$SslConnection.run(SslSocketConnector.java:713)\n\tat org.mortbay.thread.QueuedThreadPool$PoolThread.run(QueuedThreadPool.java:582)\n</pre>\n<hr /><i><small>Powered by Jetty://</small></i><br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n<br/>                                                \n\n</body>\n</html>\n"
     }
     ```

     

6. 获取当前正在的任务

   - url

     *http://localhost:8080/azkaban/running*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "project":"test",
         "flow":"test"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\n  \"execIds\" : [ 9 ,10,11]\n}"
     }
     ```

     

   - error

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{}"
     }
     ```

     

7. 重启任务

   - url

     *http://localhost:8080/azkaban/restart*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "execId":"9",
         "applicationId":"application_1592537240488_0075",    
         "project":"test",
         "flow":"test"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\n  \"project\" : \"test\",\n  \"message\" : \"Execution submitted successfully with exec id 10\",\n  \"flow\" : \"test\",\n  \"execid\" : 10\n}"
     }
     ```

     

   - error

     ```
     {
         "statusCode": 151,
         "message": "停止任务失败",
         "data": "{\n  \"error\" : \"Execution 9 of flow test isn't running.\"\n}"
     }
     ```

     

8. 停止任务

   - url

     *http://localhost:8080/azkaban/stop*

   - 请求方式

     post

   - body

     ```
     {
         "sessionId":"082acbdb-34d2-40a8-b1dd-748dcecf7193",
         "execId":"9",
         "applicationId":"application_1592537240488_0075"
     }
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": true
     }
     ```

     

   - error

     ```
     {
         "statusCode": 151,
         "message": "停止任务失败",
         "data": "{\n  \"error\" : \"Execution 9 of flow test isn't running.\"\n}"
     }
     ```

     

### yarn接口

2. 启动任务

   - 接口

     *http://localhost:8080/executor/start/{spark|flink}*

   - 请求方式

     post

   - body

     ```
     flink :
     
     {
     	"master":"yarn-cluster",
     	"configFile":"application.conf",
     	"taskName":"testFlink"  //必传且唯一
      }
     
     spark:
     
     {
     	"master":"yarn",
     	"deploy-mode":"client",
     	"configFile":"application.spark.conf",
     	"taskName":"testSpark"
      }
     
     
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "application_1592537240488_0084"
     }
     
     ```

3. 获取yarn 运行状况

   - url

     *http://localhost:8080/executor/yarnStatus*

   - 请求方式

     get

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\"scheduler\":{\"schedulerInfo\":{\"rootQueue\":{\"childQueues\":{\"queue\":[{\"fairResources\":{\"memory\":0,\"resourceInformations\":{\"resourceInformation\":[{\"maximumAllocation\":9223372 ........
     ```

     

4. 获取任务日志

   - url

     *http://localhost:8080/executor/y/logger/{applicationId}*

   - 请求方式

     get

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{\"app\":{\"preemptedMemorySeconds\":0,\"reservedVCores\":0,\"applicationType\":\"SPARK\",\"finalStatus\":\"UNDEFINED\",\"trackingUrl\":\"http://hadoop1:8088/proxy/application_1592537240488_0082/\",\"runningContainers\":2,\"preemptedVcoreSeconds\":0,\"timeouts\":{\"timeout\":[{\"remainingTimeInSeconds\":-1,\"expiryTime\":\"UNLIMITED\",\"type\":\"LIFETIME\"}]},\"clusterUsagePercentage\":15.625,\"queueUsagePercentage\":15.625,\"clusterId\":1592537240488,\"vcoreSeconds\":1658,\"preemptedResourceVCores\":0,\"numAMContainerPreempted\":0,\"allocatedMB\":2560,\"reservedMB\":0,\"id\":\"application_1592537240488_0082\",\"state\":\"RUNNING\",\"amHostHttpAddress\":\"hadoop2:8042\",\"memorySeconds\":2204169,\"unmanagedApplication\":false,\"amNodeLabelExpression\":\"\",\"preemptedResourceMB\":0,\"resourceSecondsMap\":{\"entry\":{\"value\":\"1658\",\"key\":\"vcores\"}},\"applicationTags\":\"\",\"startedTime\":1592992355532,\"trackingUI\":\"ApplicationMaster\",\"preemptedResourceSecondsMap\":{},\"priority\":0,\"numNonAMContainerPreempted\":0,\"amContainerLogs\":\"http://hadoop2:8042/node/containerlogs/container_1592537240488_0082_01_000001/root\",\"launchTime\":1592992355733,\"allocatedVCores\":2,\"diagnostics\":\"\",\"logAggregationStatus\":\"NOT_START\",\"name\":\"testSpark\",\"progress\":10.0,\"finishedTime\":0,\"user\":\"root\",\"queue\":\"root.users.root\",\"elapsedTime\":678561}}"
     }
     ```

     

5. 获取当前运行的任务状态

   - url

     *http://localhost:8080/executor/status/{applicationId}*

   - 请求方式

     get

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "RUNNING"
     }
     ```

     

6. 获取当前正在的任务

   - url

     *http://localhost:8080/executor/running*

   - 请求方式

     get

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "[{\"id\":\"Application-Id\",\"name\":\"Application-Name\"},{\"id\":\"application_1592537240488_0071\",\"name\":\"testaz\"},{\"id\":\"application_1592537240488_0073\",\"name\":\"testaz\"},{\"id\":\"application_1592537240488_0080\",\"name\":\"Flink session cluster\"},{\"id\":\"application_1592537240488_0083\",\"name\":\"testSpark\"},{\"id\":\"application_1592537240488_0081\",\"name\":\"testSpark\"},{\"id\":\"application_1592537240488_0082\",\"name\":\"testSpark\"}]"
     }
     ```

     

   - error

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "{}"
     }
     ```

     

7. 重启任务

   - url

     *http://localhost:8080/executor/restart/{spark | flink }/{applicationId}*

   - 请求方式

     post

   - body

     ```
     flink :
     
     {
     	"master":"yarn-cluster",
     	"configFile":"application.conf",
     	"taskName":"testFlink"  //必传且唯一
      }
     
     spark:
     
     {
     	"master":"yarn",
     	"deploy-mode":"client",
     	"configFile":"application.spark.conf",
     	"taskName":"testSpark"
      }
     
     ```

     

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": "application_1592537240488_0084"
     }
     ```

     

     

8. 停止任务

   - url

     *http://localhost:8080/executor/stop/{applicationid}*

   - 请求方式

     post

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": true
     }
     ```

     

     

### 文件接口

1. 创建目录

   - url

     http://localhost:8080/file/mkdir?dirName=c://test

   - method

     get

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": true
     }
     ```

     

2. 创建文件

   - url

     http://localhost:8080/file/create?fileName=c://test//test.txt

   - method

     get

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": true
     }
     ```

     

3. 删除文件

   - url

     http://localhost:8080/file/del?fileName=c://test

   - method

     get

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": true
     }
     ```

     

4. 上传文件

   - url

     http://localhost:8080/file/upload?fileName=text.txt&file=MultipartFile

   - method

     post

   - response

     ```
     {
         "statusCode": 200,
         "message": "成功!",
         "data": true
     }
     ```

   5. 下载文件

      - url

        http://localhost:8080/file/download?srcfile=c://test//test.txt

      - method

        get

        

        

