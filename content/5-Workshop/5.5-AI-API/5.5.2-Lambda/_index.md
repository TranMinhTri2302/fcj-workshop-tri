---
title : "Deploy AWS Lambda"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.5.2. </b> "
---

#### 5.5.2. Deploy AWS Lambda (Core Logic)
This Lambda function will be responsible for all the API processing logic of the system (Facial recognition, logging to DynamoDB, saving images to S3...). Since the source code uses external libraries like `FastAPI`, `mangum`, `boto3`, we cannot code directly on the AWS Console but must package the source code from our local computer.

**Step 1: Source Code Packaging**
1. Open Terminal (PowerShell) on your computer.
2. Run the following script to install the libraries into the `dist` folder and compress them into a `lambda_function.zip` file:
   ```powershell
   # 1. Navigate to the correct backend directory
   cd d:\AWS\smart-campus\backend

   # 2. Delete the old dist folder (if any)
   if (Test-Path "dist") { Remove-Item -Recurse -Force "dist" }

   # 3. Create a new dist folder
   New-Item -ItemType Directory -Force -Path "dist"

   # 4. Install libraries into the dist folder
   pip install -r requirements.txt -t ./dist

   # 5. Copy the app folder to dist
   Copy-Item -Path "app" -Destination "dist" -Recurse

   # 6. Compress the entire dist folder into lambda_function.zip
   Compress-Archive -Path "dist\*" -DestinationPath "lambda_function.zip" -CompressionLevel Optimal
   ```


**Step 2: Create a Lambda function on AWS**
1. Search for and access the **Lambda** service on the AWS Console.
> ![Search for Lambda](/aws-image/setupLambda/lambda3.png)
2. On the Lambda homepage, click the orange **Create a function** button.
> ![Create Lambda function](/aws-image/setupLambda/lambda4.png)
3. In the creation screen, select **Author from scratch**. Enter **Function name** as `smart-campus-api`. In the **Runtime** section, choose `Python 3.12`. Finally, scroll down and click **Create function**.
> ![Fill Lambda info](/aws-image/setupLambda/lambda5.png)

**Step 3: Upload source code and configure Handler**
1. On the screen of the newly created function, in the **Code** tab, click **Upload from > .zip file** and select the `lambda_function.zip` file created in Step 1.
> ![Upload Zip](/aws-image/setupLambda/lambda6.png)
2. Scroll down to the **Runtime settings** section, click the **Edit** button.
> ![Edit Runtime settings](/aws-image/setupLambda/lambda9.png)
3. Change the **Handler** to `app.main.handler` (because we use Mangum to wrap the FastAPI application). Click **Save**.
> ![Configure Handler](/aws-image/setupLambda/lambda10.png)

**Step 4: Configure Environment Variables**
1. Switch to the **Configuration** tab > **Environment variables** > **Edit**.
> ![Open Edit Environment variables](/aws-image/setupLambda/lambda15.png)
2. Add the environment variables according to the table below, getting values from the services created in previous steps:

| Variable Name (Key) | Sample Value | Where to get it? |
|---|---|---|
| `ENVIRONMENT` | `cloud` | Fixed, enter directly |
| `AWS_REGION` | `ap-southeast-1` | Fixed |
| `USERS_TABLE` | `smart-campus-users` | Table name created in 5.4.1 |
| `FACES_TABLE` | `smart-campus-faces` | Table name created in 5.4.1 |
| `ATTENDANCE_TABLE` | `smart-campus-attendance` | Table name created in 5.4.1 |
| `SECURITY_TABLE` | `smart-campus-security` | Table name created in 5.4.1 |
| `NOTIFICATIONS_TABLE` | `smart-campus-notifications` | Table name created in 5.4.1 |
| `FACE_COLLECTION_ID` | `smart-campus-faces` | Collection ID created in 5.5.1 |
| `IMAGE_BUCKET` | `smart-campus-images-{id}` | Bucket name created in 5.4.2 |
| `EVENT_BUS_NAME` | `default` *(or custom Event Bus name if any)* | Section 5.6.4 (EventBridge) |
| `SES_SENDER_EMAIL` | `your-email@gmail.com` | Email Verified in section 5.6.1 |
| `COGNITO_USER_POOL_ID` | `ap-southeast-1_XXXXXXXXX` | Copied from User Pool ID in 5.3.1 |
| `COGNITO_CLIENT_ID` | `xxxxxxxxxxxxxxxxxxxx` | Copied from Client ID in 5.3.1 |
| `COGNITO_REGION` | `ap-southeast-1` | Fixed |
| `SECURITY_ALERT_TOPIC_ARN` | `arn:aws:sns:...:smart-campus-security` | ARN of Topic created in 5.6.2 |
| `NOTIFICATION_TOPIC_ARN` | `arn:aws:sns:...:smart-campus-notifications` | ARN of Topic created in 5.6.2 |

> [!IMPORTANT]
> The variables `SECURITY_ALERT_TOPIC_ARN`, `NOTIFICATION_TOPIC_ARN` need to be retrieved from **Amazon SNS** (section 5.6.2), `SES_SENDER_EMAIL` from **Amazon SES** (section 5.6.1). You can leave them blank or enter temporary values first, **but remember to return to this section to fully update them after completing all of section 5.6**, before proceeding to testing in section 5.10.

> ![Environment variables](/aws-image/setupLambda/lambda17.png)
*(Click **Save** to save)*.

**Step 5: Grant IAM Role Permissions (Security)**

1. In the **Configuration** tab > **Permissions**, click on the existing Role name (e.g., `smart-campus-api-role-...`).
> ![Open IAM Role](/aws-image/setupLambda/lambda11.png)
2. In the new IAM window, click **Add permissions > Attach policies** to add permissions.
> ![Configure IAM Permissions](/aws-image/setupLambda/lambda12.png)
> ![Configure IAM Permissions](/aws-image/setupLambda/lambdanew.jpg)
#### 5.5.3. Next: Create API Gateway
After Lambda has been deployed and fully authorized, move to the next section **5.5.3 Initialize API Gateway** to create an entry point for requests and connect Lambda to a public URL.

---
At this point, the system's Backend Lambda is ready! Continue to **5.5.3** to attach the API Gateway.
