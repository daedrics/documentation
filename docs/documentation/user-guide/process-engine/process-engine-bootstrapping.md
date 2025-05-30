---

title: 'Process Engine Bootstrapping'
sidebar_position: 10

menu:
  main:
    identifier: "user-guide-process-engine-bootstrapping"
    parent: "user-guide-process-engine"

---


You have a number of options to configure and create a process engine depending on whether you use an application managed or a shared, container managed process engine.


# Application Managed Process Engine

You manage the process engine as part of your application. The following ways exist to configure it:

* [Programmatically via Java API](#bootstrap-a-process-engine-using-the-java-api)
* [Via XML configuration](#configure-process-engine-using-operaton-cfg-xml)
* [Via Spring](../spring-framework-integration/index.md)


# Shared, Container Managed Process Engine

A container of your choice (e.g., Tomcat, Wildfly or IBM WebSphere) manages the process engine for you. The configuration is carried out in a container specific way, see [Runtime Container Integration](../runtime-container-integration/index.md) for details.


## ProcessEngineConfiguration Bean

The Operaton engine uses the <a class="javadocref" href="org/operaton/bpm/engine/ProcessEngineConfiguration.html">ProcessEngineConfiguration</a> bean to configure and construct a standalone Process Engine. There are multiple subclasses available that can be used to define the process engine configuration. These classes represent different environments, and set defaults accordingly. It's a best practice to select the class that matches (most of) your environment to minimize the number of properties needed to configure the engine. The following classes are currently available:

* `org.operaton.bpm.engine.impl.cfg.StandaloneProcessEngineConfiguration`
The process engine is used in a standalone way. The engine itself will take care of the transactions. By default the database will only be checked when the engine boots (an exception is thrown if there is no database schema or the schema version is incorrect).
* `org.operaton.bpm.engine.impl.cfg.StandaloneInMemProcessEngineConfiguration`
This is a convenience class for unit testing purposes. The engine itself will take care of the transactions. An H2 in-memory database is used by default. The database will be created and dropped when the engine boots and shuts down. When using this, probably no additional configuration is needed (except, for example, when using the job executor or mail capabilities).
* `org.operaton.bpm.engine.spring.SpringProcessEngineConfiguration`
To be used when the process engine is used in a Spring environment. See the Spring integration section for more information.
* `org.operaton.bpm.engine.impl.cfg.JtaProcessEngineConfiguration`
To be used when the engine runs in standalone mode, with JTA transactions.


## Bootstrap a Process Engine Using the Java API

You can configure the process engine programmatically by creating the right ProcessEngineConfiguration object or by using some pre-defined one:

```java
ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```

Now you can call the `buildProcessEngine()` operation to create a Process Engine:

```java
ProcessEngine processEngine = ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration()
  .setDatabaseSchemaUpdate(ProcessEngineConfiguration.DB_SCHEMA_UPDATE_FALSE)
  .setJdbcUrl("jdbc:h2:mem:my-own-db;DB_CLOSE_DELAY=1000")
  .setJobExecutorActivate(true)
  .buildProcessEngine();
```


## Configure Process Engine Using Operaton cfg XML

The easiest way to configure your Process Engine is through an XML file called `camunda.cfg.xml`. Using that you can simply do:

```java
ProcessEngine processEngine = ProcessEngines.getDefaultProcessEngine()
```

The `camunda.cfg.xml` must contain a bean that has the id `processEngineConfiguration`, select the `ProcessEngineConfiguration` class best suited to your needs:

```xml
<bean id="processEngineConfiguration" class="org.operaton.bpm.engine.impl.cfg.StandaloneProcessEngineConfiguration">
```

This will look for a `camunda.cfg.xml` file on the classpath and construct an engine based on the configuration in that file. The following snippet shows an example configuration:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="processEngineConfiguration" class="org.operaton.bpm.engine.impl.cfg.StandaloneProcessEngineConfiguration">

    <property name="jdbcUrl" value="jdbc:h2:mem:operaton;DB_CLOSE_DELAY=1000" />
    <property name="jdbcDriver" value="org.h2.Driver" />
    <property name="jdbcUsername" value="sa" />
    <property name="jdbcPassword" value="" />

    <property name="databaseSchemaUpdate" value="true" />

    <property name="jobExecutorActivate" value="false" />

    <property name="mailServerHost" value="mail.my-corp.com" />
    <property name="mailServerPort" value="5025" />
  </bean>

</beans>
```
If no resource `camunda.cfg.xml` is found, the default engine will search for the file `activiti.cfg.xml` as a fallback. If both are missing, the engine stops and prints an error message about the missing configuration resource.

Note that the configuration XML is in fact a Spring configuration. This does not mean that the Operaton engine can only be used in a Spring environment! We are simply leveraging the parsing and dependency injection capabilities of Spring internally for building up the engine.

The ProcessEngineConfiguration object can also be created programmatically using the configuration file. It is also possible to use a different bean id:

```java
ProcessEngineConfiguration.createProcessEngineConfigurationFromResourceDefault();
ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource);
ProcessEngineConfiguration.createProcessEngineConfigurationFromResource(String resource, String beanName);
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream);
ProcessEngineConfiguration.createProcessEngineConfigurationFromInputStream(InputStream inputStream, String beanName);
```

It is also possible to not use a configuration file and create a configuration based on defaults (see the different supported classes for more information).

```java
ProcessEngineConfiguration.createStandaloneProcessEngineConfiguration();
ProcessEngineConfiguration.createStandaloneInMemProcessEngineConfiguration();
```

All these `ProcessEngineConfiguration.createXXX()` methods return a `ProcessEngineConfiguration` that can further be tweaked if needed. After calling the `buildProcessEngine()` operation, a `ProcessEngine` is created as explained above.


## Configure Process Engine in the bpm-platform.xml

The `bpm-platform.xml` file is used to configure Operaton in the following distributions:

* Apache Tomcat
* IBM WebSphere Application Server
* Oracle WebLogic Application Server

The `<process-engine ... />` xml tag allows you to define a process engine:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpm-platform xmlns="http://www.operaton.org/schema/1.0/BpmPlatform"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://www.operaton.org/schema/1.0/BpmPlatform http://www.operaton.org/schema/1.0/BpmPlatform">

  <job-executor>
    <job-acquisition name="default" />
  </job-executor>

  <process-engine name="default">
    <job-acquisition>default</job-acquisition>
    <configuration>org.operaton.bpm.engine.impl.cfg.StandaloneProcessEngineConfiguration</configuration>
    <datasource>java:jdbc/ProcessEngine</datasource>

    <properties>
      <property name="history">full</property>
      <property name="databaseSchemaUpdate">true</property>
      <property name="authorizationEnabled">true</property>
    </properties>

  </process-engine>
</bpm-platform>
```

See the [Deployment Descriptor Reference](../../reference/deployment-descriptors/descriptors/bpm-platform-xml.md) for complete documentation of the syntax of the `bpm-platform.xml` file.


## Configure Process Engine in the processes.xml

The process engine can also be configured and bootstrapped using the `META-INF/processes.xml` file. See [Section on processes.xml file](../process-applications/the-processes-xml-deployment-descriptor.md) for details.

See the [Deployment Descriptor Reference](../../reference/deployment-descriptors/descriptors/processes-xml.md) for complete documentation of the syntax of the `processes.xml` file.
