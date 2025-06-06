---

title: 'The Process Application class'
sidebar_position: 10

menu:
  main:
    identifier: "user-guide-process-applications-class"
    parent: "user-guide-process-applications"

---

You can delegate the bootstrapping of the process engine and process deployment to a process application class. The basic ProcessApplication functionality is provided by the `org.operaton.bpm.application.AbstractProcessApplication` base class. Based on this class there is a set of environment-specific sub classes that realize integration within a specific environment:

* **ServletProcessApplication**: To be used for process applications in a Servlet container like Apache Tomcat.
* **JakartaServletProcessApplication**: To be used for process applications in a Jakarta Servlet 5.0+ container like WildFly 27 and above.
* **EjbProcessApplication**: To be used in a Java EE application server like IBM WebSphere Application Server.
* **JakartaEjbProcessApplication**: To be used in a Jakarta EE 9+ application server like Wildfly 27+.
* **EmbeddedProcessApplication**: To be used when embedding the process engine in an ordinary Java SE application.
* **SpringProcessApplication**: To be used for bootstrapping the process application from a Spring Application Context.

In the following section, we walk through the different implementations and discuss where and how they can be used.

# The ServletProcessApplication and JakartaServletProcessApplication

:::note[Jakarta Servlet environments]
  The `JakartaServletProcessApplication` can only be used in environments working with Jakarta Servlet 5.0 and above like WildFly 27+.
  The mechanisms described herein for the `ServletProcessApplication` apply in the same way.
:::

