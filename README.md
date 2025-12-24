# 🚀 AppExchanger — Installation and Configuration Guide

This guide explains how to install and configure the **AppExchanger** package and deploy the required AWS Lambda functions. Once configured, supervisors, managers, and admins can manage **Hours of Operation**, **Queue Management**, **User Management**, and **Routing Profiles** directly from the Salesforce UI.

## 🧭 Table of Contents

1.  [Prerequisites](https://www.google.com/search?q=%231-prerequisites)
2.  [What this Package Provides](https://www.google.com/search?q=%232-what-this-package-provides)
3.  [Package Installation (Salesforce AppExchange)](https://www.google.com/search?q=%233-package-installation-salesforce-appexchange)
4.  [Download Lambda Packages from Git](https://www.google.com/search?q=%234-download-lambda-packages-from-git)
5.  [Add Lambda Functions to your AWS Org (Step-by-Step)](https://www.google.com/search?q=%235-add-lambda-functions-to-your-aws-org-step-by-step)
      * [5.1 Create or Choose an IAM Role for Execution](https://www.google.com/search?q=%2351-create-or-choose-an-iam-role-for-execution)
      * [5.2 Create the Lambda Functions (Console Method)](https://www.google.com/search?q=%2352-create-the-lambda-functions-console-method)
      * [5.3 Upload Function Code](https://www.google.com/search?q=%2353-upload-function-code)
      * [5.4 Create Function URL (Public HTTPS Endpoint)](https://www.google.com/search?q=%2354-create-function-url-public-https-endpoint)
      * [5.5 Add Lambda Permission so Amazon Connect Info is Fetched (AWS Console Method)](https://www.google.com/search?q=%2355-add-lambda-permission-so-amazon-connect-info-is-fetched-aws-console-method)
6.  [Update Salesforce Custom Metadata with Lambda URLs](https://www.google.com/search?q=%236-update-salesforce-custom-metadata-with-lambda-urls)
7.  [Add Lightning Web Component (LWC) to the Lightning Page](https://www.google.com/search?q=%237-add-lightning-web-component-lwc-to-the-lightning-page)
8.  [Post-Install Verification and Smoke Tests](https://www.google.com/search?q=%238-post-install-verification-and-smoke-tests)
9.  [Troubleshooting & Common Issues](https://www.google.com/search?q=%239-troubleshooting--common-issues)

-----

## 1\) Prerequisites

To successfully deploy and configure AppExchanger, you require:

  * **Salesforce:** Org with **System Administrator** access.
  * **Package:** The **AppExchanger** package file (installed from Salesforce AppExchange).
  * **AWS Account:** The AWS account/org used as the telephony provider, connected to your Salesforce org.
    > The **Amazon Connect instance** must be in the **same AWS region** where Lambda functions will be created.
  * **Source Code:** Access (download or clone) to the Git repository containing the **8 Lambda packages**.
  * **AWS Permissions:** AWS Console access with permissions to:
      * Create Lambda functions and Function URLs.
      * Attach roles/policies and add permissions (or AWS CLI access).
  * **Knowledge:** Basic knowledge of Salesforce custom metadata and Lightning App Builder.

-----

## 2\) What this Package Provides

After installation and configuration, supervisors/managers/admins will be able to manage the following features directly from the Salesforce UI:

  * **Hours of Operation**
  * **Queue Management**
  * **User Management**
  * **Routing Profile Management**

💡 **Note:** Salesforce calls AWS Lambda endpoints to enable these features. You will deploy 8 Lambda functions and register their **Function URLs** in the package's custom metadata.

-----

## 3\) Package Installation (Salesforce AppExchange)

1.  Login to your Salesforce org as a **System Administrator**.
2.  Go to **AppExchange** and find **SCV Amplifier** (or use the direct install link provided).
3.  Click **Get It Now** $\rightarrow$ choose the target org (**Production or Sandbox**).
4.  Select **Install for Admins Only** (or All Users as required).
5.  Wait for the installation to complete. Grant required permissions when prompted.

-----

## 4\) Download Lambda Packages from Git

1.  Clone or download the repository that contains the **6 Lambda packages**.
2.  You should find 6 directories/files—one corresponding to each Lambda function. **Download all of them.**

-----

## 5\) Add Lambda Functions to your AWS Org (Step-by-Step)

⚠️ **Important:** Ensure you create Lambda functions in the **same AWS region** where your Amazon Connect instance is deployed.

### 5.1 Create or Choose an IAM USER

1.  In AWS Console $\rightarrow$ **IAM** $\rightarrow$ **Users** $\rightarrow$ **Create role**.
2.  Name the user
3.  Attach the following managed policies:
      * `AWSLambdaBasicExecutionRole` (for CloudWatch logs)
      * `AmazonConnectFullAccess` (if the lambda needs to interact with Amazon Connect APIs)
      * Any additional policies needed for your environment (e.g., S3, Secrets Manager, DynamoDB).
4.   After Save, Capture the IAM Role ARN, Secret Key and Access Key. (for Salesforce Name Credential).

### 5.2 Create the Lambda Functions (Console Method)

Repeat the following steps for **each** of the 8 Lambda packages:

1.  AWS Console $\rightarrow$ **Lambda** $\rightarrow$ **Functions** $\rightarrow$ **Create function**.
2.  Choose **Author from scratch**.
3.  **Function name:** Use a clear name, e.g. `appex-hours-of-operation`.
4.  **Runtime:** Choose the runtime used in your package (e.g., `nodejs18.x`, `python3.11`).
5.  **Execution role:** Choose **Use an existing role** and select the role created in step 5.1.
6.  Click **Create function**.

### 5.3 Upload Function Code

1.  On the newly created function page, go to the **Code** section.
2.  Click **Upload from** $\rightarrow$ **.zip file** and upload the corresponding zip from the Git repo.
3.  Verify the **Handler** setting matches the package's configuration (e.g., `index.handler` or `app.lambda_handler`).
4.  Click **Deploy**.

### 5.4 Create Function URL

Repeat for all 8 functions. The **Function URL** is the public endpoint Salesforce will call.

1.  On the Lambda function page $\rightarrow$ **Configuration** tab $\rightarrow$ **Function URL**.
2.  Click **Create function URL**.
3.  **Auth type:**choose **AWS_IAM** if you secure calls with IAM (requires SigV4 in Salesforce).*
4.  Click **Create**.
5.  **Copy the Function URL value** (it will look like `https://<xyz>.lambda-url.<region>.on.aws/`). **Save this URL** for the next step.

### 5.5 Add Lambda Permission so Amazon Connect Info is Fetched (AWS Console Method)

You must grant Amazon Connect permission to invoke your Lambda functions if calls originate from Amazon Connect (e.g., from a contact flow) to all deployed Lambda functions.

Repeat these steps for **each** of the 8 Lambda functions:

1.  **Open the Lambda Function:** Navigate to the specific Lambda function in the AWS Console.
2.  Go to **Configuration**: Click on the **Configuration** tab.
3.  Find Resource-Based Policy: In the left sidebar under Configuration, click on **Permissions**.
4.  Add Permission: Scroll down to the **Resource-based policy** section and click **Add permissions**.
5.  Configure Permission Details: Attached permission **“Amazon connect full Access”**
6.  Save the Permission: Click **Save**.

-----

## 6\) Update Salesforce Named Credentials with Lambda URLs

Salesforce uses **Named Credentials** to securely call the external AWS endpoints.

1.  Login to **Salesforce as Admin**.
2.  Go to **Setup**.
3.  **User Permissions:** In Quick Find, search for and go to your **User**. Ensure your user is assigned the necessary **SCV permission set**.
4.  **External Credentials:**
      * In Quick Find, type `Named Credential` $\rightarrow$ **External Credentials**.
      * Find the record for **AWS Lambda Auth**.
      * Go to the **Principals** section $\rightarrow$ Update the **AWS_Cred Principal**.
      * Enter the correct **Access Key**, **Secret Key** (for a dedicated IAM User), and **IAM Role ARN** (if used for invocation).
5.  **Named Credentials (URLs):**
      * Navigate back to **Named Credentials**.
      * Update the **URL** for all **6 Named Credentials** related to the Lambda functions. Each function's Named Credential must be updated with its corresponding **Function URL** copied in step 5.4.
-----

## 7\) Add Lightning Web Component (LWC) to the Lightning Page

1.  Go to the target Lightning App (via App Launcher) and open the Lightning page where you want the AppExchanger UI to be located.
2.  Click the gear icon ($\text{⚙️}$) $\rightarrow$ **Edit Page**.
3.  In the Lightning App Builder, find the **LWC component** provided by the package (e.g., `appexchangerManager`).
4.  **Drag the component** onto the desired region of the page layout.
5.  **Save** and **Activate** (if required) for the relevant profiles or app defaults.
6.  Refresh the Lightning page and verify the component loads.

-----

## 8\) Post-Install Verification and Smoke Tests

1.  In the Salesforce UI, open the page containing the LWC.
2.  Attempt the following basic read/write actions:
      * **Read** Hours of Operation data.
      * **Edit and Save** Hours of Operation.
      * **Add** a test queue entry.
      * **Update** routing profile fields.
3.  For each action, **monitor CloudWatch logs** for the invoked Lambda function(s).
4.  If functions fail, capture the CloudWatch log stream, timestamp, and the Salesforce request payload for troubleshooting.

-----

## 9\) Troubleshooting & Common Issues

| Issue | Potential Cause / Solution |
| :--- | :--- |
| Lambda returns 403/401 | **AuthType=NONE:** Check Lambda code for expected secret header validation. Salesforce must include any required headers. **AuthType=AWS\_IAM:** Ensure Salesforce is calling via correctly signed (SigV4) requests. |
| Function URL not reachable | Verify the Function URL value is correct in the Salesforce Named Credentials. Test the URL directly using `curl` or Postman. |
| `add-permission` failed (duplicate) | AWS CLI will error if a statement with the same `--statement-id` exists. Use a unique statement ID or use `aws lambda remove-permission` to delete the old one. |
| Wrong AWS Region | Lambda functions, Amazon Connect, and Function URLs must be in the **same region**. Double-check the region in the Lambda console header. |
| Logs not visible in CloudWatch | Ensure the Lambda execution role has the necessary CloudWatch permissions (e.g., `AWSLambdaBasicExecutionRole` grants `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents`). |
| Salesforce metadata not saving | Confirm you have permission to edit package custom metadata. Some package metadata records are locked by the author; contact the package vendor if necessary. |

-----

Would you like me to elaborate on any of the steps, such as setting up the Named Credentials in Salesforce?
