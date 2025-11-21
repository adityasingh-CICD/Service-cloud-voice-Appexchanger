# AppExchanger — README

> This README explains how to install and configure the **AppExchanger** package and add the required AWS Lambda functions so supervisors, managers, and admins can manage Hours of Operation, Queue Management, User Management, and Routing Profiles directly from the Salesforce UI.

---

## Table of Contents

1. Prerequisites
2. What this package provides
3. Package installation (Salesforce AppExchange)
4. Download Lambda packages from Git
5. Add Lambda functions to your AWS Org (step-by-step)

   * Create Lambda functions in console
   * Set execution role and policies
   * Upload code
   * Configure function URL
   * Add Lambda permissions for Amazon Connect
6. Update Salesforce custom metadata with Lambda URLs
7. Add LWC to Lightning page
8. Post-install verification and smoke tests
9. Troubleshooting & common issues

---

## 1) Prerequisites

* Salesforce org with System Administrator access.
* AppExchanger package file (installed from Salesforce AppExchange).
* AWS account/org used as the telephony provider and connected to your Salesforce org (Amazon Connect instance in the same AWS region where Lambda functions will be created).
* Access to the Git repository that contains the 8 Lambda packages (download or clone access).
* AWS Console access with permissions to create Lambda functions, create function URLs, attach roles/policies and add permissions (or AWS CLI access).
* Basic knowledge of Salesforce custom metadata and Lightning App Builder.

---

## 2) What this package provides

After installation and configuration, supervisors/managers/admins will be able to manage from Salesforce UI:

* Hours of Operation
* Queue Management
* User Management
* Routing Profile Management

To enable those features, Salesforce calls AWS Lambda endpoints. You will deploy 8 Lambda functions and register their function URLs in the package's custom metadata.

---

## 3) Package installation (Salesforce AppExchange)

1. Login to your Salesforce org as a System Administrator.
2. Go to AppExchange and find **AppExchanger** (or use the direct install link provided by your vendor).
3. Click **Get It Now** → choose the target org (Production or Sandbox) → Install for Admins Only (or All Users as required).
4. Wait for installation to complete. Grant required permissions when prompted.

**Note:** After the package installs, a custom metadata type named **`Lambda_Code_URL__mdt`** (or similar) will be available — this is where you'll paste the function URLs later.

---

## 4) Download Lambda packages from Git

1. Clone or download the repo that contains the 8 Lambda packages.

2. You should find 8 directories/files — one per lambda, download those.

Adjust names above to match the repository.

## 5) Add Lambda functions to your AWS Org (step-by-step)

> **Important:** Ensure you create Lambda functions in the **same AWS region** where your Amazon Connect instance is deployed.

### 5.1 Create or choose an IAM execution role

1. In AWS Console → IAM → Roles → Create role.
2. Choose **AWS service** → **Lambda**.
3. Attach managed policies:

   * `AWSLambdaBasicExecutionRole` (for CloudWatch logs)
   * `AmazonConnectFullAccess` (if the lambda needs to interact with Amazon Connect APIs)
   * Any additional policies needed for your environment (e.g., access to S3, Secrets Manager, DynamoDB, etc.).
4. Name role e.g. `lambda-appExchanger-execution-role` and create it.

### 5.2 Create the Lambda functions (Console method)

Repeat the following steps for each of the 8 Lambda packages:

1. AWS Console → Lambda → Functions → **Create function**.
2. Choose **Author from scratch**.
3. Function name: use a clear name, e.g. `appex-hours-of-operation`.
4. Runtime: choose the runtime used in your package (e.g., `nodejs18.x`, `python3.11`, etc.).
5. Execution role: choose **Use an existing role** and select `lambda-appExchanger-execution-role` (or the role you created).
6. Click **Create function**.

### 5.3 Upload function code

1. On the function page, under **Code** section click **Upload from** → `.zip file` and upload the corresponding zip from the repo.
2. Verify the handler setting matches the package's handler configuration (e.g., `index.handler` or `app.lambda_handler`).
3. Save and **Deploy**.

