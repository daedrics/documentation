---

title: 'Dashboard'
sidebar_position: 10

---

![Example img](./img/dashboard.png)Cockpit Dashboard

The dashboard of Cockpit provides a quick overview of running and historic operations as well as details about deployments.

At the top of the dashboard you can see a plugin with pie charts that display the amount of running process instances, open incidents and open human tasks.
By clicking on the number or a section of the pie chart, you are forwarded to the respective search with preselected query parameters.

On the right hand side, you see an overview of deployed process definitions, decision definitions, case definitions and the total number of deployments.

Additional [plugins](../cockpit/extend/plugins.md) can be added to the dashboard.

# Metrics

![Example img](./img/dashboard-metrics.png)Cockpit Dashboard Metrics

At the bottom of the dashboard, the metrics plugin displays graphs with metrics for executed activity instances, evaluated decision instances and executed jobs.
You can select to display the details for the current day, the current week or the current month. By clicking on the metrics graphs and dragging the cursor
in either direction, you can select a specified range of time. You can also modify the range of time by using the mouse wheel. By clicking on the link that
appears under the graph, you are forwarded to the respective search with preselected query parameters.


# Multi Engine

![Example img](./img/cockpit-multi-engine.png)Multiple Engines

If you are working with more than one engine, you can select the desired engine via a dropdown selection. Cockpit provides all information of the selected engine.
