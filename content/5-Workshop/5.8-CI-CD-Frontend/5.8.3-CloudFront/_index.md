---
title : "Accelerate with CloudFront"
date : 2024-01-01
weight : 3
chapter : false
pre : " <b> 5.8.3. </b> "
---

#### 5.8.3. Initialize Amazon CloudFront (CDN)
Although S3 can be used to host a website, it does not support SSL certificates (HTTPS) for custom domain names, nor does it have a global cache. Amazon CloudFront solves all these issues.

**Step 1: Create a CloudFront Distribution**

1. From the AWS Console search bar, type and select the **CloudFront** service.
> ![Search CloudFront](/aws-image/setupCloudfront/cloudfront1.png)
2. Click the **Create distribution** button.
> ![Create Distribution](/aws-image/setupCloudfront/cloudfront2.png)
3. At the **Get started** step: In the **Distribution name** section, name the Distribution `smart-campus-frontend`. Scroll to the bottom and click **Next**.
> ![Distribution Name](/aws-image/setupCloudfront/cloudfront3.png)
4. At the **Specify origin** step: In the **Origin domain** section, click **Browse S3**.
> ![Select S3 Origin](/aws-image/setupCloudfront/cloudfront4.png)
5. A popup appears, select the S3 Bucket you just created (e.g., `smart-campus-frontend-2026`) and click **Choose**. Then return to the main screen, scroll to the bottom of the page, and click **Next**.
> ![Select S3 Location](/aws-image/setupCloudfront/cloudfront5.png)
6. At the **Enable security** step: In the **Web Application Firewall (WAF)** section, select **Do not enable security protections**. Then click **Next**.
> ![Disable WAF](/aws-image/setupCloudfront/cloudfront6.png)
7. At the **Review and create** step: Scroll to the bottom and click the **Create distribution** button.
> ![Finish creating Distribution](/aws-image/setupCloudfront/cloudfront7.png)

**Step 2: Configure Error Pages (For React/Vue SPA applications)**

Since React/Vue are Single Page Applications (SPAs), all requests to non-existent paths need to be redirected to `index.html` so the Frontend can handle the Routing itself.

1. Immediately after successfully creating the Distribution, switch to the **Error pages** tab and click the **Create custom error response** button.
> ![Create Error Response](/aws-image/setupCloudfront/cloudfront8.png)
2. Configure the 404 error redirection as follows:
   - **HTTP error code**: `404: Not Found`
   - **Customize error response**: Select `Yes`
   - **Response page path**: `/index.html`
   - **HTTP Response code**: `200: OK`
Finally, click **Create custom error response** to save.
> ![Configure 404](/aws-image/setupCloudfront/cloudfront9.png)

**Step 3: Access the website via CDN**

1. Switch to the **General** tab of the CloudFront Distribution, find and copy the **Distribution domain name** address (e.g., `d...cloudfront.net`).
> ![CloudFront Domain](/aws-image/setupCloudfront/cloudfront10.png)
2. Open your browser and access that link. You will see the website load extremely fast and protected with a secure HTTPS padlock from Amazon CloudFront!
> ![HTTPS Web Interface](/aws-image/setupCloudfront/cloudfront11.png)
