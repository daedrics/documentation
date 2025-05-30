---

title: 'Batch View'
sidebar_position: 50

menu:
  main:
    name: "Batch View"
    identifier: "user-guide-cockpit-batches-view"
    parent: "user-guide-cockpit-batch"
    description: "Display status of running and completed batches."

---

![Example img](./../img/batch.png)Batch View Page

The batch page displays the status of running and completed batches. It allows displaying details of a batch such as the start- and end-time or the number of remaining jobs. For failed jobs, it displays the error message and allows a retry or deletion of the job.

# Overview

On the left side of the page, two lists of running and finished batches are displayed. You can click on any entry to display details of the batch on the right side of the page.

## In Progress

For batches that are currently in progress, the ID, type, user, started time, number of failed jobs as well as the progress is displayed.
Please note that if a batch has failed jobs, the batch cannot be completed.
You have to go to the details view of the batch and resolve the failed jobs (i.e., retrying or deleting them).
Also, it can take some time after the progress is at 100% for the batch to actually finish.
See the [process engine section](../../../user-guide/process-engine/batch.md) for details.

### Search

![Example img](./../img/batch-search.png)Batch Search

Above the in progress batches table, you can search for batches which fulfill certain search criteria.
To do so, click in the search box and select the parameters to search for.
You can also begin typing to find the required parameter faster.
Depending on the selected property, you have to specify the value of the property.
You can combine multiple search pills to narrow down the search results.

## Completed

Below the currently running batches, completed batches along with their start- and end-time are displayed. Completed batches are only visible when the process engine [history level](../../../user-guide/process-engine/history/history-configuration.md#choose-a-history-level) is set to FULL.

# Batch Details

When selecting a batch either from in progress or completed ones, the details of the batch are displayed on the right side of the page.

## In Progress Batch Details

The displayed information corresponds to the response of the [REST API query](#getBatchStatistics).

If the batch is in progress and has failed jobs, the failed jobs and their exception message are displayed below the details table. You can access the complete stacktrace for every job by clicking on the `Full stack trace` link at the end of the exception message summary. Using the <button class="btn btn-xs"><i class="glyphicon glyphicon-repeat"></i></button> button you can increment the number of retries for this job. Using the <button class="btn btn-xs"><i class="glyphicon glyphicon-trash"></i></button> button you can delete this job. You can also increment the number of retries for all jobs by clicking the <button class="btn btn-xs"><i class="glyphicon glyphicon-repeat"></i></button> button above the details table.

You can suspend a batch that is in progress by clicking on the <button class="btn btn-xs"><i class="glyphicon glyphicon-pause"></i></button> button above the details table. Likewise, you can resume a suspended batch with the <button class="btn btn-xs"><i class="glyphicon glyphicon-play"></i></button> button.

Clicking the <button class="btn btn-xs btn-danger"><i class="glyphicon glyphicon-trash"></i></button> button above the details table deletes a running batch.

## Completed Batch Details

The displayed information corresponds to the response of the restref page="getBatch" text="REST API query" tag="Batch.

Clicking the <button class="btn btn-xs btn-danger"><i class="glyphicon glyphicon-trash"></i></button> button above the details table deletes the batch history.