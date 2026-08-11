---
title : "Create S3 Bucket Hosting"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.8.2. </b> "
---

#### 5.8.2. Hosting Frontend on Amazon S3
Amazon S3 has an extremely powerful feature which is **Static Website Hosting**. You just need to upload HTML, CSS, JS files (after building) to S3, and it will turn into a real web server without you having to maintain an operating system or web server software (like Nginx/Apache).

**Step 1: Create an S3 Bucket for the Website**

1. From the AWS Console search bar, type and select the **S3** service.
> ![Search S3](/aws-image/setupS3frontend/s31.png)
2. Click the **Create bucket** button.
> ![Create bucket button](/aws-image/setupS3frontend/s32.png)
3. **Bucket name**: Give it a meaningful name, e.g., `smart-campus-frontend-2026` (Note: Bucket name must be globally unique across AWS).
> ![Bucket Name](/aws-image/setupS3frontend/s33_1.png)
4. In the **Object Ownership** section, select **ACLs disabled**. In the **Block Public Access settings for this bucket** section, uncheck **Block all public access** and check to acknowledge AWS's warning.
> ![Block Public Access](/aws-image/setupS3frontend/s33_2.png)
5. Scroll down to the bottom and click **Create bucket**.
> ![Successfully created bucket](/aws-image/setupS3frontend/s33_3.png)

**Step 2: Enable Static Website Hosting**

1. Enter the newly created bucket, switch to the **Properties** tab.
> ![Properties Tab](/aws-image/setupS3frontend/s34_1.png)
2. Scroll to the very bottom to the **Static website hosting** section, click **Edit**. Select **Enable**. In the **Index document** field, enter `index.html`.
> ![Configure Index Document](/aws-image/setupS3frontend/s34_2.png)
3. Scroll to the bottom of the page and click **Save changes**.
> ![Save changes](/aws-image/setupS3frontend/s34_3.png)

**Step 3: Configure Policy and Upload source code**

1. Switch to the **Permissions** tab. Scroll to the **Bucket policy** section and click **Edit**.
> ![Permissions Tab](/aws-image/setupS3frontend/s35.png)
2. Enter the JSON Policy configuration to allow public `s3:GetObject` for all objects.
> ![JSON Policy Configuration](/aws-image/setupS3frontend/s36.png)
3. Scroll to the bottom and click **Save changes**.
> ![Save Policy](/aws-image/setupS3frontend/s36_2.png)
4. Switch to the **Objects** tab and click the **Upload** button.
> ![Upload Button](/aws-image/setupS3frontend/s37.png)
5. Drag and drop the files from the build folder (e.g., the Frontend's `dist` folder) into the Upload area.
> ![Drag and drop files](/aws-image/setupS3frontend/s38_1.png)
6. Scroll to the bottom and click the **Upload** button to complete uploading the code to S3.
> ![Upload Complete](/aws-image/setupS3frontend/s38_2.png)

**Step 4: View the result**

Return to the **Properties** tab, scroll down to the **Static website hosting** section at the bottom. Click on the **Bucket website endpoint** link.
> ![Static website endpoint](/aws-image/setupS3frontend/s39.png)

Wait for the browser to load, and you will see the Smart Campus Frontend interface successfully appear! Your application is officially running statically on Amazon S3.
> ![Smart Campus Interface](/aws-image/setupS3frontend/s40.png)
