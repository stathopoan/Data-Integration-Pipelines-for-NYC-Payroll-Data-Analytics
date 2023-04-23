# Data-Integration-Pipelines-for-NYC-Payroll-Data-Analytics
Develop a Data Analytics platform on Azure Synapse Analytics to answer specific questions.

## Project Overview

### Project Introduction

The City of New York would like to develop a Data Analytics platform on Azure Synapse Analytics to accomplish two primary objectives:

 1. Analyze how the City's financial resources are allocated and how much of the City's budget is being devoted to overtime.
 2. Make the data available to the interested public to show how the City’s budget is being spent on salary and overtime pay for all municipal employees.

You have been hired as a Data Engineer to create high-quality data pipelines that are dynamic, can be automated, and monitored for efficient operation. The project team also includes the city’s quality assurance experts who will test the pipelines to find any errors and improve overall data quality.

The source data resides in Azure Data Lake and needs to be processed in a NYC data warehouse in Azure Synapse Analytics. The source datasets consist of CSV files with Employee master data and monthly payroll data entered by various City agencies.

![img.png](/report_imgs/img.png)

### Project Environment

For this project, you'll do your work in the Azure Portal, using several Azure resources including:

 - Azure Data Lake Gen2 (Storage account with Hierarchical Namespaces checkbox checked when creating)
 - Azure SQL DB
 - Azure Data Factory
 - Azure Synapse Analytics

Instructions for using a temporary Azure account to complete the project are on the next page.

You'll take screenshots as proof of work for this project, so remember to collect these screenshots throughout the project steps. A checklist is provided at the end of each step so you can double-check you've collected all of the deliverables.

You'll also need to create a Github repository for this project. At the end of the project, you will connect your Azure pipelines to Github and submit the URL or contents of the repository.

