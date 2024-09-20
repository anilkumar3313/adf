# KraftHeinz Data Engineering - Azure Data Factory

Microsoft Azure's Azure Data Factory is our orchestration tool of choice. It will be the resource that schedules/triggers each of our batch pipelines loading into Snowflake. 

## Folder Structure
ADF has a strict folder structure it must follow that we will mirror

| **Folder Path** | **Folder Name** | **Usage** |
| --- | --- | --- |
| azure_data_factory | azure_data_factory | Base folder which is attached to ADF |
| azure_data_factory/dataflow | dataflow | Has Json Files which are Dataflow objects| 
| azure_data_factory/dataset | dataset | Has Json Files which are dataset objects| 
| azure_data_factory/factory | factory | Has Json Files which are factory objects| 
| azure_data_factory/integrationRuntime | integrationRuntime | Has Json Files which are integrationRuntime objects| 
| azure_data_factory/linkedService | linkedService | Has Json Files which are linkedService objects| 
| azure_data_factory/pipeline | pipeline | Has Json Files which are pipeline objects| 
| azure_data_factory/trigger | trigger | Has Json Files which are trigger objects|
| config/azure_data_factory_config | azure_data_factory_config | Had environment specific values to be replaced in the objects |
| python_code/adf_deployment | adf_deployment | Has the python Code whcih will help us with the JSON Object Deployment | 

## Naming Conventions
* Pipelines: All ADF Pipelines must start with PL_KHCDE_\[Application Name\]_Functionality
* Datasets: All ADF Datasets must start with DS_KHCDE_\[DATA SOURCE TYPE\]_NAME
* Triggers: All ADF Datasets must be start with TRG_KHCDE_\[Application Name]\_\[FREQUENCY / EVENT\]
* LinkedServices: All ADF Datasets must start with LS_\[Linke Service Type\]_NAME
* Pipelines, Data sets & Data flows have to be under the Application folder

## Build and Release Process
The DataFactory build and release process will be driven by the ADF UI. 
After making changes to your feature branch, the next step would be to create a pull request into our development branch. 
Once that pull request is approved and completed, the ADF "Publish" functionality in the UI will need to be triggered in order to update the adf_publish branch. 
Once the adf_publish branch is updated, our build process will deploy all objects within data factory into our Development ADF. This process will be replicated for UAT and Production.

## ADF Deployment Approaches:
We have 2 methods to deploy the ADF objects to higher environments
1. ARM Deployment approach
2. JSON Objects Deployment

| **Comparision** | **ARM Deployment approach** | **JSON Object Deployment** |
| --- | --- | --- |
| Deployment Approach | This is Azure Defined approach, where the ARM template is the source for deployment <br> Steps: <br> 1. ARM Template is picked from adf_publish branch <br> 2. Stop Triggers <br> 3. Deploy ADF Changes using ARM Temaplate <br> 4. Start the Triggers | JSON files from master branch are used <br> Steps: <br> 1. Objects to be deployed are determined <br> 2. Objects which had changes are only deployed <br> 3. Triggers which are beeing changed are only stopped <br> 5. Trigger are started if they were in started state when deployed else they will be left stopped |
| Advantages | Azure Prescirbed process | Selective deployment of Objects |
| Challenges | 1. Every time all Triggers are stopped & Started <br> 2. Every Object is deployed <br> 3. Time consuming process | 1. Custom coding involved <br> 2. Open Source DEVOPS Task Used |
| Limitations | 1. Triggers state is not preserved <br> 2. Tumbling window trigger deployment is complex | 1. Tumbling window trigger deployment is complex |

**Note**: We are using the JSON objects deployment approach, which will help us with over all deployment time and provides flexibility on deployment windows.

### JSON Object Deployment Flow:
------------------------------------------

Devops Task - https://marketplace.visualstudio.com/items?itemName=SQLPlayer.DataFactoryTools

Below is the flow of JSON object deployment approach:

