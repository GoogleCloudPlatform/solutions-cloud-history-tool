# Cloud History Solution

## Overview

Large production systems may consist of hundreds of dynamic components such
as instances, disks, networks, and firewall rules. These components are
transient in nature, and often it is important to understand how they change
over time. For example, a time series view of how many instances with a given
tag were created and deleted in a given week can help you understand usage
patterns of your cloud deployment.

This solution aims to help developers of large systems get answers to
history-related questions.

This solution builds on the *[Data Pipeline Solution]* to create a workflow that
captures snapshots of Compute Engine deployment configurations into BigQuery.
By using the *[App Engine Cron Service]*, deployment configurations
can be captured as time series data and queried using BigQuery. A sample
queries *[cookbook]* are provided for you to get started on the
analysis. The workflow is best illustrated with the following diagram:

![Cloud History Architecture](/cloud_history_arch.png)

### Copyright

Copyright 2014 Google Inc. All Rights Reserved.

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at

[http://www.apache.org/licenses/LICENSE-2.0]

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.

### Disclaimer

This is not an official Google product.

## Installation Guide

### 1. Install Data Pipeline

Follow the *[Data Pipeline Installation Guide]* to install the
*[Data Pipeline Solution]*.

### 2. Create the Cloud History pipelines

Two sample pipelines are provided for logging Compute Engine instance, disk, and
zone operations data into BigQuery. The instance and disk data is processed by
one pipeline. The zone operations data is processed by a separate pipeline.
Instance and disk data is transient in nature,
so the pipeline should be scheduled to run more frequently. The data
for zone operations is not transient, so the pipleline does not have to run
as often.

#### Compute Engine instance and disk pipeline

The *[cloud_history_pipeline.json]* pipeline included in Data Pipeline
reads the current Google Compute Engine instance and disk data using the
Compute Engine API interface. The data is then transformed to a JSON format
that is compatible with BigQuery and loaded into BigQuery.

You can find the file in the `app/static/examples` directory.
Follow the Data Pipeline instruction to create the pipeline for
instance/disk data.

#### Compute Engine zone operations pipeline

The *[operations_history_pipeline.json]* sample pipeline included in this
package reads the current Google Compute Engine zone operations data using
the Compute Engine API interface. The data is then transformed to a JSON
format that is compatible with BigQuery before loading it into BigQuery.

You can find the file in the `app/static/examples` directory.
Follow the Data Pipeline instruction to create the pipeline for
zone operations data.

### 3. Set up the App Engine cron service

Copy the 'Run URL' link for each pipeline that you created.
The url should be in the following format. The pipeline
variables, if defined, are appended as query parameters.

/run/cloud-history/&lt;unique id&gt;

Change the directory to where you downloaded the data pipeline
package. Create a cron.yaml file in your `app` directory
(along with an app.yaml).
Use the following as a template and fill in your values.

```yaml
cron:
- description: instance and disk data
  url: <Run URL for instance and disk data>
  schedule: every 15 minutes
  target: <YOUR APP VERSION>

- description: operations data
  url: <Run URL for operations data>
  schedule: every day 06:00
  timezone: America/Los_Angeles
  target: <YOUR APP VERSION>
```

Update the cron configuration while you are still in your `app` directory:

```sh
appcfg.py update_cron . --oauth2
```

## Cloud History Analysis

After the pipeline is set up, you can use the *[BigQuery Web User Interface]*
for analysis. Make sure that the project where you loaded the data is the
current project. You should see the cloud_history dataset. Within the
dataset, there should be three tables: Disks, Instances, and zoneoperations.

Use the sample query *[cookbook]* to get started with timeline analysis. 


[App Engine Cron Service]: https://developers.google.com/appengine/docs/python/config/cron
[BigQuery Web User Interface]: https://bigquery.cloud.google.com
[Data Pipeline Installation Guide]: https://github.com/GoogleCloudPlatform/Data-Pipeline#installation-guide 
[Data Pipeline Solution]: https://github.com/GoogleCloudPlatform/Data-Pipeline
[cloud_history_pipeline.json]: https://github.com/GoogleCloudPlatform/Data-Pipeline/blob/master/app/static/examples/cloud_history_pipeline.json
[operations_history_pipeline.json]: https://github.com/GoogleCloudPlatform/Data-Pipeline/blob/master/app/static/examples/operations_history_pipeline.json
[cookbook]: https://github.com/GoogleCloudPlatform/Data-Pipeline/blob/master/app/help/cloudhistory.md#-sample-query-cookbook
