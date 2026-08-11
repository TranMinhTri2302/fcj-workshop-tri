---
title : "Frontend CI/CD Automation"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.8.4. </b> "
---

#### 5.8.4. Set up CI/CD flow for Frontend with AWS CodePipeline
Instead of manually running a build command and dragging and dropping files to S3 every time you update the interface, we will have AWS do this completely automatically whenever there is new code pushed to GitHub.

**Prerequisites:** You have pushed the Frontend source code folder to a repository on GitHub and the root directory contains the `buildspec.yml` configuration file for the frontend.

**Step 1: Initialize CodePipeline**
1. Search for and access the **CodePipeline** service on the AWS Console.
2. Click **Create pipeline**.
3. **Pipeline name**: Enter a name, e.g., `smart-campus-frontend-pipeline`.
4. **Service role**: Select **New service role** (so AWS automatically creates permissions for the Pipeline). Click **Next**.
> ![Pipeline Settings](/aws-image/setupCodePipeline/pipeline26.png)

**Step 2: Configure Source (Code source)**
1. **Source provider**: Select **GitHub (via GitHub App)**. Click the **Connect to GitHub** button.
> ![Connect GitHub](/aws-image/setupCodePipeline/pipeline5.png)
2. Enter a connection name (e.g., `github-smart-campus`) and follow the on-screen instructions to Authorize AWS to access your GitHub account.
> ![Connection name](/aws-image/setupCodePipeline/pipeline6.png)
> ![Authorize](/aws-image/setupCodePipeline/pipeline7.png)
3. Select the Repository containing your Frontend code and click **Install & Authorize**. Then click **Connect**.
> ![Select Repo](/aws-image/setupCodePipeline/pipeline8.png)
> ![Finish connection](/aws-image/setupCodePipeline/pipeline9.png)
4. Return to the Source configuration page, ensure the Repository, Branch (e.g., `main`), and **CodePipeline default** are selected. Click **Next**.
> ![Confirm Repo and Branch](/aws-image/setupCodePipeline/pipeline11.png)

**Step 3: Configure Build (Compiler)**
1. **Build provider**: Select **AWS CodeBuild**. Choose Region `ap-southeast-1`.
2. Click the **Create project** button, a new window will appear. Name the project (e.g., `smart-campus-frontend-build`).
> ![Build project name](/aws-image/setupCodePipeline/pipeline27.png)
3. **Environment**: Select the **Amazon Linux** operating system, Runtime **Standard**, the latest Image version (`aws/codebuild/amazonlinux2-x86_64-standard...`).
> ![Configure Environment](/aws-image/setupCodePipeline/pipeline28.png)
4. Scroll down to the Buildspec section, select **Use a buildspec file** (pointing to the frontend's `buildspec-frontend.yml` file).
> ![Buildspec file](/aws-image/setupCodePipeline/pipeline29.png)
5. Scroll down to the bottom and click **Continue to CodePipeline**.
> ![Continue CodePipeline](/aws-image/setupCodePipeline/pipeline30.png)
6. Return to the CodePipeline window, click **Next**.
> ![Confirm Build step](/aws-image/setupCodePipeline/pipeline31.png)

**Step 4: Configure Deploy (Deployment)**
1. **Deploy provider**: Select **Amazon S3**.
2. **Region**: Same as the S3 Bucket's Region.
3. **Bucket**: Select the static S3 Bucket name you created in lesson 5.8.2 (e.g., `smart-campus-frontend-2026`). 
4. Check the **Extract file before deploy** box (Very important, to unpack the ZIP file containing the build code).
5. Click **Next**, review the configuration, and then click **Create pipeline**.
> ![Deploy to S3](/aws-image/setupCodePipeline/pipeline32.png)

**Step 5: Practical Experience**
Every time you change the interface, edit text, change colors, then commit and `git push` to the `main` branch. You go back to CodePipeline and will see the Pipeline automatically run from Source -> Build -> Deploy.
When complete, reload the webpage and the new interface is live!
> ![Finish Frontend deployment](/aws-image/setupCodePipeline/pipeline33.png)
