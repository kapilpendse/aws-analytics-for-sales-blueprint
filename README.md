# AWS Analytics for Sales
This is a blueprint for setting up an analytics platform for sales related data such as point of sale, loyalty, marketing etc. It connects the right AWS building blocks to get you from data to insights in minutes.

## User Guide
AWS is a very broad collection of services, and it can be a daunting task to just identify and configure the right set of services when you are just getting started. This guide keeps it simple and in a few steps takes you from zero knowledge of AWS to a point where you will be able to visualize and analyze your data with a few clicks. You can bring your own data, but if you don't have it handy, this guide also contains instructions on how to use the fake sample data that is provided.

By following this guide, you will be using the blueprint to automatically configure the following 5 AWS services in your AWS account:
- Amazon S3 for data storage
- Amazon Glue to generate a logical schema for your raw data
- Amazon Athena to query the data
- Amazon QuickSight to visualize the data
- Optionally, Amazon Kinesis Data Firehose for ingesting data as a stream

**Q. Wow, that's a lot of unfamiliar names. Doesn't it already feel daunting? How come this is "keeping it simple" you ask?**
A. Well, you will only interact with 2 of these i.e. QuickSight to create dashboards and Athena to write custom SQL queries.


## Architecture Diagram
Here's a high level diagram of what the blueprint will automatically configure for you:
![architecture diagram](/arch/diagram.png)


## Deploy the blueprint
1. The blueprint is coded as an AWS CloudFormation template. Deploy the blueprint using the button below. If you are curious, you can [open the blueprint here](/cfn/blueprint-serverless.yaml).

