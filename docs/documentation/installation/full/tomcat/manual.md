---

title: 'Manual Installation'
sidebar_position: 20
description: "Manually install and configure the Full Distribution on a vanilla Apache Tomcat Server."

---
# Install the Full Distribution on a Tomcat Application Server manually

This section describes how you can install Operaton and its components on a vanilla [Apache Tomcat](http://tomcat.apache.org/), if you are not able to use the pre-packaged Tomcat distribution. In addition, download a [Tomcat distribution](https://downloads.operaton.cloud/release/operaton-bpm/tomcat/) to fetch the required Operaton modules.

:::note[Reading the Guide}
Throughout this guide we will use a number of variables to denote common path names and constants:

* `$TOMCAT_HOME` points to the main directory of the tomcat server.
* `$TOMCAT_VERSION` denotes the version of Tomcat server.
* `$PLATFORM_VERSION` denotes the version of Operaton you want to install or already have installed, e.g. `7.0.0`.
* `$TOMCAT_DISTRIBUTION` represents the downloaded pre-packaged Operaton distribution for Tomcat, e.g. `operaton-bpm-tomcat-$PLATFORM_VERSION.zip` or `operaton-bpm-tomcat-$PLATFORM_VERSION.tar.gz`.

:::

:::note[Known Limitations in Tomcat 10]

**<h4>Weld Class Loading Issues</h4>**

In deployment scenarios involving one or more process applications with managed beans, classloading issues may occur if the WELD library is directly embedded in the WAR's `/WEB-INF/lib` folder.
To resolve this, move the weld library away from the war and place it into the `$CATALINA_HOME/lib` folder.

The above workaround is not guaranteed to work for cases with bean references between WAR deployments (WAR A referencing a bean from WAR B).

The following test scenarios fail on Tomcat 10:

* [CallActivityContextSwitchTest](https://github.com/operaton/operaton-bpm-platform/blob/f37877b822dabcbf3cee5806bd5833d18cdcb543/qa/integration-tests-engine/src/test/java/org/operaton/bpm/integrationtest/functional/context/CallActivityContextSwitchTest.java)
* [CdiBeanCallActivityResolutionTest](https://github.com/operaton/operaton-bpm-platform/blob/f37877b822dabcbf3cee5806bd5833d18cdcb543/qa/integration-tests-engine/src/test/java/org/operaton/bpm/integrationtest/functional/cdi/CdiBeanCallActivityResolutionTest.java)
:::


# Setup

Before you can install the Operaton components, you need to perform a number of required setup steps.


## Create the Database Schema and Tables

In the default configuration of the distribution, the database schema and all required tables are automatically created in an H2 database when the engine starts up for the first time. If you do not want to use the H2 database, you have to

* Create a database schema for Operaton yourself.
* Install the database schema to create all required tables and default indices using our [database schema installation guide](../installation/database-schema.md).

When you create the tables manually, then you have to configure the engine to **not** create tables at startup by setting the `databaseSchemaUpdate` property to `false` (or, in case you are using Oracle, to `noop`). In Tomcat, this is done in the `bpm-platform.xml`, located in the `$TOMCAT_DISTRIBUTION\server\apache-tomcat-$VERSION\conf\` folder.


## Add BPM Bootstrap Server Listener

Add the entry `org.operaton.bpm.container.impl.tomcat.TomcatBpmPlatformBootstrap` as Listener before the `GlobalResourcesLifecycleListener` in your `$TOMCAT_HOME/conf/server.xml`. This class is responsible for starting and stopping Operaton as Tomcat is started and stopped.

```xml
<Server port="8005" shutdown="SHUTDOWN">
  ...
  <Listener className="org.operaton.bpm.container.impl.tomcat.TomcatBpmPlatformBootstrap" />
  ...
```


## Configure a JDBC Resource

To configure a JDBC Resource you have to edit the file `$TOMCAT_HOME/conf/server.xml`. This could look like the following example for an H2 database:

```xml
<Server>
  ...
  <GlobalNamingResources>
    ...
    <Resource name="jdbc/ProcessEngine"
              auth="Container"
              type="javax.sql.DataSource"
              factory="org.apache.tomcat.jdbc.pool.DataSourceFactory"
              uniqueResourceName="process-engine"
              driverClassName="org.h2.Driver"
              url="jdbc:h2:./operaton-h2-dbs/process-engine;TRACE_LEVEL_FILE=0"
              defaultTransactionIsolation="READ_COMMITTED"
              username="sa"
              password="sa"
              maxActive="20"
              minIdle="5"
              maxIdle="20" />
  </GlobalNamingResources>
</Server>
```

For more information on the creation of JDBC datasources have a look at the documentation of your Tomcat version:
[9.0](https://tomcat.apache.org/tomcat-9.0-doc/jndi-datasource-examples-howto.html).


## Add Operaton Services

Copy the following blocks from `${TOMCAT_DISTRIBUTION}/server/apache-tomcat-${TOMCAT_VERSION}/conf/server.xml`
  into `${TOMCAT_HOME}/conf/server.xml`:

```xml
     <Resource name="global/operaton-bpm-platform/process-engine/ProcessEngineService!org.operaton.bpm.ProcessEngineService" auth="Container"
              type="org.operaton.bpm.ProcessEngineService"
              description="Operaton Platform Process Engine Service"
              factory="org.operaton.bpm.container.impl.jndi.ProcessEngineServiceObjectFactory" />

    <Resource name="global/operaton-bpm-platform/process-engine/ProcessApplicationService!org.operaton.bpm.ProcessApplicationService" auth="Container"
              type="org.operaton.bpm.ProcessApplicationService"
              description="Operaton Platform Process Application Service"
              factory="org.operaton.bpm.container.impl.jndi.ProcessApplicationServiceObjectFactory" />
```


## Add Required Libraries

Copy all libraries from the `$TOMCAT_DISTRIBUTION/lib/` folder to the Tomcat library folder `$TOMCAT_HOME/lib`:

Furthermore, you have to merge your corresponding JDBC driver into the folder `$TOMCAT_HOME/lib`.


## Add bpm-platform.xml

You have to add the file `bpm-platform.xml` to the folder `$TOMCAT_HOME/conf` or, optionally, you can configure the location through some available mechanisms, see [Configure location of the bpm-platform.xml file](../reference/deployment-descriptors/descriptors/bpm-platform-xml.md#configure-location-of-the-bpm-platform-xml-file):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpm-platform xmlns="http://www.operaton.org/schema/1.0/BpmPlatform"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.operaton.org/schema/1.0/BpmPlatform http://www.operaton.org/schema/1.0/BpmPlatform ">

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
      <property name="jobExecutorDeploymentAware">true</property>
    </properties>

  </process-engine>

</bpm-platform>
```


## Secure Tomcat

Follow the Tomcat Security Howto of your Tomcat version:
[9.0](https://tomcat.apache.org/tomcat-9.0-doc/security-howto.html),
[10.1](https://tomcat.apache.org/tomcat-10.1-doc/security-howto.html).

In particular, go to `${TOMCAT_HOME}/webapps/` and remove the directories
`ROOT`, `docs`, `examples`, `manager` and `host-manager`.


# Optional Components

This section describes how to install optional Operaton dependencies onto a Tomcat server. None of these are required to work with the core platform.


## Cockpit, Tasklist and Admin

The following steps are required to deploy the applications:

1. Download the Operaton web application that contains both applications from our [Artifact Repository](https://artifacts.camunda.com/artifactory/operaton-bpm/org/operaton/bpm/webapp/operaton-webapp-tomcat/).
   Or switch to the private repository for the enterprise version (User and password from license required).
    * For [Tomcat 10](https://artifacts.camunda.com/ui/native/operaton-bpm/org/operaton/bpm/webapp/operaton-webapp-tomcat-jakarta/), the name of the artifact is `$PLATFORM_VERSION/operaton-webapp-tomcat-jakarta-$PLATFORM_VERSION.war`.
    * For [Tomcat 9](https://artifacts.camunda.com/ui/native/operaton-bpm/org/operaton/bpm/webapp/operaton-webapp-tomcat/), the name of the artifact is `$PLATFORM_VERSION/operaton-webapp-tomcat-$PLATFORM_VERSION.war`.
2. Copy the war file to `$TOMCAT_HOME/webapps/operaton.war`.
   Optionally you may name it differently or extract it to a folder to deploy it to a different context path.
3. Startup Tomcat.
4. Access Cockpit and Tasklist via `/operaton/app/cockpit` and `/operaton/app/tasklist` or under the context path you configured.


## REST API

The following steps are required to deploy the REST API:

1. Download the REST API web application archive from our [Artifact Repository](https://artifacts.camunda.com/artifactory/operaton-bpm/org/operaton/bpm/operaton-engine-rest/).
    Or switch to the private repository for the enterprise version (User and password from license required).
    Choose the correct version named `$PLATFORM_VERSION/operaton-engine-rest-$PLATFORM_VERSION-tomcat.war`.
    * For [Tomcat 10](https://artifacts.camunda.com/artifactory/public/org/operaton/bpm/operaton-engine-rest-jakarta/), the name of the artifact is `$PLATFORM_VERSION/operaton-engine-rest-jakarta-$PLATFORM_VERSION-tomcat.war`.
    * For [Tomcat 9](https://artifacts.camunda.com/artifactory/public/org/operaton/bpm/operaton-engine-rest/), the name of the artifact is `$PLATFORM_VERSION/operaton-engine-rest-$PLATFORM_VERSION-tomcat.war`.
2. Copy the war file to `$TOMCAT_HOME/webapps`.
   Optionally you may rename it or extract it to a folder to deploy it to a specific context like `/engine-rest`.
3. Startup Tomcat.
4. Access the REST API on the context you configured.
    For example, http://localhost:8080/engine-rest/engine should return the names of all engines of the platform, provided that you deployed the application in the context `/engine-rest`.
5. Enable authentication as described in the [REST API documentation](../reference/rest/overview/authentication.md)


## Operaton Connect Plugin

Add the following artifacts (if not existing) from the folder `$TOMCAT_DISTRIBUTION/lib/` to the folder `$TOMCAT_HOME/lib/`:

* `operaton-engine-plugin-connect-$PLATFORM_VERSION.jar`
* `operaton-commons-utils-$PLATFORM_VERSION.jar`

In order to activate Operaton Connect functionality for a process engine, a process engine plugin has to be registered in `$TOMCAT_HOME/conf/bpm-platform.xml` as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpm-platform ...>
  <process-engine name="default">
    ...
    <plugins>
      ... existing plugins ...
      <plugin>
        <class>org.operaton.connect.plugin.impl.ConnectProcessEnginePlugin</class>
      </plugin>
    </plugins>
    ...
  </process-engine>

</bpm-platform>
```


## Operaton Spin

Add the following artifacts (if not existing) from the folder `$TOMCAT_DISTRIBUTION/lib/` to the folder `$TOMCAT_HOME/lib/`:

* `operaton-spin-dataformat-all-$PLATFORM_VERSION.jar`
* `operaton-spin-core-$PLATFORM_VERSION.jar`
* `operaton-engine-plugin-spin-$PLATFORM_VERSION.jar`
* `operaton-commons-utils-$PLATFORM_VERSION.jar`

In order to activate Operaton Spin functionality for a process engine, a process engine plugin has to be registered in `$TOMCAT_HOME/conf/bpm-platform.xml` as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<bpm-platform ...>
  ...
  <process-engine name="default">
    ...
    <plugins>
      ... existing plugins ...
      <plugin>
        <class>org.operaton.spin.plugin.impl.SpinProcessEnginePlugin</class>
      </plugin>
    </plugins>
    ...
  </process-engine>
  ...
</bpm-platform>
```


## Groovy Scripting

Add the following artifacts (if not existing) from the folder `$TOMCAT_DISTRIBUTION/lib/` to the folder `$TOMCAT_HOME/lib/`:

* `groovy-$GROOVY_VERSION.jar`
* `groovy-jsr223-$GROOVY_VERSION.jar`
* `groovy-json-$GROOVY_VERSION.jar`
* `groovy-xml-$GROOVY_VERSION.jar`
* `groovy-templates-$GROOVY_VERSION.jar`
* `groovy-dateutil-$GROOVY_VERSION.jar`
* `groovy-datetime-$GROOVY_VERSION.jar`


## Freemarker Integration

Add the following artifacts (if not existing) from the folder `$TOMCAT_DISTRIBUTION/lib/` to the folder `$TOMCAT_HOME/lib/`:

* `operaton-template-engines-freemarker-$PLATFORM_VERSION.jar`
* `freemarker-2.3.31.jar`
* `operaton-commons-utils-$PLATFORM_VERSION.jar`

## GraalVM JavaScript Integration

Add the following artifacts (if not existing) from the folder `$TOMCAT_DISTRIBUTION/lib/` to the folder `$TOMCAT_HOME/lib/`:

* `graal-sdk-21.1.0.jar`
* `icu4j-68.2.jar`
* `js-21.1.0.jar`
* `js-scriptengine-21.1.0.jar`
* `regex-21.1.0.jar`
* `truffle-api-21.1.0.jar`