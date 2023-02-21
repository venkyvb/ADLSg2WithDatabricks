### Overview
Both Azure Synapse and Databricks can be configured to work with an external Hive metastore. Currently Azure Synapse supports Azure SQL or Azure MySQL as the external hive metastore. 
**Note** - If you dont already use MySQL and are not able to set the `lower_case_table_names=2`, **please use Azure SQL**. If new Azure MySQL instances are provisioned, currently it is not possible to set this parameter (irrespective of the MySQL versions 5.7 or 8.0).

### Using Azure SQL as the external metastore
This [document](https://learn.microsoft.com/en-us/azure/databricks/data/metastores/external-hive-metastore) has a great summary of what needs to be done to use Azure SQL as the external hivemetastore.

#### Set-up scripts
Note that to deploy the Hive metastore schema you can either use the hive schema tool (mentioned in the above doc) or directly run the SQL scripts which are available for the appropriate hive versions - 
* [Hive versions < 3.0](https://github.com/apache/hive/tree/master/metastore/scripts/upgrade/mssql) - select the file with **hive-schema-<x.y.z>.mssql.sql**
* [Hive versions >= 3.0](https://github.com/apache/hive/tree/master/standalone-metastore/metastore-server/src/main/sql/mssql) - the file names are similar to above.

The testing has been done with ADB version `11.3 LTS (includes Apache Spark 3.3.0, Scala 2.12)` which uses the Hive metastore version of `2.3.9`.
Note that all of the set-up is to be done at a workspace level. This will be both a Synapse Workspace and a Databricks workspace.

#### Overall steps
##### Azure Synapse set-up
1. Set up the Azure SQL instance if one doesnt exist already.
2. Create a new database called `hivemetastore`.
3. Deploy the hive metastore schema (either via the hive schema tool or by running the scripts). The testing was done with running the scripts. Note that to do this, access to the Azure SQL is required which may involve setting up Private end-points per the security policies.
4. In Azure Synapse workspace create a Linked service for Azure SQL. 
5. As outlined in the [document](https://learn.microsoft.com/en-us/azure/databricks/data/metastores/external-hive-metastore), the configuration to connect with an external hive metastore can either be done via Spark config (for the pool itself) or per Spark session. The latter approach was tested, to do this just create a new Synapse notebook and enter the following (specifically tested with v2.3) -

```
%%configure -f
{
    "conf":{
        "spark.sql.hive.metastore.version":"<hive-version-first-2-parts-only>",
        "spark.hadoop.hive.synapse.externalmetastore.linkedservice.name":"<linked-service-name",
        "spark.sql.hive.metastore.jars":"/opt/hive-metastore/lib-<hive-version-first-2-parts-only>/*:/usr/hdp/current/hadoop-client/lib/*"
    }
}
```
For testing just enter the sample command -
```
spark.sql("show databases").show()
```
If the set-up is completed this should run successfully and list the set of databased under the hive metastore. (typically `default` would be displayed).

