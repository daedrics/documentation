---

title: 'Dashboard'
sidebar_position: 20

menu:
  main:
    identifier: "webapps-tasklist-dashboard"
    parent: "webapps-tasklist"

---

![Example img](./img/tasklist-dashboard-detail.png)Tasklist Dashboard

On the dashboard of Tasklist you see an overview of pending tasks. On the left side of the screen, an overview of the [filters][tasklist-filters] is displayed. On the upper right side of the screen, you can set a follow-up or due date, you can claim, unclaim and reassign tasks and you can add comments. Underneath that section, the embedded form is displayed (please note that external task forms cannot be displayed here), you can switch to the task history, you can see the diagram view or you can view the description of the user task.


# Toggle view

![Example img](./img/tasklist-toggle-view.png)Tasklist View

Tasklist offers you the option of toggling the view options on the dashboard. You can select to have a focus on the [filters][tasklist-filters], the [filter results][filter-results], the [task view][task-view] or to display the full dashboard. To do so, hit the respective *toggle view* button. At any time, you can choose to instantly set full focus on the task view by hitting the *expand* button.


# Start a process

![Example img](./img/tasklist-start-process.png)Start Process

To start a process instance via Tasklist, click on *Start process* in the header menu and select a process out of the displayed list of process definitions. If no process definitions are listed here, please verify that your process application is deployed correctly.

Depending on whether you have defined a [start form](../../user-guide/task-forms/index.md) for your process it will be displayed now. Otherwise you get a notification that no form has been defined for starting the process. In this case, a [generic start form](../../user-guide/task-forms/index.md#generic-task-forms) will be displayed and Tasklist will offer the option of adding variables to the process instance.


# Create a standalone task

![Example img](./img/tasklist-start-task.png)Standalone Tasks

Tasklist offers you the possibility of creating standalone tasks. To do so, click on the *Create task* button. In the screen that appears, you can define a name of the task, the assignee and you can add a description. Once you click on *Save*, the task is created.

Now the standalone task becomes visible in the filter results and can be handled. Variables can be added and the Task can be completed. Once the task has been completed, the data is flushed to the database and the task is no longer visible in Tasklist.


# Filter results

![Example img](./img/tasklist-filter-results.png)Filter Results

Here you can see an overview of all tasks for the selected [filter][tasklist-filters]. The filter with the lowest priority is displayed first. After selecting the appropriate filter, you will see an overview of all tasks, sorted by a specified criteria (by default it is sorted by the creation date). You can change the sorting of the tasks by clicking on the name of the sorting property. You can toggle between ascending and descending order.

You can sort by more than one property by adding additional properties with the *plus* button. The sorting is performed in a hierarchical manner (i.e., first by the property displayed first, then by the second, etc). You can also sort by the value of variables, which allows for use cases like sorting invoices by the value of their amount.

To start working on the task, simply select the task.


## Search for tasks

![Example img](./img/tasklist-task-search.png)Task Search

Above the filter results, you have the option of searching for user tasks within the selected filter results. To do so, click in the search box and select the parameters to search for. You can also begin typing to find the required parameter faster. Depending on the selected property, you have to specify the value of the property. Some properties also allow operators other than equal, e.g., `'like'`, which allows to search for a task where the entered value is a substring of the property value. The `'in'` operator allows you to provide a comma-separated list of possible values.

If you are searching for variables, you also have to enter the variable name you want to search for. If the filter you have selected has defined labels for variables, you can select the label of the variable as variable name. Otherwise (if there is no label definition for a variable), you have to enter the variable name to search for it. If you change the filter selection, the search will be performed on the selected filter and the results will be updated accordingly.

If you are searching for a variable of type string, which has a numeric, boolean or null value, you have to wrap the value in single quotes (e.g `'93288'` or `'NULL'`).

Furthermore, you can copy a link to the current search query to your clipboard by clicking on the <button class="btn btn-xs"><i class="glyphicon glyphicon-link"></i></button> button and you can save search queries to your local browser storage by clicking on the <button class="btn btn-xs"><i class="glyphicon glyphicon-floppy-disk"></i></button> button and inserting a name in the drop down menu that appears. You can then retrieve the search query by clicking on the <button class="btn btn-xs"><i class="glyphicon glyphicon-floppy-disk"></i></button> button and selecting the chosen name in the drop down menu.

### Case insensitive search
![Example img](./img/tasklist-case-insensitive.png)Task Search

Sometimes it is not relevant for your search whether the value is capitalized or not. For search queries for Process-, Task- and Case Variables, you can configure if the search should be performed case sensitive or case insensitive. A case insensitive search for `'Fruits Inc.'` will also return matches for values like `'fruits inc.'` or `'FRUITS Inc.'`.

When your search contains one of the Variables mentioned above, two checkboxes appear. You can choose the case handling for the variable name and variable value independently. The option applies to all variable queries within the same search.

### `IN` operator

![Example img](./img/tasklist-in-operator.png)IN Operator

Tasklist provides `IN` operator support for the following query criteria:

* Tenant ID
* Process instance ID

By default, the criteria defined in the search are linked together with a logical `AND` ([conjunctive normal form](https://en.wikipedia.org/wiki/Conjunctive_normal_form)).
Occasionally, you may search for multiple query criterion values. The `IN` operator allows searching for multiple values where any of the values match.

To use the `IN` operator, select a query criterion that supports the `IN` operator, and provide the values as a comma-separated list. To adjust the comma-separated list of values,
start editing by clicking on the value. You can expand the value in a modal dialog for easier editing by clicking on the <button class="btn btn-xs"><i class="glyphicon glyphicon-resize-full"></i></button> button.

![Example img](./img/tasklist-in-operator-modal.png)IN Operator Modal

# Task view

![Example img](./img/tasklist-task-view.png)Task Detail View

On the right section of the dashboard, you can see the task view. Here you can work on tasks and perform the following operational actions.


## Set due dates and follow-up dates

In the upper section of the task view, you can set a due date and follow-up date for the selected task. A due date can be set to determine when the task needs to be completed and a follow-up date can be set as a reminder or for monitoring purposes.


## Claim, unclaim, and reassign tasks

Within the task view, you can claim, unclaim and reassign tasks. To claim a task, simply select *Claim*. Unclaim a task by hitting the remove button next to the username of a claimed task and reassign a task to a different user by clicking on the username and inserting the username of the user you want to assign the task to. You can also assign tasks to user groups by clicking on *Add Groups*.


## Comments

In Tasklist you can add and view comments on specific tasks. After selecting a task from the [filter results][filter-results], click on *Add Comment* at the top of the [task view section][task-view] to add a comment to the selected task. The comments of a task can be viewed in the task history.


# Task Detail Tabs

![Example img](./img/tasklist-task-view-tabs.png)Detail Tabs

In the lower section of the task view there are several tabs which can be selected to display both the task form itself and additional information related to this user task.

* **Task form view** - The *Form* tab, which is selected by default, displays the task form (provided that the task form is an embedded, generated or generic task form). Here you can work on and complete the task.
* **Task history** - The *History* tab displays the history of this user task. Here you can see detailed information, such as the assignment history, updates to the due date and follow-up dates and claiming and unclaiming of tasks. Comments are also displayed here.
* **Diagram view** - The *Diagram* tab shows the diagram of the process definition. The current user task is highlighted in this diagram.
* **Task description** - Open the *Description* tab to inspect the User Task description. Have a look at the [BPMN 2.0 reference](../../reference/bpmn20/tasks/user-task.md#description) for more information about descriptions of tasks.


[task-view](#task-view)
[filter-results](#filter-results)
[tasklist-filters](../tasklist/filters.md)
