---

title: 'User Assignment'
sidebar_position: 40

menu:
  main:
    identifier: "webapps-tasklist-user-assignment"
    parent: "webapps-tasklist"

---

For users to be able to work on the tasks they should work on, they must be able to find these tasks. Enabling users to find _"their"_ tasks requires the correct interaction between the initial assignment of the task and the filters configured in Tasklist.

This works as follows:

1. Initial Assignment: when a task is created, it is initially assigned to a user or group according to the configuration in the BPMN process (or the CMMN case).
2. Building Filters: [filters][filter] then allow users to find tasks which are assigned to them or to the groups they belong to.
3. Claiming group tasks: if a task is not directly assigned to a given user, the user must claim the task before working on it.

# Implementing Initial Assignment

![Example img](./img/tasklist-task-form-modeler.png)User Task Assignment

You can read up on how to implement the inital user assignment for BPMN User Tasks and CMMN Human Tasks in the corresponding reference sections:

* [Implementing user assignments for BPMN User Tasks][bpmn-user-assignment]
* [Implementing user assignments for CMMN Human Tasks][cmmn-user-assignment]

# Claiming a task in Tasklist

In Tasklist, a user can ony work on a task (i.e., filling in the task form) if the task is assigned to that user. This means that a user must `claim` a task if it is not yet assigned to him.
Claiming a task sets the assignee of the task to the user who claimed the task.

See the [Claiming, unclaiming and reassigning tasks](../tasklist/dashboard.md#claim-unclaim-and-reassign-tasks) section for more information.

[bpmn-user-assignment]: ../../reference/bpmn20/tasks/user-task.md#user-assignment
[cmmn-user-assignment]: ../../reference/cmmn11/tasks/human-task.md#user-assignment
[filter]: filters.md
