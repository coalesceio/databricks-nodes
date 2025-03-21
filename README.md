# Coalesce Databricks Base Node Types Package

The Coalesce Base Node Types Package includes:

* [Work](#work)
* [Persistent Stage](#persistent-stage)
* [Dimension](#dimension)
* [Fact](#fact)
* [View](#view)
* [Code](#code)

---

## Work

The Coalesce work node is a versatile node that allows you to develop and deploy a Work table/view in Databricks.

A Work node serves as an intermediary object and is commonly employed to store raw data before undergoing the crucial phases of transformation and loading into the main tables of the data warehouse.

This pivotal step ensures that the raw data is processed and structured effectively.

### Work Node Configuration

The Work node type has two configuration groups:

* [Node Properties](#work-node-properties)
* [Options](#work-options)

![Fact_config](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/6f863c3b-3abd-4318-bd7e-ed50a829f911)

#### Work Node Properties

| **Property** | **Description** |
|----------|-------------|
| **Storage Location** | Storage Location where the WORK will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/> If FALSE the node will not be deployed or will be dropped during redeployment |

#### Work Options

You can create the node as:

* [Table](#work-options-table)
* [View](#work-options-view)

##### Work Options Table

![Work_options_table1](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/f5643f40-fb37-4182-a11b-577bdc3e8f8d)

| **Property** | **Description** |
|---------|-------------|
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>- **INSERT**: Individual insert for each source<br/>**False**: Single source node or multiple sources combined using a join |
| **Truncate Before** | Toggle: True/False<br/>This determines whether a table will be overwritten each time a task executes. **True**: Uses INSERT OVERWRITE<br/>**False**: Uses INSERT to append data |
| **Enable tests** | Toggle: True/False<br/>Determines if tests are enabled |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |
| **Order By** | Toggle: True/False<br/>**True**: Sort column and sort order drop down are visible and are required to form order by clause<br/>**False**: Sort column and sort order drop down are invisible |
| **Pre-SQL** | SQL to execute before data insert operation |
| **Post-SQL** | SQL to execute after data insert operation |

##### Work Options View

![Work_options_view1](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/7881c46e-0424-46ca-9a89-d111b5dbc379)

| **Setting** | **Description** |
|---------|-------------|
| **Override Create SQL** | Toggle: True/False<br/>**True**: Customized Create SQL specified in the Create SQL space is executed. All other options are invisible<br/>**False**: Create view SQL based on options chosen are framed and executed |
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |

### Work Joins

Join conditions and other clauses can be specified in the join space next to mapping of columns in the UI.

![work_join](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/2dc81bb8-2285-46e1-8b93-ce7082800fc5)

> 📘 **Specify Group by and Order by Clauses**
>
> You should specify group by and order by clauses in this space if you are not opting for the group by all and order by provided in OPTIONS config.

### Work Deployment

#### Work Initial Deployment

When deployed for the first time into an environment the Work node of materialization type table will execute the below stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Work Table** | This will execute a CREATE OR REPLACE statement and create a table in the target environment |
| **Create Work View** | This will execute a CREATE OR REPLACE statement and create a view in the target environment |

#### Work Redeployment

After the WORK node with materialization type table has been deployed for the first time into a target environment, subsequent deployments may result in either altering the WORK Table or recreating the WORK table.

#### Altering the Work Tables

A few types of column or table changes will result in an ALTER statement to modify the Work Table in the target environment, whether these changes are made individually or all together:

* Changing table names
* Dropping existing columns
* Altering column data types
* Adding new columns

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table\| Alter Column \| Delete Column \| Add Column \| Edit table description** | Alter table statement is executed to perform the alter operation |
| **Swap Cloned Table** | Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost |
| **Delete Table** | Drops the internal table |

#### Recreating the Work Tables

If any of the following change are detected, then the table will be recreated using a CREATE or REPLACE.

* Join clause
* Adding transformation
* Changes in configuration like adding distinct, group by, or order by

One of the following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Table** | Creates a new table |
| **Replace Table** | Replaces an existing table|

#### Recreating the Work Views

The subsequent deployment of Work node of materialization type view with changes in view definition, adding table description or renaming view results in deleting the existing view and recreating the view.

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Removes existing view |
| **Create View** | Creates new view with updated definition |

### Work Undeployment

If a Work Node of materialization type table is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the WorkTable in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Databricks is dropped |

If a Work Node of materialization type view is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the WorkView in the target environment will be dropped.

The stage executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Drops the existing Work view from target environment |

---

## Persistent Stage 

The Coalesce Persistent Stage Nodes element, serving as an intermediary object, is frequently utilized to maintain data persistence across multiple execution cycles.

It plays a crucial role in tracking the historical changes of columns linked to business keys.

This functionality is particularly beneficial when the objective is to retain raw data for prolonged durations.

### Persistent Stage Node Configuration

The Persistent node type has two configuration groups:

* [Node Properties](#persistent-stage-node-properties)
* [Options](#persistent-stage-options)

![Fact_config](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/6f863c3b-3abd-4318-bd7e-ed50a829f911)

#### Persistent Stage Node Properties

| **Property** | **Description** |
|----------|-------------|
| **Storage Location** | Storage Location where the WORK will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/> If FALSE the node will not be deployed or will be dropped during redeployment |

#### Persistent Stage Options

![pstage_options1](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/3914feb3-1db7-452b-8590-7c657b99c0eb)

| **Option** | **Description** |
|---------|-------------|
| **Create As** | Table is the only option at this time |
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Business key** | Required column for both Type 1 and Type 2 |
| **Change tracking** | Required column for Type 2 |
| **Truncate Before** | Toggle: True/False<br/>This determines whether a table will be overwritten each time a task executes.<br/> **True**: Uses INSERT OVERWRITE<br/>**False**: Uses INSERT to append data |
| **Enable tests** | Toggle: True/False<br/>Determines if tests are enabled |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |
| **Order By** | Toggle: True/False<br/>**True**: Sort column and sort order drop down are visible and are required to form order by clause<br/>**False**: Sort column and sort order drop down are invisible |
| **Pre-SQL** | SQL to execute before data insert operation |
| **Post-SQL** | SQL to execute after data insert operation |

### Persistent Stage Joins

Join conditions and other clauses can be specified in the join space next to mapping of columns in the UI.

![pstage_join](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/84ce06c9-103c-4700-ad3d-2e8222995b31)

> 📘 **Specify Group by and Order by Clauses**
>
> You should specify group by and order by clauses in this space if you are not opting for the group by all and order by provided in OPTIONS config.

### Persistent Stage Deployment

#### Persistent Stage Initial Deployment

When deployed for the first time into an environment the Persistent node will execute the below stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Persistent Table** | This will execute a CREATE OR REPLACE statement and create a table in the target environment |

#### Persistent Stage Redeployment

After the Persistent node has been deployed for the first time into a target environment, subsequent deployments may result in either altering the Persistent Table or recreating the Persistent table.

#### Altering the Persistent Tables

A few types of column or table changes will result in an ALTER statement to modify the Persistent Table in the target environment, whether these changes are made individually or all together:

* Changing table names
* Dropping existing columns
* Altering column data types
* Adding new columns

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table\| Alter Column \| Delete Column \| Add Column \| Edit table description** | Alter table statement is executed to perform the alter operation |
| **Swap Cloned Table** | Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost |
| **Delete Table** | Drops the internal table |

### Recreating the Persistent Tables

If any of the following change are detected, then the table will be recreated using a CREATE or REPLACE.

* Join clause
* Adding transformation
* Changes in configuration like adding distinct, group by, or order by

One of the following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Table** | Creates a new table |
| **Replace Table** | Replaces an existing table|

### Persistent Stage Undeployment

If a Persistent Node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Persistent Table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Drops table |
| **Drop Table or View** | Removes the table |

---

## Dimension

The Coalesce Dimension UDN is a versatile node that allows you to develop and deploy a Dimension table in Snowflake.

A dimension table or dimension entity is a table or entity in a star, Snowflake, or starflake schema that stores details about the facts. Dimension tables describe the different aspects of a business process.

### Dimension Node Configuration

The Dimension node type has two configuration groups:

* [Node Properties](#dimension-node-properties)
* [Options](#dimension-options)

![Fact_config](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/6f863c3b-3abd-4318-bd7e-ed50a829f911)

#### Dimension Node Properties

| **Property** | **Description** |
|----------|-------------|
| **Storage Location** | Storage Location where the WORK will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/> If FALSE the node will not be deployed or will be dropped during redeployment |

#### Dimension Options

##### Dimension Table Options

![Dimension](https://github.com/user-attachments/assets/8f65ee5c-ce09-457e-b4de-9e6c94038795)

| **Options** | **Description** |
|---------|-------------|
| **Create As** | Table or View |
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Business key** | Required column for both Type 1 and Type 2 Dimensions |
| **Change tracking** | Required column for Type 2 Dimension |
| **Truncate Before** | Toggle: True/False<br/>This determines whether a table will be overwritten each time a task executes. **True**: Uses INSERT OVERWRITE<br/>**False**: Uses INSERT to append data |
| **Enable tests** | Toggle: True/False<br/>Determines if tests are enabled |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |
| **Order By** | Toggle: True/False<br/>**True**: Sort column and sort order drop down are visible and are required to form order by clause<br/>**False**: Sort column and sort order drop down are invisible |
| **Pre-SQL** | SQL to execute before data insert operation |
| **Post-SQL** | SQL to execute after data insert operation |

##### Dimension View Options

![Work_options_view1](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/7881c46e-0424-46ca-9a89-d111b5dbc379)

| **Options** | **Description** |
|---------|-------------|
| **Override Create SQL** | Toggle: True/False<br/>**True**: Customized Create SQL specified in the Create SQL space is executed. All other options are invisible<br/>**False**: Create view SQL based on options chosen are framed and executed |
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Business key** | Required column for both Type 1 and Type 2 Dimensions |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |

### Dimension Joins

Join conditions and other clauses can be specified in the join space next to mapping of columns in the UI.

![Dimension_join](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/5c3df3b0-f56d-4276-a51f-22364206b3c3)

> 📘 **Specify Group by and Order by Clauses**
>
> You should specify group by and order by clauses in this space if you are not opting for the group by all and order by provided in OPTIONS config.

### Dimension Deployment

#### Dimension Initial Deployment

When deployed for the first time into an environment the Dimension node of materialization type table will execute the Create Dimension Table stage.

| **Stage** | **Description** |
|-----------|----------------|
| **Create Dimension Table** | This will execute a CREATE OR REPLACE statement and create a table in the target environment |
| **Create Dimension View** | This will execute a CREATE OR REPLACE statement and create a view in the target environment |

#### Dimension Redeployment

After the Dimension node of materialization type table has been deployed for the first time into a target environment, subsequent deployments may result in either altering the Dimension Table or recreating the Dimension table.

#### Altering the Dimension Tables

A few types of column or table changes will result in an ALTER statement to modify the Dimension Table in the target environment, whether these changes are made individually or all together:

* Changing table names
* Dropping existing columns
* Altering column data types
* Adding new columns

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table\| Alter Column \| Delete Column \| Add Column \| Edit table description** | Alter table statement is executed to perform the alter operation |
| **Swap Cloned Table** | Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost |
| **Delete Table** | Drops the internal table |

#### Recreating the Dimension Tables

If any of the following change are detected, then the table will be recreated using a CREATE or REPLACE.

* Join clause
* Adding transformation
* Changes in configuration like adding distinct, group by, or order by

One of the following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Table** | Creates a new table |
| **Replace Table** | Replaces an existing table|

#### Recreating the Dimension Views

Any of the following changes to views will result in deleting and recreating the Dimension view.

* View defintion
* Adding table description
* Renaming view results

### Dimension Undeployment

If a Dimension Node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Dimension Table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Databricks is dropped |

If a Dimension Node of materialization type view is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Dimension View in the target environment will be dropped.

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Drops the existing Dimension view from target environment. |

## Fact

The Coalesce Fact UDN is a versatile node that allows you to develop and deploy a Fact table in Databricks.

A fact table or a fact entity is a table or entity in a star or Databricks schema that stores measures that measure the business, such as sales, cost of goods, or profit. Fact tables and entities aggregate measures, or the numerical data of a business.

### Fact Node Configuration

The Fact node has two configuration groups:

* [Node Properties](#fact-node-properties)
* [Options](#fact-options)

![Fact_config](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/6f863c3b-3abd-4318-bd7e-ed50a829f911)

#### Fact Node Properties

| **Properties** | **Description** |
|----------|-------------|
| **Storage Location** | Storage Location where the WORK will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/> If FALSE the node will not be deployed or will be dropped during redeployment |

#### Fact Options

![fact_options](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/fe15f4d1-fccc-4522-a75b-71682a8ef493)

| **Options** | **Description** |
|---------|-------------|
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Business key** | Required column for Fact table creation |
| **Truncate Before** | Toggle: True/False<br/>This determines whether a table will be overwritten each time a task executes. **True**: Uses INSERT OVERWRITE<br/>**False**: Uses INSERT to append data |
| **Enable tests** | Toggle: True/False<br/>Determines if tests are enabled |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |
| **Order By** | Toggle: True/False<br/>**True**: Sort column and sort order drop down are visible and are required to form order by clause<br/>**False**: Sort column and sort order drop down are invisible |
| **Pre-SQL** | SQL to execute before data insert operation |
| **Post-SQL** | SQL to execute after data insert operation |

#### Fact View

![Work_options_view1](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/7881c46e-0424-46ca-9a89-d111b5dbc379)

| **Setting** | **Description** |
|---------|-------------|
| **Override Create SQL** | Toggle: True/False<br/>**True**: Customized Create SQL specified in the Create SQL space is executed. All other options are invisible<br/>**False**: Create view SQL based on options chosen are framed and executed |
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |

### Fact Joins

Join conditions and other clauses like where, qualify can be specified in the join space next to mapping of columns in the UI.

![fact_join](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/e540d2d0-2623-4b99-a435-a26df5fe3306)

> 📘 **Specify Group by and Order by Clauses**
>
> You should specify group by and order by clauses in this space if you are not opting for the group by all and order by provided in OPTIONS config.

### Fact Deployment

#### Fact Initial Deployment

When deployed for the first time into an environment the Fact node of materialization type table will execute the Create Fact Table stage.

| **Stage** | **Description** |
|-----------|----------------|
| **Create Fact Table** | This will execute a CREATE OR REPLACE statement and create a table in the target environment |
| **Create Fact View** | This will execute a CREATE OR REPLACE statement and create a view in the target environment |

#### Fact Redeployment

After the Fact node has been deployed for the first time into a target environment, subsequent deployments may result in either altering the Fact Table or recreating the Fact table.

#### Altering the Fact Tables

A few types of column or table changes will result in an ALTER statement to modify the Fact Table in the target environment, whether these changes are made individually or all together:

* Changing table names
* Dropping existing columns
* Altering column data types
* Adding new columns

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Clone Table** | Creates an internal table |
| **Rename Table\| Alter Column \| Delete Column \| Add Column \| Edit table description** | Alter table statement is executed to perform the alter operation |
| **Swap Cloned Table** | Upon successful completion of all updates, the clone replaces the main table ensuring that no data is lost |
| **Delete Table** | Drops the internal table |

#### Recreating the Fact Tables

If any changes like change in join clause, adding transformations, change in business key column, change in configs like adding distinct, group by or orderby, the Fact Table will be recreated by running a CREATE OR REPLACE statement.

If any of the following change are detected, then the table will be recreated using a CREATE or REPLACE.

* Join clause
* Adding transformation
* Change in business key
* Changes in configuration like adding distinct, group by, or order by

One of the following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Table** | Creates a new table |
| **Replace Table** | Replaces an existing table|

#### Recreating the Fact Views

The subsequent deployment of Work node of materialization type view with changes in view definition, adding table description or renaming view results in deleting the existing view and recreating the view.

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Removes existing view |
| **Create View** | Creates new view with updated definition |

### Fact Undeployment

If a Fact Node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the Fact Table in the target environment will be dropped.

This is executed in two stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete Table** | Coalesce Internal table is dropped |
| **Delete Table** | Target table in Databricks is dropped |

## View 

The Coalesce View UDN is a versatile node that allows you to develop and deploy a View in Databricks.

A view allows the result of a query to be accessed as if it were a table. Views serve a variety of purposes, including combining, segregating, and protecting data.

### View Node Configuration

The View node type has two configuration groups:

* [Node Properties](#view-node-properties)
* [Options](#view-options)

![Fact_config](https://github.com/coalesceio/Coalesce-Base-Node-Types/assets/7216836/6f863c3b-3abd-4318-bd7e-ed50a829f911)

#### View Node Properties

| **Properties** | **Description** |
|----------|-------------|
| **Storage Location** | Storage Location where the WORK will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/> If FALSE the node will not be deployed or will be dropped during redeployment |

#### View Options

| **Options** | **Description** |
|---------|-------------|
| **Override Create SQL** | Toggle: True/False<br/>**True**: Customized Create SQL specified in the Create SQL space is executed. All other options are invisible<br/>**False**: Create view SQL based on options chosen are framed and executed |
| **Multi Source** | Toggle: True/False<br/>Implementation of SQL UNIONs<br/>**True**: Combine multiple sources in a single node<br/>True Options:<br/>- **UNION**: Combines with duplicate elimination<br/>- **UNION ALL**: Combines without duplicate elimination<br/>**False**: Single source node or multiple sources combined using a join |
| **Distinct** | Toggle: True/False<br/>**True**: Group by All is invisible. DISTINCT data is chosen for processing<br/>**False**: Group by All is visible |
| **Group by All** | Toggle: True/False<br/>**True**: DISTINCT is invisible. Data is grouped by all columns for processing<br/>**False**: DISTINCT is visible |
| **Secure** | Toggle: True/False<br/>**True**: A secured view is created<br/>**False**: A normal view is created |

### View Joins

Join conditions and other clauses like where, qualify can be specified in the join space next to mapping of columns in the Coalesce app.

> 📘 **Specify Group by Clauses**
>
> Best Practice is to specify group by clauses in this space if you are not opting for the group by all provided in OPTIONS config.

### View Deployment

#### View Initial Deployment

When deployed for the first time into an environment the View node will execute the Create View stage.

| **Stage** | **Description** |
|-----------|----------------|
| **Create View** | This will execute a CREATE OR REPLACE statement and create a View in the target environment |

#### View Redeployment

The subsequent deployment of View node with changes in view definition, adding table description, adding secure option or renaming view results in deleting the existing view and recreating the view.

The following stages are executed:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Removes existing view |
| **Create View** | Creates new view with updated definition |

#### View Undeployment

If a View Node is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher level environment then the View in the target environment will be dropped.

This is executed in the below stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Delete View** | Removes the view from the environment |

## Code

### Work Code

* [Node definition](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Work-363/definition.yml)
* [Create Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Work-363/create.sql.js)
* [Run Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Work-363/run.sql.js)

### Persistent Stage Code

* [Node definition](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/PersistentStage-367/definition.yml)
* [Create Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/PersistentStage-367/create.sql.j2)
* [Run Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/PersistentStage-367/run.sql.j2)

  ### Dimenison Code

* [Node definition](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Dimension-365/definition.yml)
* [Create Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Dimension-365/create.sql.j2)
* [Run Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Dimension-365/run.sql.j2)
  
### Fact Code

* [Node definition](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Fact-366/definition.yml)
* [Create Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Fact-366/create.sql.j2)
* [Run Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/Fact-366/run.sql.j2)

### View Code

* [Node definition](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/View-452/definition.yml)
* [Create Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/View-452/create.sql.j2)
* [Run Template](https://github.com/coalesceio/databricks-nodes/blob/main/nodeTypes/View-452/run.sql.j2)

[Macros](https://github.com/coalesceio/databricks-nodes/tree/main/macros/macro-1.yml)
