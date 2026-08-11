---
title : "Configure Amazon Rekognition"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.5.1. </b> "
---

#### 5.5.1. Create a Collection on Amazon Rekognition
Amazon Rekognition does not keep the original images for recognition; rather, it extracts and stores the metadata/vectors of facial features in a repository called a **Collection**.

Currently, the AWS Management Console **does not support** a user interface (UI) for creating Collections. However, we don't need complex local computer installations. Instead, we will use **AWS CloudShell** - a command-line environment available right in the AWS browser.

**Step 1: Open AWS CloudShell**
1. In the AWS Console search bar, type **Rekognition** and access the service.
2. Look at the top right corner of the screen (next to the notification bell icon), click the **CloudShell** `>_` icon to open the command-line interface.
> ![Open CloudShell](/aws-image/setupRekognition/regco-1.png)
*(Initializing CloudShell may take a few dozen seconds)*.

**Step 2: Create a Rekognition Collection**
3. When the command prompt `$` appears, copy and paste the following command (remember to change the region if you use a different one):
   ```bash
   aws rekognition create-collection --collection-id smart-campus-faces --region ap-southeast-1
   ```
4. If successful, you will receive a JSON result returned with `StatusCode: 200`:
> ![Create Collection Success](/aws-image/setupRekognition/regco-2.png)

**Step 3: Verify the Collection list**
5. You can run the following command to list the existing Collections in your account to ensure it was created correctly:
   ```bash
   aws rekognition list-collections --region ap-southeast-1
   ```
6. The result will display `smart-campus-faces` in the `CollectionIds` array:
> ![Check Collection](/aws-image/setupRekognition/regco-3.png)

That's it! The virtual facial data repository is ready. Later, when an employee registers in the system via API, Lambda will receive the image from S3 and call the `IndexFaces` function to push their facial vectors into this Collection.
