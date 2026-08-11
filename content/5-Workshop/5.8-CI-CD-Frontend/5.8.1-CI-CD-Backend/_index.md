---
title : "Backend CI/CD Automation"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.8.1. </b> "
---

#### 5.8.1. Set up CI/CD flow for Backend with AWS CodePipeline
The deployment of Lambda functions (Backend) every time there is new code also needs to be automated. Because the Backend architecture uses SAM (Serverless Application Model), the CI/CD process will be a bit different from the Frontend: We will use CodeBuild to directly run `sam build` and `sam deploy` commands to AWS, and **skip the Deploy step** of the Pipeline.

**Prerequisites:** The Backend code is on a GitHub repository and the root directory of the project **already has a `buildspec-backend.yml` file** with the content below. If not, create this file before proceeding:

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.12
    commands:
      - pip install --upgrade pip
      - pip install aws-lambda-powertools boto3

  build:
    commands:
      # Package the dist folder
      - cd backend
      - pip install -r requirements.txt -t ./dist
      - cp -r app ./dist/
      # Create zip file to upload to Lambda
      - cd dist && zip -r ../lambda_function.zip . && cd ..
      # Update Lambda function directly using AWS CLI
      - aws lambda update-function-code
          --function-name smart-campus-api
          --zip-file fileb://lambda_function.zip
          --region ap-southeast-1

artifacts:
  files:
    - backend/lambda_function.zip
```



**Step 1: Initialize CodePipeline**
1. Search for and access the **CodePipeline** service on the AWS Console.
> ![Search CodePipeline](/aws-image/setupCodePipeline/pipeline1.png)
2. Click **Create pipeline**. 
> ![Create Pipeline](/aws-image/setupCodePipeline/pipeline2.png)
3. Select **Build custom pipeline** then click **Next**.
> ![Select Build Custom](/aws-image/setupCodePipeline/pipeline3.png)
4. **Pipeline name**: Enter a name, e.g., `smart-campus-backend-pipeline`. Select **New service role** and click **Next**.
> ![Pipeline Name and Role](/aws-image/setupCodePipeline/pipeline4.png)

**Step 2: Configure Source (Code source)**
1. **Source provider**: Select **GitHub (via GitHub App)**. Click the **Connect to GitHub** button.
> ![Connect GitHub](/aws-image/setupCodePipeline/pipeline5.png)
2. Enter a connection name (e.g., `github-smart-campus`) and click **Connect to GitHub**.
> ![Connection name](/aws-image/setupCodePipeline/pipeline6.png)
3. The system will open a pop-up for authorization, click **Authorize**.
> ![Authorize](/aws-image/setupCodePipeline/pipeline7.png)
4. Select the correct Repository containing your Backend code and click **Install & Authorize**.
> ![Select Repo](/aws-image/setupCodePipeline/pipeline8.png)
5. The CodePipeline screen will display App Installation, click **Connect**.
> ![Finish connection](/aws-image/setupCodePipeline/pipeline9.png)
6. Return to the Source configuration page, ensure the Repository, branch `main`, **CodePipeline default**, and **Webhook** are configured, then click **Next**.
> ![Confirm Repo and Branch](/aws-image/setupCodePipeline/pipeline10.png)

**Step 3: Configure Build (CodeBuild)**
1. **Build provider**: Select **AWS CodeBuild**. Choose Region `ap-southeast-1`. Click the **Create project** button.
> ![Select CodeBuild](/aws-image/setupCodePipeline/pipeline11.png)
2. In the new window, name the Project `smart-campus-backend-build`.
> ![CodeBuild Name](/aws-image/setupCodePipeline/pipeline12.png)
3. **Environment**: Select **Amazon Linux**, Runtime **Standard**, Image `aws/codebuild/amazonlinux-x86_64-standard:6.0` (or latest version). Select **New service role**.
> ![Configure Environment](/aws-image/setupCodePipeline/pipeline13.png)
4. Scroll down to the **Buildspec** section, select **Use a buildspec file** and enter the file name as `buildspec-backend.yml`.
> ![Buildspec file](/aws-image/setupCodePipeline/pipeline14.png)
5. Scroll down to the Logs section, skip or keep defaults then click **Continue to CodePipeline**.
> ![Continue](/aws-image/setupCodePipeline/pipeline15.png)
6. Once there is a green notification that the project was successfully created, scroll down and click **Next**.
> ![Confirm Build Stage](/aws-image/setupCodePipeline/pipeline16.png)

**Step 4: Skip Test and Deploy steps**
Because the `sam deploy` command inside CodeBuild has automatically deployed the Backend infrastructure (CloudFormation, Lambda, API Gateway) to AWS already, we no longer need the Deploy step of CodePipeline.
1. At the **Add test stage** step, click **Skip test stage**.
> ![Skip Test](/aws-image/setupCodePipeline/pipeline17.png)
2. At the **Add deploy stage** step, click **Skip deploy stage**.
> ![Skip Deploy](/aws-image/setupCodePipeline/pipeline18.png)
3. The Review screen appears, scroll to the bottom and click **Create pipeline**.
> ![Review Stage](/aws-image/setupCodePipeline/pipeline19.png)

**Step 5: Grant additional permissions for CodeBuild's Role**
At this point, the Pipeline will start running but the Build step will show a red error because CodeBuild's Role does not have permissions to create Lambda, S3, IAM, CloudFormation... You need to assign additional permissions to it.
1. From the search bar, type **IAM** and select the service.
> ![Search IAM](/aws-image/setupCodePipeline/pipeline20.png)
2. Select the **Roles** section on the left, find the Role whose name contains `codebuild-smart-campus-backend-build`, and click on that Role name.
> ![Find CodeBuild Role](/aws-image/setupCodePipeline/pipeline21.png)
3. Click the **Add permissions** > **Attach policies** button.
> ![Attach policies](/aws-image/setupCodePipeline/pipeline22.png)
4. Search and check Policies like: **AWSLambda_FullAccess**, **AmazonS3FullAccess**... (and other permissions depending on application requirements). Then click **Add permissions**.
> ![AWSLambda_FullAccess](/aws-image/setupCodePipeline/pipeline23.png)
> ![AmazonS3FullAccess](/aws-image/setupCodePipeline/pipeline24.png)

**Complete**
Return to the CodePipeline page, click the **Release change** (or Retry) button to run again. If all steps show green, your Backend has successfully gone to the cloud automatically!
> ![Complete](/aws-image/setupCodePipeline/pipelinenew.png)