Region| Launch
------|-----
Asia Pacific (Singapore) | [![cfn](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=ap-southeast-1#/stacks/new?stackName=sales-analytics&templateURL=https://kapilpen.s3-ap-southeast-1.amazonaws.com/labs/aws-analytics-for-sales-blueprint/blueprint-serverless.yaml)

2. Scroll to the bottom and click **Next**.
	* *Stack name*: **sales-analytics**
	* *databaseName*: **sales-data**
	* *rawDataBatchSizeInMBs*: **1**
	* *rawDataBatchSizeInSeconds*: **60**
	* *s3BucketName*: **sales-data-YOUR NAME OR INITIALS**
	* Scroll to the bottom and click **Next**, then **Next**
	* **Acknowledge the checkbox** and click **Create stack**

This deploys everything in about 5 to 10 minutes. The rest is just testing and getting familiar. Wait until the *status* of the stack becomes **CREATE_COMPLETE** and then proceed to the next section.


## Upload Data
In order to test this analytics platform, we need to have some data. You can bring your own data such as point-of-sale data, loyalty data, marketing campaign related data etc. This blueprint uses AWS Glue to infer the logical structure of your data. AWS Glue comes with built-in support for a number of [file formats](https://docs.aws.amazon.com/glue/latest/dg/add-classifier.html#classifier-built-in).

For starters however, just to get familiar, you can use the fake sample data that is [provided here](/data). The instructions below assume that you are using the provided fake sample data.

2. Download the files 'pos-data.csv' and 'loyalty.csv'.
3. Go to [Amazon S3 console](https://s3.console.aws.amazon.com/s3/buckets/) and open the bucket **sales-data-YOUR NAME OR INITIALS**. This is the same name that you specified during the launch of the blueprint.
4. The bucket will be empty. Create a folder called 'raw'.
5. Inside the 'raw' folder, create another folder 'imported'.
6. Inside the 'imported' folder, create 2 folders - 'pos' and 'loyalty'.
7. Upload the 'pos-data.csv' file to the 'pos' folder and the 'loyalty.csv' file to the 'loyalty' folder.
8. Wait for about 5 minutes.


## Verify Glue Crawler Result
The blueprint has scheduled a AWS Glue crawler to scan these folders every 5 minutes. The crawler attempts to make sense of the raw data and creates a meta data table in the Glue data catalog.

9. After about 5 minutes of uploading the fake sample data, open the [crawlers page in AWS Glue console](https://ap-southeast-1.console.aws.amazon.com/glue/home?region=ap-southeast-1#catalog:tab=crawlers).
10. Find the crawler named 'sales-data-crawler', and observe the schedule column. It should say 'Every 5 minutes'.
11. In the left hand menu, click [databases](https://ap-southeast-1.console.aws.amazon.com/glue/home?region=ap-southeast-1#catalog:tab=databases), then click 'sales-data', and then click the link 'Tables in sales-data'. You should see 2 tables - 'loyalty' and 'pos'. Click on any one.
12. Scroll down to the 'Schema' section. An automatically infered schema will be displayed. Verify that the column names and data types have been correctly infered by Glue crawler.
	- AWS Glue crawler automatically infers the data schema for [most common data formats like JSON, CSV, TSV etc.](https://docs.aws.amazon.com/glue/latest/dg/add-classifier.html#classifier-built-in)
	- If your data is not correctly infered by AWS Glue crawler, you can create a [Custom Classifier](https://docs.aws.amazon.com/glue/latest/dg/custom-classifier.html).


## Interactively Query Your Data
Now that your data is in Amazon S3 and Glue has infered its schema successfully, let's run some SQL queries to explore the data.

13. Go to [Athena in your AWS console](https://ap-southeast-1.console.aws.amazon.com/athena/home?force&region=ap-southeast-1#query).
14. In the left hand menu, under **Database**, click the dropdown and select *sales-data*
15. Under **Tables**, click on *loyalty* or *pos*. The table should expand to reveal all the column names.
16. On the right hand side, click **+** to create a new query. Click into the query editor and type the following:
	```
	select * from loyalty limit 5;
	```
17. Click **Run query**. Within a few seconds, you should see the query result at the bottom.
18. In the query editor, click **+** to create a new query. Click into the query editor and type the following:
	```
	select * from pos limit 5;
	```
19. Click **Run query**. Within a few seconds, you should see the result which 5 point-of-sale transactions from the *pos* table.

This was just a very simple example of what you can do with Athena. You can write much more complex queries. If you want to learn about that, [continue to the Athena User Guide](https://docs.aws.amazon.com/athena/latest/ug/querying-athena-tables.html).

## Visualize Your Data
Now that we have explored the data little bit, let's learn how to visualize it using Amazon QuickSight. Amazon QuickSight is a business intelligence service in AWS.

20. Go to [QuickSight in your AWS console](https://ap-southeast-1.quicksight.aws.amazon.com/sn/start). Sign up for the service if you haven't done so already.
21. After you are inside the QuickSight console, in the top-right corner switch to the region in which you have deployed this blueprint (by default it should be Singapore).
22. Connect QuickSight to your dataset using Amazon Athena by following [these instructions](https://docs.aws.amazon.com/quicksight/latest/user/create-a-data-set-athena.html)
	- name your data source 'sales-data'
	- select 'sales-data' from the database dropdown
	- select 'pos' from the list of tables
	- select 'Directly query your data'. Click here to learn more about [QuickSight SPICE](https://docs.aws.amazon.com/quicksight/latest/user/welcome.html#spice).
23. So far we've only added the 'pos' table to QuickSight. The 'loyalty' table in the fake sample data is actually related to the 'pos' data, so let's make that connection in QuickSight. Click [here](https://ap-southeast-1.quicksight.aws.amazon.com/sn/data-sets).
24. Click the 'pos' data set, then click 'Edit data set'.
25. At the top-center of the page, just below the dark blue bar you will see a small link 'Add data'. Click it. If you can't find it, refer to the [screenshot here](https://docs.aws.amazon.com/quicksight/latest/user/joining-tables.html).
26. Select the 'sales-data' and the 'loyalty' table.
27. On the next screen, click on the 2 pink dots. The 'Join Configuration' section should open in the bottom half of the page.
28. Select 'cust_id' column for both 'pos' and 'loyalty' tables and leave the Join type as 'Inner'. This creates a join between the 2 tables on the customer ID column. Click *Apply*. Then click *Save & visualize* at the top-center of the page.
29. Now you will be able to create visualizations and publish them as dashboards. Feel free to explore the QuickSight tool. For starters, try to create a visualization like below:
![viz-1](/assets/viz-1.png)

30. You can add parameterized date filter to your visualization in order to restrict your visualization to data from a specific date range. For this, you first need to [set up start date and end date parameters](https://docs.aws.amazon.com/quicksight/latest/user/parameterize-a-filter.html) and then [add a date filter](https://docs.aws.amazon.com/quicksight/latest/user/add-a-date-filter.html) of the type 'time range''between'.


## Stream Your Data
You can stream your sales data such as point-of-sale transactions straight to your Amazon Kinesis Data Firehose delivery stream. There are multiple ways to do this, but here's a couple that are popular:

- [Fluent Plugin for Amazon Kinesis](https://github.com/awslabs/aws-fluent-plugin-kinesis)
- [Kinesis Agent](https://docs.aws.amazon.com/firehose/latest/dev/writing-with-agents.html)

Once your streaming data starts coming into this platform, within a few minutes you will be able to start querying and visualizing it.

You can start querying and visualizing your data only after the Glue crawler creates a table for it in the Glue data catalog. Remember that we've set up the Glue crawler to run once every 5 minutes. If you don't want to wait for its next run, you can run it manually via the AWS Glue console. 

## Additional Configurations

### Pre-ingestion
There are many ways to adapt the above pipeline to your particular requirements. For example, if you want to implement filtering or other data transformations before the data is stored to S3, you can do so using the [Kinesis Firehose Data Transformation feature](https://docs.aws.amazon.com/firehose/latest/dev/data-transformation.html). You can have your data automatically converted from JSON to Apache Parquet or Apache ORC formats that are better suited for querying. To do so, configure the [Record Format Conversion](https://docs.aws.amazon.com/firehose/latest/dev/record-format-conversion.html) feature of Kinesis Data Firehose.

### Post-ingestion
If you want to implement ETL jobs on the data that is stored in your S3 bucket, an easy way to do this is using [AWS Glue Jobs](https://docs.aws.amazon.com/glue/latest/dg/add-job.html). There are two types of jobs in AWS Glue: Spark and Python shell. You can get started quickly by using [built-in transforms](https://docs.aws.amazon.com/glue/latest/dg/built-in-transforms.html) to process your data. When you [automatically generate](https://docs.aws.amazon.com/glue/latest/dg/console-edit-script.html) the source code logic for your job, a script is created. You can edit this script, or you can provide your own script to process your ETL work. IMPORTANT: To develop and test your AWS Glue scripts, remember to use [Development Endpoints](https://docs.aws.amazon.com/glue/latest/dg/console-development-endpoint.html) for better dev/test iterative workflow.

### Access to managed services using private endpoints
This blueprint uses fully managed services from AWS, such as AWS Glue, Amazon Kinesis Data Firehose, Amazon S3 and Amazon QuickSight. As fully managed services, by default they run outside of your VPCs and are accessed via public API endpoints. You can access some of these services via private [VPC Endpoints](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints.html). Depending on your security requirements, you may want to implement VPC Endpoints to these services in your AWS accounts. Learn more here:
    * [Endpoints for Amazon S3](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-endpoints-s3.html)
    * [Using AWS Glue with VPC Endpoints](https://docs.aws.amazon.com/glue/latest/dg/vpc-endpoint.html)
    * [Using an Amazon VPC with Amazon QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/working-with-aws-vpc.html)
    * [Using Amazon Kinesis Data Firehose with AWS PrivateLink](https://docs.aws.amazon.com/firehose/latest/dev/vpc.html)

### Resource Tagging
Objects stored in Amazon S3 can be tagged. Tags are simple key-value pairs that help with organizing and classifying your data. To learn more, [click here](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-tagging.html). AWS Glue also supports tagging of resources such as Crawlers and Jobs. To help you manage your AWS Glue resources, you can optionally assign your own tags to some AWS Glue resource types. Learn more [here](https://docs.aws.amazon.com/glue/latest/dg/monitor-tags.html).