![ADF Deployment](https://heinz365.sharepoint.com/:i:/r/sites/Analytics/Shared%20Documents/COE%20Docs/Data%20Lake/Cloud/Cloud%20Analytics%20Platfrom%20Project/Cloud%20Analytics%20Platform%20Project%20Docs/Data%20Engineering%20Development/Generic%20Frameworks/documentation_images/adf_json_deployment_flow.png)

### Steps for Developer in ADF Deployment:
------------------------------------------

![Developer Role](https://heinz365.sharepoint.com/:i:/r/sites/Analytics/Shared%20Documents/COE%20Docs/Data%20Lake/Cloud/Cloud%20Analytics%20Platfrom%20Project/Cloud%20Analytics%20Platform%20Project%20Docs/Data%20Engineering%20Development/Generic%20Frameworks/documentation_images/adf_deployment_developer_role.png)

## Self Hosted Integration Runtime Servers:

The Integration Runtime (IR) is the compute infrastructure used by Azure Data Factory to provide the data integration capabilities across different network environments.<br>

Below are the list of SHIR's created for DE Data load process:
|**IR Name**|**Server Name**|**Connectivity Mode (to IR Server)**|**Environment**|**Hosted**|**ADF's in IR can be used**|**IR is Created in ADF**|**To be Used for**|
|---|---|---|---|---|---|---|---|
|KHAZ-EUS2-SHIR|KHAZE2DAPPIR01.mykft.net (10.13.20.4) |VPN|DEV & UAT|Azure - EUS2|KHAZ-ADF-DEV <br>KHAZ-ADF-UAT <br>KHAZ-ADF-INT <br>|KHCDE-DEV-ADF-IR|When the Resources / Source System are in AZ - VPC|
|KHAZ-EUS2-SHIR|(Clustered Via ADF IR Process)<br>KHAZE2PAPPIR01.mykft.net (10.13.16.4)<br>EUS2PAPPIR001.mykft.net (10.13.16.10)<br>EUS2PAPPIR002.mykft.net (10.13.16.11)|VPN|PROD|Azure - EUS2|KHAZ-ADF-PROD|KHCDE-ADF-IR|When the Resources / Source System are in AZ - VPC|
|KHC-TULSA-ONPREM-SHIR|KHUSOKTULMSO023.mykft.net (146.238.52.67) |VPN|DEV & UAT|On-Prem - Tulsa|KHAZ-ADF-DEV <br>KHAZ-ADF-UAT <br>KHAZ-ADF-INT <br>KHCDE-ADF-SECURE-NONPROD <br>|KHAZE2-DataFactory-D01|When the Resources / Source System are KHC Network|
|KHC-TULSA-ONPREM-SHIR|(Clustered Via ADF IR Process)<br>KHUSOKTULMSO058.mykft.net (146.238.53.0) <br>KHUSOKTULMSO031.mykft.net (146.238.52.152) <br>KHUSOKTULMSO060.mykft.net(146.238.53.2) <br>|VPN|PROD|On-Prem - Tulsa|KHAZ-ADF-PROD <br>KHCDE-ADF-SECURE-PROD <br>|KHCDE-ADF-IR|When the Resources / Source System are KHC Network|

**Note:** If there is need for a specific IP to be provided to source team to whitelist provide the IP(s) **(KHAZEUS2CGFPUB001-nic0-public)**
1.	52.252.18.209
2.	20.62.30.121 
3.	52.252.18.208 
4.	52.167.248.60

##### For more details on Azure ADF IR: https://docs.microsoft.com/en-us/azure/data-factory/concepts-integration-runtime

## Generic ADF Pipelines Details:
* In ADF KHC COE has built a lot of Generic Pipelines which does the activity or can be used as a sub-pipeline in a end to end flow of a application data load.<br>
* Below are the list of Generic pipelines:<br>

|**Pipeline Name**|**Usage**|**Parameters**|**How is Triggered?**|
|---|------|-----------|---|
|PL_KHCDE_GENERIC_MASTER|Copy the files from Source systems to AZ Storage and creats a trigger file|1. application_name<br> 2. dataset_name<br> 3. parent_id<br> 4. delete_file_from_source(y/n) <br> 5. delete_file_from_stage(y/n) <br>|Scheduled - Every 1 Hour|
|PL_KHCDE_GENERIC_BLOB_TO_SFTP_PUSH|Push the files from Storage to SFTP Location|1. app_name<br> 2. dataset_group|To be used as a sub pipeline in the Application specific pipeline|
|PL_KHCDE_GENERIC_DBTPROCESSING|Execute the DBT - Tag / Mode / Multiple Commands|1. ApplicationName<br> 2. DatasetGroup<br> 3. ParameterJson|To be used as a sub pipeline in the Application specific pipeline|
|PL_KHCDE_GENERIC_SNOWFLAKEINGESTION|Ingestion in Snowflake Raw Tables|1. ApplicationName<br> 2.DatasetGroup<br> 3. ParameterJson|To be used as a sub pipeline in the Application specific pipeline|
|PL_KHCDE_GENERIC_SNOWFLAKEEXTRACTION|Extraction of Snowflake final tables to ADLS Storgae|1. ApplicationName<br> 2. DatasetGroup<br> 3. ParameterJson|To be used as a sub pipeline in the Application specific pipeline|
|PL_KHCDE_GENERIC_PORTALDOWNLOAD|Download the files from Portals - Webscrapping |1. ApplicationName<br> 2. DatasetGroup<br> 3. ParameterJson|To be used as a sub pipeline in the Application specific pipeline |
|PL_KHCDE_GENERIC_PYTHON|Execute the Python code for Applications which need Python Execution|1. ApplicationName<br> 2. DatasetGroup<br> 3. ParameterJson<br> 4. pythonArgs|To be used as a sub pipeline in the Application specific pipeline|
|PL_KHCDE_GENERIC_SAP_CATLYST_HANA_DATA_EXTRACT|Extract data from SAP CATALYST HANA DB|1. application_name<br> 2. sap_schema<br> 3. dataset_group<br> 4. extraction_query<br> 5. parent_id|To be used as a sub pipeline in the Application specific pipeline |
|PL_KHCDE_GENERIC_SAP_KEYSTONE_DATA_EXTRACT|Extract data from SAP Keystone DB|1. application_name<br> 2. sap_schema<br> 3. dataset_group<br> 4. extraction_query<br> 5. parent_id|To be used as a sub pipeline in the Application specific pipeline |
|PL_KHCDE_GENERIC_FS_SFDC_DATA_EXTRACT|Extract dara from Salesforce - Foodservices |1. application_name<br> 2. dataset_name|Scheduled daily @10:30 UTC|
|PL_KHCDE_GENERIC_SFDC_DATA_EXTRACT_EDIFY|Extract dara from Salesforce - Edify |1. application_name<br> 2. dataset_name|Scheduled - Every 8 hours|