When you submit your project, it will be assessed against this [project rubric](https://review.udacity.com/#!/rubrics/4857/view). 

### Project Data
 [Download](https://video.udacity-data.com/topher/2022/May/6283aff5_data-nyc-payroll/data-nyc-payroll.zip) these .csv files that provide the data for the project.
 
## Step 1: Prepare the Data Infrastructure

### 1. Create the data lake and upload data

Log into your temporary Azure account (instructions on the previous page) and create the following resources. Please use the provided resource group to create each resource. You will use these resources for the whole project, in all of the steps, so you'll only need to create one of each:

Create an Azure Data Lake Storage Gen2 (storage account) and associated storage container resource named adlsnycpayroll-yourfirstname-lastintial. Create three directories in this storage container named

 - dirpayrollfiles
 - dirhistoryfiles
 - dirstaging

Upload these files from the [project data](https://video.udacity-data.com/topher/2022/May/6283aff5_data-nyc-payroll/data-nyc-payroll.zip) to the dirpayrollfiles folder

 - EmpMaster.csv
 - AgencyMaster.csv
 - TitleMaster.csv
 - nycpayroll_2021.csv

Upload this file (historical data) from the project data to the dirhistoryfiles folder

 - nycpayroll_2020.csv

![img_1.png](/report_imgs/img_1.png)

![img_3.png](/report_imgs/img_3.png)

### 2. Create an Azure Data Factory Resource

Just a normal ADF resource would suffice

### 3. Create a SQL Database to store the current year of the payroll data

In the Azure portal, create a SQL Database resource named db_nycpayroll

 - Add client IP address to the SQL DB firewall

Tip: Make sure you tick "Allow Azure services and resources to access this server"

![img_2.png](/report_imgs/img_2.png)

Create a table called NYC_Payroll_Data in db_nycpayroll in the Azure Query Editor with this SQL Script:

`CREATE TABLE [dbo].[NYC_Payroll_Data]( [FiscalYear] [int] NULL, [PayrollNumber] [int] NULL, [AgencyID] [varchar](10) NULL, [AgencyName] [varchar](50) NULL, [EmployeeID] [varchar](10) NULL, [LastName] [varchar](20) NULL, [FirstName] [varchar](20) NULL, [AgencyStartDate] [date] NULL, [WorkLocationBorough] [varchar](50) NULL, [TitleCode] [varchar](10) NULL, [TitleDescription] [varchar](100) NULL, [LeaveStatusasofJune30] [varchar](50) NULL, [BaseSalary] [float] NULL, [PayBasis] [varchar](50) NULL, [RegularHours] [float] NULL, [RegularGrossPaid] [float] NULL, [OTHours] [float] NULL, [TotalOTPaid] [float] NULL, [TotalOtherPay] [float] NULL ) GO`

Go to Query Editor and run the above script. You should see the table created in the Tables section

![img_4.png](/report_imgs/img_4.png)

### 4. Create A Synapse Analytics workspace, or use one you already have created.

 - Create a new Azure Data Lake Gen2 and file system for Synapse Analytics when you are creating the Synapse Analytics workspace in the Azure portal.

 - Create a SQL dedicated pool in the Synapse Analytics workspace

 - Select DW100c as performance level. Keep defaults for other settings.
In the SQL dedicated pool, Create master data tables and payroll transaction tables using these SQL scripts (You can execute all four sql scripts at same time) :

Create Employee Master Data table:

`CREATE TABLE [dbo].[NYC_Payroll_EMP_MD](
    [EmployeeID] [varchar](10) NULL,
    [LastName] [varchar](20) NULL,
    [FirstName] [varchar](20) NULL
) 
GO`

Create Job Title Table:

`CREATE TABLE [dbo].[NYC_Payroll_TITLE_MD](
    [TitleCode] [varchar](10) NULL,
    [TitleDescription] [varchar](100) NULL
) 
GO`

Create Agency Master table:

`CREATE TABLE [dbo].[NYC_Payroll_AGENCY_MD](
    [AgencyID] [varchar](10) NULL,
    [AgencyName] [varchar](50) NULL
) 
GO`

Create Payroll transaction data table:

`CREATE TABLE [dbo].[NYC_Payroll_Data](
    [FiscalYear] [int] NULL,
    [PayrollNumber] [int] NULL,
    [AgencyID] [varchar](10) NULL,
    [AgencyName] [varchar](50) NULL,
    [EmployeeID] [varchar](10) NULL,
    [LastName] [varchar](20) NULL,
    [FirstName] [varchar](20) NULL,
    [AgencyStartDate] [date] NULL,
    [WorkLocationBorough] [varchar](50) NULL,
    [TitleCode] [varchar](10) NULL,
    [TitleDescription] [varchar](100) NULL,
    [LeaveStatusasofJune30] [varchar](50) NULL,
    [BaseSalary] [float] NULL,
    [PayBasis] [varchar](50) NULL,
    [RegularHours] [float] NULL,
    [RegularGrossPaid] [float] NULL,
    [OTHours] [float] NULL,
    [TotalOTPaid] [float] NULL,
    [TotalOtherPay] [float] NULL
) 
GO`

After creating the dedicated SQL pool go to Synapse workspace in Develop Tab and run the above scripts. 
TIP: Remember to switch to your dedicated pool and not default Built-in selection

![img_5.png](/report_imgs/img_5.png)

![img_6.png](/report_imgs/img_6.png)

After this step you will be able to see the 4 tables created under the dbo schema

## Step 2: Create Linked Services

### 1. Create a Linked Service for Azure Data Lake

In Azure Data Factory, create a linked service to the data lake that contains the data files

 - From the data stores, select Azure Data Lake Gen 2
 - Test the connection

### 2. Create a Linked Service to SQL Database that has the current (2021) data

If you get a connection error, remember to add the IP address to the firewall settings in SQL DB in the Azure Portal

### 3. Create a Linked Service for Synapse Analytics

 - Create the linked service to the SQL pool.

![img_7.png](/report_imgs/img_7.png)

## Step 3: Create Datasets in Azure Data Factory

Datasets simply point to the source/target data that we want to use in Dataflows. Datasets cannot be created without Linked Services

### 1. Create the datasets for the 2021 Payroll file on Azure Data Lake Gen2

 - Select DelimitedText
 - Set the path to the nycpayroll_2021.csv in the Data Lake
 - Preview the data to make sure it is correctly parsed

### 2. Repeat the same process to create datasets for the rest of the data files in the Data Lake

 - EmpMaster.csv
 - TitleMaster.csv
 - AgencyMaster.csv
 - Remember to publish all the datasets

### 3. Create the dataset for transaction data table that should contain current (2021) data in SQL DB

### 4. Create the datasets for destination (target) tables in Synapse Analytics

 - dataset for NYC_Payroll_EMP_MD
 - for NYC_Payroll_TITLE_MD
 - for NYC_Payroll_AGENCY_MD
 - for NYC_Payroll_Data

![img_8.png](/report_imgs/img_8.png)

## Step 4: Create Data Flows

### 1.In Azure Data Factory, create the data flow to load 2021 Payroll Data to SQL DB transaction table (in the future NYC will load all the transaction data into this table).

 - Create a new data flow
 - Select the dataset for the 2021 payroll file as the source
 - Select the sink dataset as the payroll table on SQL DB
 - Make sure to reassign any missing source to target mappings

Tip: To enable Data preview  you need to enable Data flow debug. This will spin off a spark cluster for you for as long as you define it

![img_9.png](/report_imgs/img_9.png) ![img_10.png](/report_imgs/img_10.png) ![img_11.png](/report_imgs/img_11.png)

NOTE: In the projection tab in the source of the dataflow make sure to select correct projections for the sink. Example: 
Fiscal year is in string format in the source but int format in the sink in SQL DB.

![img_12.png](/report_imgs/img_12.png)

Special attention to Date format

![img_14.png](/report_imgs/img_14.png)

NOTE: Make sure to disable automapping in the sink and make sure the source and sink columns are in in sync

![img_13.png](/report_imgs/img_13.png)


### 2. Create Pipeline to load 2021 Payroll data into transaction table in the SQL DB

 - Create a new pipeline
 - Select the data flow to load the 2021 file into SQLDB
 - Trigger the pipeline
 - Monitor the pipeline
 - Take a screenshot of the Azure Data Factory screen pipeline run after it has finished.
 - Make sure the data is successfully loaded into the SQL DB table

![img_18.png](/report_imgs/img_18.png)

![img_15.png](/report_imgs/img_15.png)

![img_16.png](/report_imgs/img_16.png)

![img_17.png](/report_imgs/img_17.png)

### 3. Create data flows to load the data from the data lake files into the Synapse Analytics data tables

 - Create the data flows for loading Employee, Title, and Agency files into corresponding SQL pool tables on Synapse Analytics
 - For each Employee, Title, and Agency file data flow, sink the data into each target Synaspe table

![img_26.png](/report_imgs/img_26.png)



### 4. Create a data flow to load 2021 data from SQL DB to Synapse Analytics

![img_20.png](/report_imgs/img_20.png)
![img_27.png](/report_imgs/img_27.png)
![img_28.png](/report_imgs/img_28.png)


### 5. Create pipelines for Employee, Title, Agency, and year 2021 Payroll transaction data to Synapse Analytics containing the data flows.

 - Select the dirstaging folder in the data lake storage for staging
 - Optionally you can also create one master pipeline to invoke all the Data Flows
 - Validate and publish the pipelines


![img_29.png](/report_imgs/img_29.png)

### 6. Trigger and monitor the Pipelines

 - Take a screenshot of each pipeline run after it has finished, or one after your master pipeline run has finished.

![img_21.png](/report_imgs/img_21.png)
![img_22.png](/report_imgs/img_22.png)
![img_23.png](/report_imgs/img_23.png)
![img_24.png](/report_imgs/img_24.png)
![img_25.png](/report_imgs/img_25.png)


## Step 5: Data Aggregation and Parameterization

In this step, you'll extract the 2021 year data and historical data, merge, aggregate and store it in Synapse Analytics. The aggregation will be on Agency Name, Fiscal Year and TotalPaid.

### 1. Create a Summary table in Synapse with the following SQL script and create a dataset named table_synapse_nycpayroll_summary

`CREATE TABLE [dbo].[NYC_Payroll_Summary]( [FiscalYear] [int] NULL, [AgencyName] [varchar](50) NULL, [TotalPaid] [float] NULL )`

### 2. Create a new dataset for the Azure Data Lake Gen2 folder that contains the historical files.

Select dirhistoryfiles in the data lake as the source

### 3.Create new data flow and name it Dataflow Aggregate Data

 - Create a data flow level parameter for Fiscal Year
 - Add first Source for table_sqldb_nyc_payroll_data table
 - Add second Source for the Azure Data Lake history folder

### 4.Create a new Union activity in the data flow and Union with history files

### 5.Add a Filter activity after Union

 - In Expression Builder, enter `toInteger(FiscalYear) >= $dataflow_param_fiscalyear`

### 6.Derive a new TotalPaid column

 - In Expression Builder, enter `RegularGrossPaid + TotalOTPaid+TotalOtherPay`

![img_33.png](/report_imgs/img_33.png)

### 7.Add an Aggregate activity to the data flow next to the TotalPaid activity

 - Under Group By, Select AgencyName and Fiscal Year

### 8.Add a Sink activity to the Data Flow

 - Select the dataset to target (sink) the data into the Synapse Analytics Payroll Summary table.
 - In Settings, select Truncate Table

![img_30.png](/report_imgs/img_30.png)

![img_31.png](/report_imgs/img_31.png)

### 9.Create a new Pipeline and add the Aggregate data flow

 - Create a new Global Parameter (This will be the Parameter at the global pipeline level that will be passed on to the data flow
 - In Parameters, select Pipeline Expression
 - Choose the parameter created at the Pipeline level

![img_32.png](/report_imgs/img_32.png)

### 10.Validate, Publish and Trigger the pipeline. Enter the desired value for the parameter.

### 11.Monitor the Pipeline run and take a screenshot of the finished pipeline run.

![img.png](/report_imgs/img_35.png)

![img_34.png](/report_imgs/img_34.png)