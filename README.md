# **Delta Live Tables**

## **Introduction**

The Coalesce DLT node type allows you to create a streaming table,a type
of Delta Live Table.

[[Delta Live
Tables]{.underline}](https://docs.databricks.com/en/delta-live-tables/index.html)
is a declarative framework for building reliable, maintainable, and
testable data processing pipelines. Delta Live Tables manages how your
data is transformed based on queries you define for each processing
step. You can also enforce data quality with Delta Live Tables
expectations, which allow you to define expected data quality and
specify how to handle records that fail those expectations.

Databricks recommends using streaming tables to ingest data using
Databricks SQL. A [*[streaming
table]{.underline}*](https://docs.databricks.com/en/tables/streaming.html#additional-resources)
is a table registered to Unity Catalog with extra support for streaming
or incremental data processing. A Delta Live Tables pipeline is
automatically created for each streaming table.

## **Pre-requisites:**

1.Loading file from external location is supported.Loading from
databricks managed volume or external volume is currently not supported

2.The node which loads from a file creates a streaming table with suffix
\_raw.For further processing on the table created,we have add the table
to the Browser using 'Add source' tab

3.Materialized View is currently not supported by DLT node type.

### **DLT Node Configuration**

The DLT has two configuration groups:

-   [[Node
    > Properties]{.underline}](https://github.com/Anandhi-Sekaran-DE/PRJAS100073/edit/databricks-04/README.md#DLT-work-node-properties)

-   [[DLT
    > Options]{.underline}](https://github.com/Anandhi-Sekaran-DE/PRJAS100073/edit/databricks-04/README.md#DLT-options)

**DLT Node Properties**

There are four configs within the Node Properties group.

-   Storage Location(required): Storage Location where the Dynamic Table
    > will be created.

-   Node Type(required): Name of template used to create node objects.

-   Description: A description of the node\'s purpose.

-   Deploy Enabled(required):

    -   If TRUE the node will be deployed or redeployed when changes are
        > detected.

    -   If FALSE the node will not be deployed or the node will be
        > dropped during redeployment.

**DLT Options**

![](vertopal_e2891625931a467189b3388ccf4f195b/media/image1.png){width="3.3766415135608048in"
height="4.640625546806649in"}

-   Type of Delta Live Table: The type of Delta Live Table

\* Streaming table

\* Materialized view

-   Read Files: True or False Toggle to whether the source id file or
    > table

    -   True - Allows you to load the file from external cloud location
        > into streaming table

    -   False - Allows you to load from table source

```{=html}
<!-- -->
```
-   File location(Read Files-true):The external cloud location of file

-   File format(Read Files-true):The file format of the file to be
    > loaded

> \* csv
>
> \* json

-   Delimiter(Read Files-true):The delimiter used in case of csv file

-   Header(Read Files-true):In case of csv file,the header is needed for
    > file parsing

-   Table Properties:Table properties like quality can be mentioned here

-   Partition by(Read Files-false):Columns based on which streaming
    > table is to be partitioned

-   Table constraints(Read Files-false):Primary key and Foreign key
    > constraints can be specified

\*Primary key

\*Foreign key

-   Other Constraints(Read Files-false):Any other constraints specific
    > to columns can be specified here.

> \*Column name
>
> \*Expectation expression
>
> \*On Violation action

## **DLT Deployment**

### **DLT Initial Deployment**

When deployed for the first time into an environment the DLT node will
execute the following stage:

-   Create Streaming Table: This stage will execute a CREATE OR REFRESH
    > statement and creates a Streaming Table in the target environment.

### **DLT Redeployment**

If the initial deployment failed,then the redeployment after correcting
any errors successfully deploys the node.

Other scenarios like change in data type,drop or add columns are not
supported

### **DLT Undeployment**

If a DLT Node is deleted from a Workspace, that Workspace is committed
to Git and that commit deployed to a higher level environment then the
Table in the target environment will be dropped.

This is executed in two stages:

-   Delete Table: Coalesce Internal table is dropped.

-   Delete Table: Target table in Snowflake is dropped.
