<p align="center">
    <a alt="License"
        href="https://github.com/coalesceio/sap/blob/SAP_Dynamic_Tables/LICENSE">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" /></a>
    <a alt="Maintained?">
        <img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" /></a>
    <a alt="PRs">
        <img src="https://img.shields.io/badge/Contributions-welcome-blueviolet" /></a>
    <a alt="Fivetran Quickstart Compatible"
</p>

# SAP Transformation Coalesce Pipeline ([Docs](https://github.com/coalesceio/sap/))
# What does this Coalesce pipeline do?
- Produces modeled tables that leverage SAP data from [Fivetran's connector](https://fivetran.com/docs/applications) in the format described by [this ERD](https://fivetran.com/docs/applications).
- Enables you to better understand the workload, performance, and velocity of your team's work using SAP issues. It performs the following actions:
  - Creates a daily issue history table so you can quickly create agile reports, such as burndown charts, along any issue field.
  - Enriches the core issue table with relevant data regarding its workflow and current state.
  - Aggregates bandwidth and issue velocity metrics along projects and users.
- Generates a comprehensive data dictionary of your source and modeled SAP data available in your Coalesce Organization Generated Documentation tab. [Environment or Generated Documentation](https://docs.coalesce.io/docs/generated-documentation).

<!--section="SAP_transformation_model"-->
The following table provides a detailed list of all models materialized within this package by default. 

| **model**                         | **description**                                                                                                                                                                                                                             |
|--------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| sap__0comp_code_attr   |  This model is used for loading company code attributes, extracting from the t001 data source.                              |
| sap__0comp_code_text   |  This model is used for loading company code text information, extracting from the t001 data source.   |
| sap__0company_text  |  This model is used for loading customer text data, extracting from the t880 data source.   |
| sap__0customer_attr |  This model is used for loading customer master data, originating from the kna1 source.                               |
| sap__0employee_attr | This model contains information that concerns the employee's work relationship, extracting master data from the personnel administration tables. |
| sap__0fi_gl_10   | This model extracts the transaction figures from the leading ledger in the new General Ledger.  |
| sap__0fi_gl_14  | This model extracts line items from the leading ledger in new General Ledger Accounting.                                                                |                            
| sap__0gl_account_attr  |   This model is used for loading G/L Account Number master data, originating from the ska1 source.                 |
| sap__0material_attr  |  This model is used to display material attribute information, originating from the mara source.                                                           |
| sap__0vendor_attr |  This model is used to display vendor attributes, originating from the lfa1 source.                           |
| sap__0vendor_text  |  This model is used to display vendor text, originating from the lfa1 source.     |

<!--section-end-->

# How do I use the Coalesce pipeline?

## Step 1: Prerequisites
To use this Coalesce pipeline, you must have the following:

- A Fivetran SAP connector syncing data into your **Snowflake** destination.
- Schemas that map to Coalesce Storage Locations
- Coalesce Markeplace Packages (already included in Git repo)
  - [Base Node Types](https://docs.coalesce.io/page/package_coalesce_base-node-types) (version 1.1.2+)
  - [Incremental Loading](https://docs.coalesce.io/page/package_coalesce_incremental-loading) (version 1.1.0+)
  - [Dynamic Tables](https://docs.coalesce.io/page/package_coalesce_dynamic-tables) (version 1.1.3+)

## Step 2: Import the coalesceio SAP repository

1. In Github select create a new repository and select `Import a repository`.
2. The URL for the SAP source repository is https://github.com/coalesceio/sap.  This is a Public repository so no credentials are required.
3. Select an owner and give your repository a name.
4. After the import is complete you will see two branches available:
    - **sap_dynamic_tables** - This is an entire pipeline of Dynamic Tables to manage the SAP pipeline.
    - **sap_incremental_tables** - This is an entire pipeline of mostly incremental and persistent tables  
5. Either branch can be selected when setting up a Workspace which will be described below.

## Step 3: Set up a Project / Workspace in Coalesce

1. After the Git repo has been imported follow the Coalesce [documentation](https://docs.coalesce.io/docs/projects#create-a-new-project) to create a new project.  Initially, choose the option `Skip and Create` in the window for `Setup Version Control`.  We will connect to the Git repository after creating a Workspace.
Once the Project has been created select `Create Workspace`.  Enter a name and meaninful desription based on the Git branch you want to start from, either Dynamic Table or full load based.
2. At this point we are going to set up version control.  Select `Project Settings` and in the [Git Repository](https://docs.coalesce.io/docs/changing-a-git-repository-in-coalesce) section enter the URL of the repository you imported into your Git account as the Git Repository URL.
3. Save the `Project Settings`.
4. If you have enabled security for your Git repo, [Configure Git Account](https://docs.coalesce.io/docs/set-up-your-git-integration#add-through-the-project-dashboard).
5. After configuring the Git repo select `Launch` to launch the Workspace so we can attach it to a Git branch.
6. A Workspace can be attached to a branch by either selecting the `Git` modal or selecting `git branch` from the Workspace warning message `"Finish setting up version control for this workspace and avoid losing any work. Attach this workspace to a git branch"`.
7. After the `Attach Workspace to Branch` opens select the desired branch - **sap_dynamic_tables** or **sap_incremental_tables** to attach and `Attach` it.
8. Click on the `Git` modal, navigate to the `Branches` tab and select the [Branch Action](https://docs.coalesce.io/docs/git-branches#branch-actions) `Force Checkout` to populate the workspace with the latest Git commit.
9. This will overwrite any uncommitted work in the Workspace, which is what we want, so you will be required to confirm the Force Checkout by typing **FORCE** in the screen.
10. At this point the DAG objects should appear in your Workspace with errors.  Some workspace configuration is required to fix these errors.

## Step 4: Workspace Configuration

In this section you will configure the following Workspace settings:
- **Build Settings** - Configure [Storage Locations](https://docs.coalesce.io/docs/storage-locations-and-storage-mappings)
- **Workspace** - Configure Settings, User Credentials, Storage Mappings and Parameters

### Build Settings
Build Settings changes are required for both the `sap_dynamic_tables` and `sap_incremental_tables` versions of the Sap pipeline.

#### Storage Locations
The pipeline equires four Storage Locations be created.
- **Target (Default Location)** - The final output of the pipeline is stored in the `TARGET` location.  
- **Source** - Source location where Fivetran replicated tables are locations

#### Environments
Environments must be configured in order to deploy pipeline to higher level environments (QA, UAT, Pre-Prod, Prod, etc.) based on how you are managing your environments.

If implementing the `sap_dynamic_tables` version of the pipeline then the parameters reference in Workspace Settings below will be required.

If implementing the `sap_incremental_tables` then no parameters are required.

### Workspace Settings
In Workspace Settings the only difference between the `sap_dynamic_tables` and `sap_incremental_tables` versions of the SAP pipeline is that the `sap_dynamic_tables` version requires some parameters to be created and set.  Other configurations are the same.

- **Settings** - Configure the Snowflake account that Coalesce will be utilizing
- **User Credentials / OAuth Settings** - Enter the credentials required to connect to Snowflake
- **Storage Mappings** - This can be configured to use one database / schema for all Storage Locations or up to four database / schema mappings, one for each Storage Location, depending on whether or not you want to seperate Source, Staging, Intermediate and Target objects.
- **Parameters** - The Dynamic Tables in the sap pipeline require two Parameters to function.  

    The first, `targetDynamicTableWarehouse` is the standard Dynamic Table Parameter described in the Dynamic Table Package documentation.  
    
    The second one, `sapPipelineWarehouse` is specific to this pipeline.  It allows you to set a warehouse for the entire pipeline using a parameter instead of configuring individual nodes.  
    
    This parameter can be used or individual nodes can have their configs updated to use different warehouses.

```json
{
    "targetDynamicTableWarehouse": "DEV ENVIRONMENT",
    "sapPipelineWarehouse": "<warehouse_name>"
}
```

# Running / Deploying the Pipeline
The way the SAP pipeline is deployed and run differs between the two versions of the pipeline.

## Dynamic Table DAG Specifics
### Pipeline LAG
The pipeline is comprised of 17 sources and 18 Dynamic Tables.  The entire Dynamic Table DAG refresh is controlled using the `Downstream` option.  The Dynamic Table which controls the `LAG` of the entire pipeline is named `PIPELINE_SYNCED`.  The option delivered with the pipeline sets a target lag = 1 Day for the entire pipeline.

The target lag of the pipeline can be changed by changing the `Lag Specification` of the PIPELINE_SYNCED node.

Alternatively, individual nodes could be changed from Downstream to a Lag Specification.

### Missing Sources
If there are sources not available in your specific case (SPRINT, COMPONENT, VERSION), nodes related to those areas can be deleted from the pipeline.  This will require modification of any downstream objects that rely on said sources, but should be quick to figure out utilizing Coalesce object and column level lineage capabilities.

### Executing Pipeline

After the Workspace has been configured commit any changes into Git.  If the only changes have been Build Settings and Workspace Settings then there may be nothing to commit.

At this point you can create the pipeline.  The easiest way to do this is select `Create All` from the Graph action menu.

Assuming there are no errors the SAP Pipeline has now been deployed into your Development environment.  From here you can deploy to higher level environments, assuming you have created environments, utilizing the standard Coalesce deployment mechanisms.

Dynamic Tables are DDL only Snowflake objects, meaning there is no `Run` component to execute.  There are no Jobs to execute either.

## Incremental table Specifics
### Pipeline Definition
The pipeline is comprised of 17 sources, 16 incremental table, 11 persistent table and 6 views .  

### Missing Sources
If there are sources not available in your specific case (SPRINT, COMPONENT, VERSION), nodes related to those areas can be deleted from the pipeline.  This will require modification of any downstream objects that rely on said sources, but should be quick to figure out utilizing Coalesce object and column level lineage capabilities.

### Executing Pipeline

After the Workspace has been configured commit any changes into Git.  If the only changes have been Build Settings and Workspace Settings then there may be nothing to commit.

At this point you can create the pipeline.  The easiest way to do this is select `Create All` from the Graph action menu.

Assuming there are no errros the sap Pipeline has now been deployed into your Development environment. 

From here you can deploy to higher level environments, assuming you have created environments, utilizing the standard Coalesce deployment mechanisms.

Once successfully deployed, execute the Job **sap_model_job** against an environment.

# How is this pipeline maintained and can I contribute?
## Pipeline Maintenance
The Coalesce team maintaining this package _only_ maintains the latest version of the package. 

## Contributions
A small team of analytics engineers at Coalesce develops these pipelines. However, the pipelines are made better by community contributions! 

# Are there any resources available?
- If you have questions or want to reach out for help, please refer to the [GitHub Issue](https://github.com/coalesceio/sap/issues) section to find the right avenue of support for you.
- If you would like to provide feedback to the Coalesce pipeline team or would like to request a new Coalesce pipeline, fill out our [Feedback Form](https:tbd).
- Any other questions about Coalesce then check out our [Help Center](https://help.coalesce.io/)
