---
title : "Create API Gateway"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.5.3. </b> "
---

#### 5.5.3. Initialize and configure API Gateway
API Gateway is the sole entry point receiving all requests from the Frontend and routing them down to the Lambda function created in the previous step. Since Lambda is ready, we can completely configure the integration right in this step.

1. Search for and select the **API Gateway** service on the AWS Console search bar.
> ![Search API Gateway](/aws-image/setupAPI/api1.png)
2. On the main interface of API Gateway, scroll down and select **Create an API**.
> ![Create API](/aws-image/setupAPI/api2.png)
3. Look for the **HTTP API** section (AWS's modern, low-cost API type), click the **Build** button.
> ![Select HTTP API](/aws-image/setupAPI/api3.png)
4. At the *Configure API* step, set up the parameters and connect with Lambda:
   - **API name**: Name the API (e.g., `SmartCampusHTTPApi`).
   - Click the **Add integration** button and set up:
     - **Integrations**: Select `Lambda`.
     - **AWS Region**: `ap-southeast-1`.
     - **Lambda function**: Select the `smart-campus-api` function just created in section 5.5.2.
   - Click **Next**.
> ![Configure API](/aws-image/setupAPI/api4.png)
5. At the *Configure routes* step, set up a Route to redirect all Requests down to Lambda (Serverless Proxy Model):
   - **Method**: Select `ANY`.
   - **Resource path**: Enter `/{proxy+}`.
   - **Integration target**: Select the `smart-campus-api` Lambda function.
   - Click **Next**.
> ![Configure Route](/aws-image/setupAPI/api5.png)
6. At the *Define stages* step, the system defaults to creating a `$default` Stage with Auto-deploy. Leave as is and click **Next**.
> ![Configure Stages](/aws-image/setupAPI/api6.png)
7. At the *Review and create* step, double-check the information and then click **Create**.
> ![Review and create](/aws-image/setupAPI/api7.png)
8. After successful creation, the system generates an **Invoke URL**. **Please copy and save this URL** — it is needed immediately in the next step (5.5.4 WAF).
> ![Copy Invoke URL](/aws-image/setupAPI/api8.png)
9. Verify by opening a new tab, pasting the Invoke URL, and appending `/docs` at the end. If the **Swagger UI** interface appears, API Gateway has successfully connected to Lambda!
> ![Check Swagger UI](/aws-image/setupAPI/api9.png)

**Checklist:** Save the **Invoke URL**. The next step (5.5.4 WAF) will use this URL to configure CloudFront to protect the API.
