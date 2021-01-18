
![License: CISCO](https://img.shields.io/badge/License-CISCO-blue.svg)
[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/)

# SecureX orchestration workflow: AMP MSSP customer events to SecureX incident and SNOW incident [DRAFT, NOT FINISHED]

This is a set of sample workflows to work with the MSSP environment of Cisco Secure Endpoint (formerly known as Advanced Malware Protection for Endpoints (AMP4E)). It can obtain events from the various customers and create Securex and ServiceNow incidents based on these security events.

## Index

1. [Features and flow](##features-and-flow)
2. [Installation](#installation)
    1. [Create table to store encoded MSSP API keys](#create-table-to-store-encoded-mssp-api-keys)
    2. [Import the first workflow to add encoded AMP API keys to table](#import-the-first-workflow-to-add-encoded-amp-api-keys-to-table)
    3. [Import the second workflow to retrieve AMP events and create SecureX and ServiceNow incidents](#import-the-second-workflow-to-retrieve-amp-events-and-create-securex-and-servicenow-incidents)
    4. [Import the third workflow that sets a global variable containing the ID of the second workflow](#import-the-third-workflow-that-sets-a-global-variable-containing-the-id-of-the-second-workflow)
    5. [Import the fourth workflow that is triggered when ServiceNow incident is closed](#import-the-fourth-workflow-that-is-triggered-when-servicenow-incident-is-closed)
    6. [Testing and running the solution](#testing-and-running-the-solution)
3. [Notes](#notes)
4. [Author(s)](#authors)

## Features and flow
1. The **first** workflow ([ADD-AMP-MSSP-CREDS.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/ADD-AMP-MSSP-CREDS.json)) will be able to obtain user input to add Cisco Secure Endpoint (formerly known as Advanced Malware Protection for Endpoints (AMP4E)) API Credentials + customer name and store them base 64 encoded in a table. Please note that the credentials are base 64 encoded, however are stored in the global table variable. SecureX is secured with MFA, but this still needs to be taken into consideration. This workflow only needs to be run initially and every time you add a customer to your MSSP portal.
2. The **second** workflow ([AMP-MSSP-TO-SNOW.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/AMP-MSSP-TO-SNOW.json)) will loop through these API keys and obtain the AMP events for the past 5 minutes. This workflow can be scheduled to run every 5 minutes (or otherwise). It is also possible to configure which events are deemed as important to retrieve. The suggestion is to retrieve only high priority events, such as events with a `HIGH` or `CRITICAL` severity. This workflow will then create a SecureX incident, as well as a ServiceNow incident. It will make sure the ServiceNow incidents has information to close the loop back to SecureX by closing the incident. Optionally this workflow is able to isolate the AMP host or move it to a Triage group. 
3. The **third** workflow ([SET-SNOW-RESPONSE-WF-ID.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/SET-SNOW-RESPONSE-WF-ID.json)) only needs to be run once initially. This workflow sets a global variable containing the ID of the second workflow. This is needed by ServiceNow (using an outbound API call) to run the fourth and final workflow of this solution.
4. The **fourth** workflow ([SNOW-TO-AMP.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/SNOW-TO-AMP.json)) will be able to close the SecureX incident when the ServiceNow incident is closed. It will also optionally be able to stop the AMP host isolation and move the host back to its original group. The ServiceNow incident ID will be added to the SecureX incident to fully sync the 2 systems. This workflow will be called via an outbound API call from ServiceNow.

Below you can view the current workflows. Please feel inspired to add to it as you see fit. **Please always test thoroughly before using in production!**

![](screenshots/mssp-amp-overview-add-new.png)

**MORE SCREENSHOTS**

# Installation

## Create table to store encoded MSSP API keys

### Required workflows, targets and accounts keys

* None

### Installation steps:

1. Browse to your SecureX orchestration instance. This wille be a different URL depending on the region your account is in: 

* US: https://securex-ao.us.security.cisco.com/orch-ui/workflows/
* EU: https://securex-ao.eu.security.cisco.com/orch-ui/workflows/
* APJC: https://securex-ao.apjc.security.cisco.com/orch-ui/workflows/

2. Before we get started with importing the workflows, we will create a new variable type and a global variable. This will store our encoded API credentials. In the left hand menu, go to **Variables**:

![](screenshots/mssp-amp-left-menu.png)

3. Click on the **Variable Types** tab and then click on **NEW VARIABLE TYPE**:

![](screenshots/mssp-amp-var-type.png)

4. Now we are going to create a new table. Please fill in the details exactly as below and click **SUBMIT**:

* Variable type **DISPLAY NAME** `AMP_MSSP_credentials`.
* **Required** column 1: **FIELD NAME** `customer_name`, **FIELD TITLE** `Customer Name` and **FIELD TYPE** `String`.
* **Required** column 1: **FIELD NAME** `encoded_api_credentials`, **FIELD TITLE** `Encoded API Credentials` and **FIELD TYPE** `String`.

![](screenshots/mssp-amp-new-var-type.png)

5. Now go to the **Global Variables** tab and then click on **NEW VARIABLE**:

![](screenshots/mssp-amp-global-var.png)

6. From the first drop down menu (**Data Type**) select the type that you have just created (`AMP_MSSP_credentials`). 

* **DISPLAY NAME** `AMP_MSSP_credentials_table`.
* **SCOPE** `Global`.
* Fill in 1 dummy row to initiate the variable

![](screenshots/mssp-amp-new-global.png)

## Import the first workflow to add encoded AMP API keys to table

### Required workflows, targets and accounts keys

* Main Workflow: [ADD-AMP-MSSP-CREDS.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/ADD-AMP-MSSP-CREDS.json) 

### Installation steps:

1. Browse to the **Workflows** section in the left pane menu.

2. Click on **IMPORT** to import the workflow:

![](screenshots/import-workflow.png)

3. Click on **Browse** and copy paste the content of the [ADD-AMP-MSSP-CREDS.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/ADD-AMP-MSSP-CREDS.json) file inside of the text window. 

![](screenshots/copy-paste.png)

4. Click on **IMPORT**. You can now select the workflow and inspect it. Click **RUN** in the rop right corner: 

![](screenshots/mssp-amp-run-add.png)

5. You will now be prompted to enter the `CUSTOMER NAME`, `AMP API CLIENT ID` and `AMP API CLIENT SECRET`. Run this workflow as many times as needed (as many times as you have customers).

![](screenshots/mssp-amp-user-prompt.png)

## Import the second workflow to retrieve AMP events and create SecureX and ServiceNow incidents. 

### Required workflows, targets and accounts keys

* Atomic Workflow: **Generate Access Token for SecureX**, **Copy-Threat Response - Create Incident**, **AMP - Move Computer to Group**, **AMP - Isolate Host**, **Service Now - Create Incident**, **Service Now - Add Work Note to Incident**
* Main Workflow: [AMP-MSSP-TO-SNOW.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/AMP-MSSP-TO-SNOW.json)
* Target: **Private Intel Base**, **ServiceNow**, **AMP_Target**
* Account keys: **Private Intel Base**, **ServiceNow**

### Installation steps:

1. Browse to the **Workflows** section in the left pane menu.

2. Click on **IMPORT** to import the workflow.

3. Click on **Browse** and copy paste the content of the [AMP-MSSP-TO-SNOW.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/AMP-MSSP-TO-SNOW.json) file inside of the text window. 

4. Click on **IMPORT**. You might receive an error that information is missing.

5. Click on **UPDATE** and fill in the CTR (SecureX threat response) keys. These are not stored as plain text, as they are stored as "secure strings" in SecureX.

> **Note:** To obtain the threat response API keys, create one here: https://securex.us.security.cisco.com/settings/apiClients. Please change the _.us._ in the url to _.eu._ or _.apjc._ respectively for the European or Asian instances. If you are using the EU or APJC instance, you will also need to change the target of the `CTRGenerateAccessToken` and `CTR Create Incident` activities in the workflow. You do this by clicking on the activity and scrolling to the `target` section. **Make sure to do this for all 4 related CTR targets!**

## Import the third workflow that sets a global variable containing the ID of the second workflow

### Required workflows, targets and accounts keys

* Atomic Workflow: **Generate Access Token for SecureX**, **List CTR response actions** 
* Main Workflow: [SET-SNOW-RESPONSE-WF-ID.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/SET-SNOW-RESPONSE-WF-ID.json)
* Target: **CTR API Target**
* Account keys: **CTR_Credentials**

### Installation steps:

1. Browse to the **Workflows** section in the left pane menu.

2. Click on **IMPORT** to import the workflow.

3. Click on **Browse** and copy paste the content of the [SET-SNOW-RESPONSE-WF-ID.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/SET-SNOW-RESPONSE-WF-ID.json) file inside of the text window. 

4. Click on **IMPORT**. You might receive an error that information is missing.

5. Click on **UPDATE** and fill in the CTR (SecureX threat response) keys.

6. After importing this workflow, you can open it and then click **RUN** in the top right corner. You will not have to use this workflow again after taking this action.

## Import the fourth workflow that is triggered when ServiceNow incident is closed

### Required workflows, targets and accounts keys

* Atomic Workflow: **Generate Access Token for SecureX**, **AMP - Move Computer to Group**
* Main Workflow: [SNOW-TO-AMP.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/SNOW-TO-AMP.json)
* Target: **CTR API Target**, **AMP_Target**
* Account keys: **CTR_Credentials**

### Installation steps:

1. Browse to the **Workflows** section in the left pane menu.

2. Click on **IMPORT** to import the workflow.

3. Click on **Browse** and copy paste the content of the [SNOW-TO-AMP.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/SNOW-TO-AMP.json) file inside of the text window. 

4. Click on **IMPORT**. You might receive an error that information is missing.

5. Click on **UPDATE** and fill in the CTR (SecureX threat response) keys.

#### **IMPORTANT** this workflow is not working right now. Needs troubleshooting!!!

## Testing and running the solution

1. Now it is time to test, click on **RUN** in the top right of your window, and everything shopuld be working now. If not try troubleshooting by click on the activity that is colored red. 

2. As a final step you could choose to enable to scheduled trigger for the [AMP-MSSP-TO-SNOW.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/AMP-MSSP-TO-SNOW.json) workflow. This is recommended, as the workflow only retrieves the security events of the last 5 minutes. By scheduling it, the Security analysts will be updated every hour for potential new malicious activity. To enable the trigger, click on the hyperlink below and uncheck the `DISABLE TRIGGER` checkbox. This can be found in the workflow properties in the right menu pane. 

![](screenshots/trigger-5min.png)

![](screenshots/trigger-5min-disabled.png)

> **Note:** make sure not to select an activity when looking for the global workflow properties.

## Notes

* Please test this properly before implementing in a production environment. This is a sample workflow!
* The roadmap will include a webhook based trigger, instead of a scheduled run. 

## Author(s)

* Christopher van der Made (Cisco)
