---

title: 'None Events'
sidebar_position: 20

menu:
  main:
    identifier: "bpmn-ref-events-none-events"
    parent: "bpmn-ref-events"
    description: "Events without a specified trigger and no specified behavior."
---

None events are unspecified events, also called 'blank' events. For instance, a 'none' start event technically means that the trigger for starting the process instance is unspecified. This means that the engine cannot anticipate when the process instance must be started. The none start event is used when the process instance is started through the API by calling one of the `startProcessInstanceBy...` methods.

```java
ProcessInstance processInstance = runtimeService.startProcessInstanceByKey('invoice');
```

Note: a subprocess must always have a none start event.

<div data-bpmn-diagram="../bpmn/event-none"></div>


# None End Event

A 'none' end event means that the result thrown when the event is reached is unspecified. As such, the engine will not do anything besides ending the current path of execution. The XML representation of a none end event is the normal end event declaration, without any sub-element (other end event types all have a sub-element declaring the type).

```xml
<endEvent id="end" name="my end event" />
```

# Intermediate None Event (throwing)

The following process diagram shows a simple example of an intermediate none event, which is often used to indicate some state achieved in the process.

<div data-bpmn-diagram="../bpmn/event-none-intermediate" ></div>


This can be a good hook to monitor some KPI's, basically by adding an execution listener

```xml
<intermediateThrowEvent id="noneEvent">
  <extensionElements>
    <operaton:executionListener class="org.operaton.bpm.engine.test.bpmn.event.IntermediateNoneEventTest$MyExecutionListener" event="start" />
  </extensionElements>
</intermediateThrowEvent>
```

You can add some own code to the execution listener to maybe send some event to your BAM tool or DWH. The engine itself doesn't do anything in the event, it just passes through it.


# Operaton Extensions

<table class="table table-striped">
  <tr>
    <th>Attributes</th>
    <td>
      <a href="../reference/bpmn20/custom-extensions/extension-attributes.md#asyncbefore">operaton:asyncBefore</a>,
      <a href="../reference/bpmn20/custom-extensions/extension-attributes.md#asyncafter">operaton:asyncAfter</a>,
      <a href="../reference/bpmn20/custom-extensions/extension-attributes.md#exclusive">operaton:exclusive</a>,
      <a href="../reference/bpmn20/custom-extensions/extension-attributes.md#jobpriority">operaton:jobPriority</a>
    </td>
  </tr>
  <tr>
    <th>Extension Elements</th>
    <td>
      <a href="../reference/bpmn20/custom-extensions/extension-elements.md#failedjobretrytimecycle">operaton:failedJobRetryTimeCycle</a>,
      <a href="../reference/bpmn20/custom-extensions/extension-elements.md#inputoutput">operaton:inputOutput</a>
    </td>
  </tr>
  <tr>
    <th>Constraints</th>
    <td>
      The <code>operaton:exclusive</code> attribute is only evaluated if the attribute
      <code>operaton:asyncBefore</code> or <code>operaton:asyncAfter</code> is set to <code>true</code>
    </td>
  </tr>
</table>
