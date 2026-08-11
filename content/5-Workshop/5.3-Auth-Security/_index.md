---
title : "Authentication & Security"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

### Goal

In this section, we will build the first line of defense for user authentication for the entire Smart Campus system. Instead of manually coding password encryption logic and generating complex Tokens, we will fully delegate this to **Amazon Cognito**.

> [!NOTE]
> **AWS WAF** (protecting the API from unauthorized external access) will be configured in **section 5.5.4** — after the API Gateway and Lambda have been created, because WAF requires the API Gateway Invoke URL to operate.

### Detailed Practice Content

#### 5.3.1. Initialize Amazon Cognito User Pool
1. Go to the AWS Console, and search for the **Cognito** service.
> ![Search Cognito service](/aws-image/setupCognito/cognito1.png)
2. In the Amazon Cognito interface, select **Create user pool**.
> ![Initialize Cognito](/aws-image/setupCognito/cognito2.png)
3. On the setup screen, under *Application type*, choose **Single-page application (SPA)**. Enter your application name (e.g., `smart-campus-client`).
> ![Setup SPA application](/aws-image/setupCognito/cognito3.png)
4. Scroll down to the *Configure options* section, and check **Email** as the primary sign-in method. Then click the **Create user directory** button.
> ![Configure Email](/aws-image/setupCognito/cognito4.png)
5. The system will display a green notification indicating that the User Pool and App Client have been successfully created.
> ![Creation successful](/aws-image/setupCognito/cognito5.png)
6. On the information page (User pool information), copy and save the **User pool ID**. This code will be added to the `.env` file of the ReactJS source code.
> ![Copy User Pool ID](/aws-image/setupCognito/cognito6.png)
7. On the left menu, switch to the **App clients** tab under *Applications* and click on the App client you just created.
> ![Select App Client](/aws-image/setupCognito/cognito7.png)
8. On the App client details page, you can copy the **Client ID**. However, we need to configure a little more.
> ![View Client ID](/aws-image/setupCognito/cognito8.png)
9. Click the **Edit** button in the top right corner of the *App client information* section to edit authentication permissions.
> ![Edit App Client](/aws-image/setupCognito/cognito9.png)
10. Under *Authentication flows*, tick **ALLOW_USER_PASSWORD_AUTH**. This step is extremely important for the Frontend to call the traditional Username and Password login API.
> ![Configure Auth Flows](/aws-image/setupCognito/cognito10.png)
11. Scroll to the bottom of the page and click the **Save changes** button to save.
> ![Save changes](/aws-image/setupCognito/cognito11.png)
12. After saving successfully, you can copy the **Client ID** again to make sure the configuration is complete.
> ![Configuration complete](/aws-image/setupCognito/cognito12.png)
