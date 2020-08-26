# Lab 4: Analyze data across data sets

Information Server enables users to better understand their data. It uses pre-built and custom rules that will apply meaning and quality measurements, which are available for users of the data and interested parties.

DataStage Flow Designer enables users to create, edit, load, and run DataStage jobs which can be used to perform integration of data from various sources in order to glean meaningful and valuable information.

This lab will give you hands-on experience using Information Server's datastage flow designer and rule capabilities. In this lab, you will learn the following:

* How to join two data sets using DataStage Flow Designer to create a single data set
* How to create a data rule that runs on columns from each data set
* Apply the rules to the joined data set
* View data that does not apply to the rule

This lab is comprised of 2 sections. In the first section, we will see how to associate records within 2 data sets to create a single combined data set using DataStage Flow Designer. In the second section, we will apply a data rule on the combined data set and analyze the data that does not apply to the rule.

> Note: The merged data set is already available in DB2WH. If you wish to, you may skip section 1 and directly move on to section 2 where you can import this merged data set.

This lab consists of the following steps:

[Section 1: Join the data sets](#section-1-join-the-data-sets)
1. [Create a Transformation project](#1-create-a-transformation-project)
1. [Add database connection](#2-add-database-connection)
1. [Create the job](#3-create-the-job)
1. [Compile and run the job](#4-compile-and-run-the-job)
1. [View output](#5-view-output)

[Section 2: Create and run data rule](#section-2-create-and-run-data-rule)
1. [Import and view the data](#1-import-and-view-the-data)
1. [Create a data rule](#2-create-a-data-rule)
1. [Re-analyze and view results](#3-re-analyze-and-view-results)

## Section 1: Join the data sets

## Before you start

Before we start the lab, let's switch to the `iis-client` VM and launch `Firefox`. 

![Switch to iis-client](images/switch-to-iis-client.png)

Click on `Classic Launchpad` in the Bookmarks tab. The first time you try this out, you might see a certificate error. To get past it, click on `Advanced...` and then click `Accept the Risk and Continue`.

![Classic Launchpad](images/select-classic-launchpad.png)

Click on `DataStage Flow Designer`.

![Select DFD](images/select-dfd.png)

Login with the credentials `isadmin`/`inf0Xerver`.

![Log into DFD](images/log-into-dfd.png)

This brings up the `DataStage Flow Designer`. Click `OK`.

![DFD is up](images/dfd-is-up.png)

## 1. Create a Transformation project

* On the IBM DataStage Flow Designer, click on the `Projects` tab and click `+ Create`. In the modal that opens up, type in a name for the project and click `Create`.

![Create project](images/create-project.png)

The project takes a few minutes to be created and once ready, it will be visible on the `Projects` tab.

![Project created](images/created-project.png)

* Click on the tile for your newly created project. In the modal that opens up, verify that the name of your project is provided as the `Project Name` and click `OK` to switch the project.

![Switch project](images/switch-project.png)

## 2. Add database connection

The input tables - `EMP` (containing employee data) and `DEPT` (containing department data) - are already loaded in Db2 Warehouse. Let's add a Db2 warehouse instance as a `Connection` in DataStage.

* Click on the `Connections` tab and then click `+ Create` to add a new connection.

![Create connection](images/create-connection.png)

* Provide the following connection details and click `OK`. Click `Save` on the new modal that pops up.

```ini
Name: DB2WH
Connector type: JDBC
URL: jdbc:db2://db2w-kzwbsid.us-east.db2w.cloud.ibm.com:50001/BLUDB:sslConnection=true;
Username: bluadmin
Password: ****************
```

![Add connection](images/add-connection.png)

A tile for the new connection will now be displayed in the `Connections` tab.

![Created connection](images/created-connection.png)

## 3. Create the job

* Click on the `Jobs` tab and then click `+ Create`. Click `Parallel Job`.

![Create parallel job](images/create-parallel-job.png)

A new tab with the name `Job_1*` opens up where you can now start designing the parallel job.

The first step is to load the input tables `DEPT` and `EMP` into DataStage. The `WORKDEPT` column of the `EMP` table is the same as the `DEPTNO` column of the `DEPT` table.

![Data set definitions](images/data-set-definitions.png)

* First, drag a ***Connection*** connector to the canvas. In the modal that opens up, select the `DB2WH` connection that was created earlier and click `Next`.

![Create connection - select connection](images/connection-stage-connection.png)

* On the next screen, select the `BLUADMIN` schema and click `Next`.

![Create connection - select schema](images/connection-stage-schema.png)

* On the next screen, select the `DEPT` table and click `Next`.

![Create connection - select table](images/connection-stage-table.png)

* On the next screen, click `Add to Job`.

![Create connection - add to job](images/connection-stage-add-to-job.png)

* Drag another ***Connection*** connector to the canvas and repeat the steps given above but this time, select the `EMP` table instead. Once you complete the steps, you should see the two ***Connection*** connectors on the canvas.

![Create connection - completed](images/create-connection-completed.png)

The `EMP` table uses the `WORKDEPT` column to identify the department number whereas the `DEPT` table uses the `DEPTNO` column. Use a ***Transformer*** stage to modify the output of the `EMP` table by changing the name of the `WORKDEPT` column to `DEPTNO`. This is needed for a future step where we will ***Join*** the two tables.

* Drag and drop a ***Transformer*** stage next to the ***Connection*** connector for the `EMP` table. Provide the output of the `EMP` table ***Connection*** connector as the input to the ***Transformer*** stage. For this, click on the little blue dot on the right side of the ***Connection*** connector and drag the mouse pointer to the ***Transformer*** stage.

**NOTE**: For another method to connect the ***Connection*** connector to the ***Transformation*** stage, click on the ***Connection*** connector to select it, then drag and drop the ***Transformation*** stage. The ***Transformation*** stage will automatically be connected to the ***Connection*** connector.

![Add transformer for EMP](images/add-transformer-for-emp.png)

* Drag and drop a ***Join*** stage to the canvas and provide the output of the ***Transformer*** stage as the input to this ***Join*** stage.

![Add join stage](images/add-join-stage.png)

* Double click on the ***Transformer*** stage to open up the stage page. Go to the `Outputs` tab and in the table find the entry for the `WORKDEPT` column. Double click on the `WORKDEPT` value under the `Column name` column and replace the text with `DEPTNO`. Click `OK`.

![Transformer - updates on Output tab](images/transformer-updates-on-output-tab.png)

* Both the tables now have a column called `DEPTNO` which can be used to join the tables. Provide the output of the `DEPT` table ***Connection*** connector as the second input to the ***Join*** stage. Double clicking the ***Join*** stage brings up the stage page where you can verify that the `DEPTNO` is being used as the `JOIN KEY` and the `Join Type` is `Inner`. Click `OK`.

![Connect DEPT to Join](images/connect-dept-to-join.png)

* Drag and drop a ***Connection*** connector to the canvas. In the modal that pops up in the screen, check the box for `Add selected connection as target` and click `Add to Job`. Provide the output of the ***Join*** stage as the input to this connector.

![Add output connection](images/add-output-connection.png)

* Double click on the ***Connection*** connector to open up the Properties page on the right. Verify that the URL, username and password are already populated. 

![Connection properties - 1 - already populated](images/connection-properties-1-already-populated.png)

* Scroll down and under the `Usage` section, provide the *Table name* as `<user>_DEPTEMP` where \<user\> is your name, and update the *Table action* to `Replace`. Click `OK`.

![Connection properties - 2 - table name action](images/connection-properties-2-table-name-action.png)

## 4. Compile and run the job

* Click the `Save` icon to save the job. If you wish to, you can provide a different name for the job in the modal that pops up. Click `Save`. Once the job is saved, click on the `Compile` icon to compile it. If compilation is successful, you should see a green check mark and the message `Compiled successfully` displayed on the screen.

![Save compile](images/save-compile.png)

* Click the `Run` icon to run the job. In the modal that opens up, click `Run`.

![Run job](images/run-job.png)

## 5. View output

After a successful run, the results will be stored within the DB2WH connection in the `BLUADMIN` schema. Because we specified the *Table action* as `Replace` in the ***Connection*** connector that represents the output, each subsequent run of the job will delete all existing records in the table and replace them with the new output.

* To view the results of the job, double click on the ***Connection*** connector that represents the output. This will open up the Properties page on the right. Click on `View Data` to bring up a modal that shows the contents of the `<user>_DEPTEMP` table in which the output was stored.

![view table data](images/view-table-data.png)

**This marks the end of Section 1 of this lab.**

## Section 2: Create and run data rule

## 1. Import and view the data

Switch to the `iis-client` VM and launch `Firefox`. 

![Switch to iis-client](images/switch-to-iis-client.png)

* Click on the `Launchpad` bookmark. When the Information Server launchpad shows up click on the `Information Governance Catalog New` tile.

![1-iis-launchpad-new](images/1-iis-launchpad-new.png)

* Log in with the username `isadmin` and password `inf0Xerver`.

![2-gc-login](images/2-gc-login.png)

* The overview page will appear.

![3-gc-landing](images/3-gc-landing.png)

* Click on the `Connections` tab and click `+ Create connections`. 

![4-connections](images/4-connections.png)

* On the next screen, provide the following details and click `Test connection` to test the connection. Once successful, click `Save connection`.

```ini
Name: DB2WH
Choose connection: Db2
JDBC URL: jdbc:db2://db2w-kzwbsid.us-east.db2w.cloud.ibm.com:50001/BLUDB:sslConnection=true;
Username: bluadmin
Password: ****************
```

![5-add-connection](images/5-add-connection.png)

* Now locate the tile for the connection that was just added, click on the kebab icon (⋮) on the tile and click `Discover`.

![6-discover](images/6-discover.png)

* Click on `Browse`.

![7-browse-discovery-root](images/7-browse-discovery-root.png)

* Expand `db2`. Expand `BLUADMIN` and select the `<user>_DEPTEMP` table where \<user\> is your name. This is the table that was created in section 1 above. If you skipped section 1, then you can select the `SANDHYA_DEPTEMP` table. Click `OK`.

![8-select-table](images/8-select-table.png)

* Under `Discovery options`, select `Analyze data quality`. This will automatically check the boxes for `Analyze columns` and `Assign terms`.

![9-discovery-options](images/9-discovery-options.png)

* Scroll to the bottom and select the `Host` as `IIS-SERVER.IBM.DEMO` and the `Workspace` as `UGDefaultWorkspace`. Click `Discover`.

![10-discover](images/10-discover.png)

* The discovery process will take some time. Once the assets are discovered, the analysis process will begin. Once that completes, you will be able to see what percentage of records were successfully analyzed.

![11-discovery-complete](images/11-discovery-complete.png)

* Now let us go and have a look at the data. Go to `Quality` tab and click on the tile for `UGDefaultWorkspace`.

![12-open-workspace](images/12-open-workspace.png)

* The workspace overview will load. Take a few moments to browse the graphics on the page and click on `Data sets` link to view the data in this exercise.

![13-workspace-overview](images/13-workspace-overview.png)

* Before we create new rules, let's look at the data set that will be used in this lab. Click on the tile for `<user>_DEPTEMP`. 

![14-data-sets](images/14-data-sets.png)

* Click on the `Columns` tab to view findings from the analyzer. It found many things when the data was imported, like maximum and minimum values, distinct values, format, and uniqueness.

![15-deptemp](images/15-deptemp.png)

We're now ready to create our first data rule!

## 2. Create a data rule

* From the `UGDefaultWorkspace` workspace, click on the `Data rules` tab.

![16-data-rules](images/16-data-rules.png)

* Expand `Published Rules` > `08 Validity and Completeness` > `Valid Value Combination`. Find the rule for `IfFieldaIsXThenFieldbGtQty`. Click on the `...` overflow menu on the right and select `Manage in workspace`.

![17-manage-in-workspace](images/17-manage-in-workspace.png)

* The rule should now be available under `All`. We will now edit the rule. If you wish to rename the rule, you will first need to `Copy` the rule and you will be provided with the option to rename the rule. Click on the `...` overflow menu of the rule and select `Edit`.

![18-edit-rule](images/18-edit-rule.png)

* Switch to the `Rule logic` tab and update the rule to say `IF DEPTNAME = 'OPERATIONS' THEN SALARY > 36000`.

![19-update-rule-logic](images/19-update-rule-logic.png)

* Next, switch to the `Rule testing` tab. Here you need to bind the variables in the rule logic to specific columns in the data source. Select the `salary` variable in the left table and select the `SALARY` column under *Available data sources*. It should be under `IIS-SERVER.IBM.DEMO` > `db2` > `BLUADMIN` > `<user>_DEPTEMP`. Click on `+ Bind`. The value `<user>_DEPTEMP.SALARY` will now be shown under *Implemented bindings* for the `salary` variable.

* Uncheck the `salary` variable, check the `deptname` variable and bind it with the `DEPTNAME` column under *Available data sources*. As in case of the `SALARY` column, it should be under `IIS-SERVER.IBM.DEMO` > `db2` > `BLUADMIN` > `<user>_DEPTEMP`. Click on `+ Bind`. The value `<user>_DEPTEMP.DEPTNAME` will now be shown under *Implemented bindings* for the `salary` variable.

![20-bind-columns.png](images/20-bind-columns.png)

* Scroll down and click on `Test` to test out the rule. You will see a message at the top of the screen that says the test results can be viewed once ready. When the message disappears, go to the `Rule test results` tab to view the test results.

![21-test-the-rule](images/21-test-the-rule.png)

* You can see that of the 42 total rows, 39 met the rule, and 3 did not. Click on `Did not meet rule conditions` to view the 3 rows. You can see that these rows have `DEPTNAME` = `OPERATIONS` but have `SALARY` < `36000`, and therefore they did not match the rule conditions. Click `Save` to save the rule.

![22-save-the-rule](images/22-save-the-rule.png)

* When you are brought back to the `Data rules` tab, you'll notice that the new rule has an error. We need to publish the rule. To do so navigate to the right to show the `⋯` menu. Choose the `Publish` option from the menu.

![23-publish-the-rule](images/23-publish-the-rule.png)

* In the modal that pops up, click `Publish` to confirm the publish action.

![24-confirm-publish](images/24-confirm-publish.png)

## 3. Re-analyze and view results

* Go back to the `UGDefaultWorkspace` workspace and click on the `Data sets` link.

![13-workspace-overview](images/13-workspace-overview.png)

* Click on the `<user>_DEPTEMP` data set.

![14-data-sets](images/14-data-sets.png)

* We can now apply the newly created rule by switching to the `Rules (0)` tab and clicking the `+ Add rule` button.

![25-add-rule](images/25-add-rule.png)

* Choose the `IfFieldaisXThenFieldbGtQty` rule under `All` and click `Next`.

![26-select-rule](images/26-select-rule.png)

* As before, we need to bind the rule variables to specific columns in our data set. Select the `salary` variable in the left table and select the `SALARY` column under *Available data sources*. Click on the `+ Bind` button. Once bound, select the `deptname` variable and bind it with the `DEPTNAME` column under *Available data sources*. Once both the variables are bound with the right columns, click `Next`.

![27-bind](images/27-bind.png)

* This time, we don't need to test the rule. Simply click `Save`.

![28-save](images/28-save.png)

* You should now see the rule in the data set view. Click on the `Analyze` button to restart the analysis with the new rule.

![29-analyze](images/29-analyze.png)

* In the modal that pops up, click `Analyze` to confirm analysis.

![30-analyze-confirm](images/30-analyze-confirm.png)

* The analysis will take a few minutes. You may need to refresh your browser a few times. You will see the state go from `Running` to `Successful` when analysis is complete.

![31-analysis-running](images/31-analysis-running.png)

* Once analysis has completed successfully, go to the `Data quality` tab. You'll see that the new rule has six findings - three against DEPTNAME and three against SALARY. Click on either DEPTNAME or SALARY to view the exceptions.

![32-findings](images/32-findings.png)

Scrolling to the right you'll see that the three entries shown have `DEPTNAME` = `OPERATIONS` but have `SALARY` less than `36000`, which goes against our rule.

![33-findings-view](images/33-findings-view.png)

**CONGRATULATIONS!!** You have completed this lab!