
![License: CISCO](https://img.shields.io/badge/License-CISCO-blue.svg)
[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/)

# SexureX orchestration workflow: AMP MSSP customer events to SecureX incident and SNOW incident [DRAFT, NOT FINISHED]

This is a set of sample workflows to work with the MSSP environment of Cisco Secure Endpoint (formerly known as Advanced Malware Protection for Endpoints (AMP4E)). It can obtain events from the various customers and create Securex and ServiceNow incidents based on these security events.


## Features
* The first workflow ([ADD-AMP-MSSP-CREDS.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/ADD-AMP-MSSP-CREDS.json)) will be able to obtain user input to add Cisco Secure Endpoint (formerly known as Advanced Malware Protection for Endpoints (AMP4E)) API Credentials + customer name and store them base 64 encoded in a table. Please note that the credentials are base 64 encoded, however are stored in the global table variable. SecureX is secured with MFA, but this still needs to be taken into consideration. 
* The second workflow (ADD JSON FILE HERE) will loop through these API keys and obtain the AMP events for the past 5 minutes. This workflow can be scheduled to run every 5 minutes. It is also possible to configure which events are deemed as important to retrieve. The suggestion is to retrieve only high priority events, such as events with a `HIGH` or `CRITICAL` severity. This workflow will then create a SecureX incident, as well as a ServiceNow incident.
* The third workflow (ADD JSON FILE HERE) will be able to close the SecureX incident when the ServiceNow incident is closed.

Below you can view the current workflows. Please feel inspired to add to it as you see fit. **Please always test thoroughly before using in production!**

![](screenshots/mssp-amp-overview-add-new.png)

## Installation

### Create table to store encoded MSSP API keys

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


### Import first workflow to add encoded AMP API keys to table
1. Browse to the **Workflows** section in the left pane menu.

2. Click on **IMPORT** to import the workflow:

![](screenshots/import-workflow.png)

3. Click on **Browse** and copy paste the content of the [ADD-AMP-MSSP-CREDS.json](https://raw.githubusercontent.com/chrivand/amp-mssp-events-to-snow/main/ADD-AMP-MSSP-CREDS.json) file inside of the text window. 

![](screenshots/copy-paste.png)

4. Click on **IMPORT**. You will now receive an error that information is missing: 

![](screenshots/missing-info.png)

** NEW CONTENT NEEDED BELOW (DO NOT USE)


5. Click on **UPDATE** and fill in the CTR (SecureX threat response), Meraki and Webex API key. These are not stored as plain text, as they are stored as "secure strings" in SecureX.

> **Note:** To obtain the threat response API keys, create one here: https://securex.us.security.cisco.com/settings/apiClients. Please change the _.us._ in the url to _.eu._ or _.apjc._ respectively for the European or Asian instances. It might be that you have these already created, just make sure it has at least the `Casebook` scope checked. If you are using the EU or APJC instance, you will also need to change the target of the `CTRGenerateAccessToken` and `CTR Create Casebook` activities in the workflow. You do this by clicking on the activity and scrolling to the `target` section. **Make sure to do this for all 4 related CTR targets!** Here is an example:

![](screenshots/edit-target.png)

![](screenshots/update-target.png)

> **Note:** To obtain your Meraki API key, please follow these steps: https://documentation.meraki.com/zGeneral_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API

> **Note:** Please retrieve your Webex key from: [https://developer.webex.com/docs/api/getting-started](https://developer.webex.com/docs/api/getting-started). Please be aware that the personal token from the getting started page only works for 12 hours. Please follow these steps to request a "bot" token: https://developer.webex.com/docs/integrations.

6. You are still missing 2 more values before you are done. Click on the workflow like below, and let's fill in the Meraki Org ID and Webex Team space ID.

![](screenshots/import-succes.png)

7. Click on the `Meraki Org ID` variable and fill in the Org ID of the Meraki organization that you want to track security events for. More info on this can be found here: https://documentation.meraki.com/zGeneral_Administration/Other_Topics/The_Cisco_Meraki_Dashboard_API#Organizations

![](screenshots/workflow-variables.png)

8. Next click on `webex space ID`. You can create a new space or find an existing one via these link: retrieve the Room ID from: https://developer.webex.com/docs/api/v1/rooms/list-rooms. You can also add the roomid@webex.bot bot to the room and it will send you the roomId in a private message and then remove itself from the room.

9. Now it is time to test, click on **RUN** in the top right of your window, and eveyrhting shopuld be working now. If not try troubleshooting by click on the activity that is colored red. 

![](screenshots/run.png)

10. As a final step you could choose to enable to scheduled trigger for this workflow. This is recommended, as the workflow only retrieves the security events of the last hour. By scheduling it, the Security analysts will be updated every hour for potential new malicious activity. To enable the trigger, click on the hyperlink below and uncheck the `DISABLE TRIGGER` checkbox. This can be found in the workflow properties in the right menu pane. 

![](screenshots/schedule.png)

> **Note:** make sure not to select an activity when looking for the global workflow properties.

## Notes

* Please test this properly before implementing in a production environment. This is a sample workflow!
* The roadmap will include a webhook based trigger, instead of a scheduled run. 

## Author(s)

* Christopher van der Made (Cisco)