**Supported on:** Apache Tomcat, Wildfly. The Servlet process application is supported on all containers. Read the [note about Servlet Process Application and EJB/Java EE containers](#using-servlet-process-applications-inside-an-ejb-jakarta-ee-java-ee-container-such-as-wildfly)

**Packaging**: WAR (or embedded WAR inside EAR)

The `ServletProcessApplication` class is the base class for developing process applications based on the Servlet Specification (Java Web Applications). The servlet process application implements the Java EE `javax.servlet.ServletContextListener` interface which allows it to participate in the deployment lifecycle of your Web application

The following is an example of a Servlet Process Application:

```java
package org.operaton.bpm.example.loanapproval;

import org.operaton.bpm.application.ProcessApplication;
import org.operaton.bpm.application.impl.ServletProcessApplication;

@ProcessApplication("Loan Approval App")
public class LoanApprovalApplication extends ServletProcessApplication {
  // empty implementation
}
```

Notice the `@ProcessApplication` annotation. This annotation fulfills two purposes:

* **provide the name of the ProcessApplication**: You can provide a custom name for your process application using the annotation: `@ProcessApplication("Loan Approval App")`. If no name is provided, a name is automatically detected. In case of a servlet process application, the name of the `ServletContext` is used.
* **trigger auto-deployment**. In a Servlet 3.0 container, the annotation is sufficient for making sure that the process application is automatically picked up by the servlet container and automatically added as a ServletContextListener to the Servlet Container deployment. This functionality is realized by a `javax.servlet.ServletContainerInitializer` implementation named `org.operaton.bpm.application.impl.ServletProcessApplicationDeployer` which is located in the operaton-engine module. The implementation works for both embedded deployment of the operaton-engine.jar as a web application library in the `WEB-INF/lib` folder of your WAR file, or for the deployment of the operaton-engine.jar as a shared library in the shared library (e.g., Apache Tomcat global `lib/` folder) directory of your application server. The Servlet 3.0 Specification foresees both deployment scenarios. In case of embedded deployment, the `ServletProcessApplicationDeployer` is notified once, when the web application is deployed. In case of deployment as a shared library, the `ServletProcessApplicationDeployer` is notified for each WAR file containing a class annotated with `@ProcessApplication` (as required by the Servlet 3.0 Specification).

This means that in case you deploy to a Servlet 3.0 compliant container (such as Apache Tomcat) annotating your class with `@ProcessApplication` is sufficient.

:::note
  There is a [project template for Maven](../user-guide/process-applications/maven-archetypes.md) called ```operaton-archetype-servlet-war```, which gives you a complete running project based on a servlet process application.
:::

## Using Servlet process applications inside an EJB/Jakarta EE/Java EE container such as Wildfly

You can use the `ServletProcessApplication` inside an EJB / Java EE container such as Wildfly 26 and below.
In Jakarta EE 9+ containers like WildFly 27 and above, you need to use the `JakartaServletProcessApplication`.

Process application bootstrapping and deployment will work in the same way as in a Servlet container.
However, you will not be able to use all Jakarta EE/Java EE features at runtime. In contrast to the
`EjbProcessApplication` (see the next section), the `ServletProcessApplication` and `JakartaServletProcessApplication`
do not perform proper Jakarta EE/Java EE cross-application context switching. When the process engine invokes Java
Delegates from your application, only the Context Class Loader of the current Thread is set to the classloader of your application.
This does allow the process engine to resolve Java Delegate implementations from your application but the container will not
perform an EE context switch to your application. As a consequence, if you use the Servlet process application inside a
Jakarta EE/Java EE container, you will not be able to use features like:

* Using CDI beans and EJBs as JavaDelegate implementations in combination with the Job Executor.
* Uusing `@RequestScoped` CDI Beans with the Job Executor.
* Looking up JNDI resources from the application's naming scope.

If your application does not use such features, it is perfectly fine to use the servlet process application inside an EE container.
In that case you only get servlet specification guarantees.

# The EjbProcessApplication and JakartaEjbProcessApplication

:::note[Jakarta EJB environments]
  The `JakartaEjbProcessApplication` can only be used in environments working with Jakarta EJB.
  The mechanisms described herein for the `EjbProcessApplication` apply in the same way.
:::

**Supported on:** Wildfly. The `EjbProcessApplication` is supported on Java EE 6 to Java EE 8 containers like WildFly 26 and below.
The `JakartaEjbProcessApplication` is supported on Jakarta EE 9+ containers like WildFly 27 and above.
It is not supported on Servlet Containers like Apache Tomcat. It may be adapted to work inside Java EE 5 Containers.

**Packaging:** JAR, WAR, EAR

The `EjbProcessApplication` and `JakartaEjbProcessApplication` are the base classes for developing Jakarta EE/Java EE-based process applications.
An EJB process application class itself must be deployed as an EJB.

To add an EJB process application to your Java application, you have two options:

* **Bundle the Operaton EJB Client**: we provide a generic, reusable EJB process application implementation (named
`org.operaton.bpm.application.impl.ejb.DefaultEjbProcessApplication`) bundled as a maven artifact. You can add this
implementation to your application as a maven dependency. Use the `operaton-ejb-client` artifact for Java EE or
the `operaton-ejb-client-jakarta` artifact for Jakarta EE 9+ applications.
* **Write a custom EJB process application**: if you want to customize the behavior of the `EjbProcessApplication`
or `JakartaEjbProcessApplication`, you can write a custom subclass of the respective class and add it to your application.

Both options are explained in greater detail below.


## Bundling the Operaton EJB Client Jar

The most convenient option for deploying a process application to a Java EE EJB container is by adding the following maven dependency to your maven project:

:::note
  Please import the [Operaton BOM](/get-started/apache-maven/) to ensure correct versions for every Operaton project.
:::

```xml
<dependency>
  <groupId>org.operaton.bpm.javaee</groupId>
  <artifactId>operaton-ejb-client</artifactId>
</dependency>
```

For Jakarta EE 9+ EJB containers, use the following dependency instead:

```xml
<dependency>
  <groupId>org.operaton.bpm.javaee</groupId>
  <artifactId>operaton-ejb-client-jakarta</artifactId>
</dependency>
```

The Operaton EJB Client contains a reusable default implementation of the respective EJB process application as a Singleton Session Bean with auto-activation.

This deployment option requires that your project is a composite deployment (such as a WAR or EAR) since you need to add a library JAR file.
You could of course use something like the maven shade plugin for adding the class contained in the Operaton EJB Client artifact to a JAR-based deployment.

:::note
  We always recommend using the Operaton EJB Client over deploying a custom `EjbProcessApplication` class unless you want to customize the behavior of the `EjbProcessApplication`.

  There is a [project template for Maven](../user-guide/process-applications/maven-archetypes.md) called ```operaton-archetype-servlet-war```, which gives you a complete running project based on a Java EE servlet process application.
:::


## Deploying a Custom `EjbProcessApplication` Class

If you want to customize the behavior of the `EjbProcessApplication` class you have the option of writing a custom `EjbProcessApplication` class. The following is an example of such an implementation:

```java
@Singleton
@Startup
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
@TransactionAttribute(TransactionAttributeType.REQUIRED)
@ProcessApplication
@Local(ProcessApplicationInterface.class)
public class MyEjbProcessApplication extends EjbProcessApplication {

  @PostConstruct
  public void start() {
    deploy();
  }

  @PreDestroy
  public void stop() {
    undeploy();
  }

}
```


## Expose Servlet Context Path Using a Custom EJB process application

If your application is a `WAR` (or a `WAR` inside an `EAR`) and you want to use [embedded forms](../user-guide/task-forms/index.md#embedded-task-forms) or [external task forms](../user-guide/task-forms/index.md#external-task-forms) inside the [Tasklist](../webapps/tasklist/index.md) application, then your custom EJB process application must expose the servlet context path of your application as a property. This enables the Tasklist to resolve the path to the embedded or external task forms.

Therefore, your custom EJB process application must be extended by a `Map` and a getter-method for that `Map` as follows:

```java
@Singleton
@Startup
@ConcurrencyManagement(ConcurrencyManagementType.BEAN)
@TransactionAttribute(TransactionAttributeType.REQUIRED)
@ProcessApplication
@Local(ProcessApplicationInterface.class)
public class MyEjbProcessApplication extends EjbProcessApplication {

  protected Map<String, String> properties = new HashMap<String, String>();

  @PostConstruct
  public void start() {
    deploy();
  }

  @PreDestroy
  public void stop() {
    undeploy();
  }

  public Map<String, String> getProperties() {
    return properties;
  }

}
```

Furthermore, to provide the servlet context path a custom Java EE `javax.servlet.ServletContextListener` must be added to your application. Inside your custom implementation of the `ServletContextListener` you have to

* inject your custom EJB process application using the `@EJB` annotation,
* resolve the servlet context path and
* expose the servlet context path through the `ProcessApplicationInfo#PROP_SERVLET_CONTEXT_PATH` property inside your custom EJB process application.

This can be done as follows:

```java
public class ProcessArchiveServletContextListener implements ServletContextListener {

  @EJB
  private ProcessApplicationInterface processApplication;

  public void contextInitialized(ServletContextEvent contextEvent) {

    String contextPath = contextEvent.getServletContext().getContextPath();

    Map<String, String> properties = processApplication.getProperties();
    properties.put(ProcessApplicationInfo.PROP_SERVLET_CONTEXT_PATH, contextPath);
  }

  public void contextDestroyed(ServletContextEvent arg0) {
  }

}
```
Finally, the custom `ProcessArchiveServletContextListener` has to be added to your `WEB-INF/web.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">

  <listener>
    <listener-class>org.my.project.ProcessArchiveServletContextListener</listener-class>
  </listener>

  ...

</web-app>
```


## Invocation Semantics of the EJB process application

The fact that the EJB process application exposes itself as a Session Bean Component inside the EJB container determines

 * the invocation semantics when invoking code from the process application and
 * the nature of the `ProcessApplicationReference` held by the process engine.

When the process engine invokes the EJB process application, it gets EJB invocation semantics. For example, if your process application provides a `JavaDelegate` implementation, the process engine will call the `EjbProcessApplication`'s `execute(Callable)` method and from that method invoke the `JavaDelegate`. This makes sure that

  * the call is intercepted by the EJB container and "enters" the process application legally.
  * the `JavaDelegate` may take advantage of the EJB process application's invocation context and resolve resources from the component's environment (such as a `java:comp/BeanManager`).

<pre>
                   Big pile of EJB interceptors
                                |
                                |  +--------------------+
                                |  |Process Application |
                  invoke        v  |                    |
 ProcessEngine ----------------OOOOO--> Java Delegate   |
                                   |                    |
                                   |                    |
                                   +--------------------+
</pre>

The EJB process application allows to hook into the invocation by overriding the `execute(Callable callable, InvocationContext invocationContext)` method. It provides the context of the current invocation (e.g., the execution) and can be used to execute custom code, for example initialize the security context before a service task is invoked.

```java
public class MyEjbProcessApplication extends EjbProcessApplication {

  @Override
  public <T> T execute(Callable<T> callable, InvocationContext invocationContext) {

    if(invocationContext != null) {
      // execute custom code (e.g. initialize the security context)
    }

    return execute(callable);
  }
}
```

When the EJB process application registers with a process engine (see `ManagementService#registerProcessApplication(String, ProcessApplicationReference)`, the process application passes a reference to itself to the process engine. This reference allows the process engine to reference the process application. The EJB process application takes advantage of the EJB Containers naming context and passes a reference containing the EJB process application's Component Name to the process engine. Whenever the process engine needs access to process application, the actual component instance is looked up and invoked.

# The EmbeddedProcessApplication

**Supported on:** JVM, Apache Tomcat, Wildfly

**Packaging:** JAR, WAR, EAR

The `org.operaton.bpm.application.impl.EmbeddedProcessApplication` can only be used in combination with an embedded process engine. Usage in combination with a Shared Process Engine is not supported as the class performs no process application context switching at runtime.

The embedded process application also does not provide auto-startup. You need to manually call the `#deploy` method of your process application:

```java
// instantiate the process application
MyProcessApplication processApplication = new MyProcessApplication();

// deploy the process application
processApplication.deploy();

// interact with the process engine
ProcessEngine processEngine = BpmPlatform.getDefaultProcessEngine();
processEngine.getRuntimeService().startProcessInstanceByKey(...);

// undeploy the process application
processApplication.undeploy();
```

Where the class `MyProcessApplication` could look like this:

```java
@ProcessApplication(
    name="my-app",
    deploymentDescriptors={"path/to/my/processes.xml"}
)
public class MyProcessApplication extends EmbeddedProcessApplication {

}
```

Remember that to make a manually managed EmbeddedProcessApplication work, you will have to register your ProcessEngine on the RuntimeContainer:

```java
RuntimeContainerDelegate runtimeContainerDelegate = RuntimeContainerDelegate.INSTANCE.get();
runtimeContainerDelegate.registerProcessEngine(processEngine);
```


# The SpringProcessApplication

**Supported on:** JVM, Apache Tomcat.

**Packaging:** JAR, WAR, EAR

The `org.operaton.bpm.engine.spring.application.SpringProcessApplication` class allows bootstrapping a process application through a Spring Application Context. You can either reference the SpringProcessApplication class from an XML-based application context configuration file or use an annotation-based setup.

If your application is a web application you should use `org.operaton.bpm.engine.spring.application.SpringServletProcessApplication` as it provides support for exposing the servlet context path through the `ProcessApplicationInfo#PROP_SERVLET_CONTEXT_PATH` property.

We recommend to always use `SpringServletProcessApplication` unless the deployment is not a web application. Using this class requires the ```org.springframework:spring-web``` module to be on the classpath.



## Configuring a Spring Process Application

The following shows an example of how to bootstrap a SpringProcessApplication inside a Spring application context XML file:

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

  <bean id="invoicePa" class="org.operaton.bpm.engine.spring.application.SpringServletProcessApplication" />

</beans>
```

Remember to additionally add a `META-INF/processes.xml` file.

> If you are manually managing your processEngine, you will have to register it on the RuntimeContainerDelegate as described in the EmbeddedProcessEngine section.


## Process Application Name

The `SpringProcessApplication` will use the bean name (`id="invoicePa"` in the example above) as auto-detected name for the process application. Make sure to provide a unique process application name here (unique across all process applications deployed on a single application server instance). As an alternative, you can provide a custom subclass of `SpringProcessApplication` (or `SpringServletProcessApplication`) and override the `getName()` method.


## Configure a Managed Process Engine Using Spring

If you use a Spring process application, you may want to configure your process engine inside the Spring application context Xml file (as opposed to the processes.xml file). In this case, you must use the `org.operaton.bpm.engine.spring.container.ManagedProcessEngineFactoryBean` class for creating the process engine object instance. In addition to creating the process engine object, this implementation registers the process engine with the Operaton infrastructure so that the process engine is returned by the `ProcessEngineService`. The following is an example of how to configure a managed process engine using Spring.

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="dataSource" class="org.springframework.jdbc.datasource.TransactionAwareDataSourceProxy">
        <property name="targetDataSource">
            <bean class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
                <property name="driverClass" value="org.h2.Driver"/>
                <property name="url" value="jdbc:h2:mem:operaton;DB_CLOSE_DELAY=1000"/>
                <property name="username" value="sa"/>
                <property name="password" value=""/>
            </bean>
        </property>
    </bean>

    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <bean id="processEngineConfiguration" class="org.operaton.bpm.engine.spring.SpringProcessEngineConfiguration">
        <property name="processEngineName" value="default" />
        <property name="dataSource" ref="dataSource"/>
        <property name="transactionManager" ref="transactionManager"/>
        <property name="databaseSchemaUpdate" value="true"/>
        <property name="jobExecutorActivate" value="false"/>
    </bean>

    <!-- using ManagedProcessEngineFactoryBean allows registering the ProcessEngine with the BpmPlatform -->
    <bean id="processEngine" class="org.operaton.bpm.engine.spring.container.ManagedProcessEngineFactoryBean">
        <property name="processEngineConfiguration" ref="processEngineConfiguration"/>
    </bean>

    <bean id="repositoryService" factory-bean="processEngine" factory-method="getRepositoryService"/>
    <bean id="runtimeService" factory-bean="processEngine" factory-method="getRuntimeService"/>
    <bean id="taskService" factory-bean="processEngine" factory-method="getTaskService"/>
    <bean id="historyService" factory-bean="processEngine" factory-method="getHistoryService"/>
    <bean id="managementService" factory-bean="processEngine" factory-method="getManagementService"/>

</beans>
```