### 5.4 Create Function URL (public HTTPS endpoint)

AppExchanger expects publicly reachable function URLs which are saved into Salesforce custom metadata.

**Console method:**

1. On the Lambda function page → **Configuration** → **Function URL**.
2. Click **Create function URL**.
3. Auth type: choose **None** (if the design expects an open endpoint) or **AWS_IAM** if you want to secure calls with IAM. **Most AppExchanger installs expect function URLs with no auth** and restrict access via additional checks; confirm with your vendor/security team.
4. Click **Create**.
5. Copy the **Function URL** value (it will look like `https://<xyz>.lambda-url.<region>.on.aws/`)

# Fetch URL
aws lambda get-function-url-config --function-name appex-hours-of-operation

### 5.5 Add Lambda Permission so Amazon Connect (or other services) can invoke it

You must grant Amazon Connect permission to invoke the function if calls come from Amazon Connect. Change user role to 'lambda-appExchanger-execution-role' 

## 6) Update Salesforce custom metadata with Lambda URLs

1. Login to Salesforce as Admin.
2. Go to **Setup** → in Quick Find type **Custom Metadata** → **Manage Records** for the package's custom metadata (`Lambda_Code_URL__mdt` or similar).
3. Edit the existing metadata records or create new ones. Example fields you may see:

   * **Name**: `Appex_HoursOfOperation` (or function-key)
   * **Lambda_Url__c**: paste the Function URL (e.g., `https://...lambda-url.../`)
   * **Description**: optional
4. Repeat for all 8 Lambda functions — each function must have its corresponding metadata record updated with the correct URL.

## 7) Add Lightning Web Component (LWC) to the Lightning page

1. Go to the Lightning App (App Launcher) and open the Lightning page where you want the AppExchanger UI.
2. Click the gear icon → **Edit Page**.
3. In the Lightning App Builder, find the LWC component provided by the package (name will vary; example: `appexchangerManager`).
4. Drag the component onto the desired region of the page.
5. Save and **Activate** (if required) for profiles or app.
6. Refresh the Lightning page and verify the component loads and calls the backend lambdas (you may see loading spinners while it fetches configuration).

## 8) Post-install verification and smoke tests

1. In Salesforce UI, open the page with the LWC.
2. Try basic actions such as:

   * Read Hours of Operation
   * Edit and Save Hours of Operation
   * Add a test queue entry
   * Update routing profile fields
3. For each action, monitor CloudWatch logs for the Lambda function(s) invoked.
4. If functions fail, capture the CloudWatch log stream, timestamp, and the Salesforce request payload for troubleshooting.

---

## 9) Troubleshooting & common issues

**1. Lambda returns 403/401 when called from Salesforce**

* If you used `AuthType=NONE` for function URLs: check your Lambda code for header validation. Salesforce must include any expected secret headers.
* If you used `AWS_IAM` auth: ensure you are calling via signed requests (SigV4) or use a service that signs requests.

**2. Function URL not reachable**

* Verify the function URL value is correct in custom metadata.
* Test the function URL in curl or Postman.

**3. `add-permission` failed or duplicate statement**

* `aws lambda add-permission` will error if a statement with the same `--statement-id` already exists. Use a unique statement id per function.
* Use `aws lambda remove-permission` to remove an existing statement and re-run add-permission.

**4. Wrong AWS Region**

* Lambda functions, Amazon Connect, and Function URLs must be in the same region. Double-check region in the Lambda console header.

**5. Logs not visible**

* Ensure the Lambda execution role has `logs:CreateLogGroup`, `logs:CreateLogStream`, `logs:PutLogEvents` (managed by `AWSLambdaBasicExecutionRole`).

**6. Salesforce metadata not saving**

* Confirm you have correct permissions to edit package custom metadata; some package metadata records are editable only if the package author allowed it. If locked, contact the package vendor.

*End of README*
