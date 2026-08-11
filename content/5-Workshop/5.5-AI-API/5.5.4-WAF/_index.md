---
title : "Configure AWS WAF"
date : 2024-01-01
weight : 4
chapter : false
pre : " <b> 5.5.4. </b> "
---

#### 5.5.4. Configure API Protection with AWS WAF and CloudFront



To ensure that the attendance API `/api/attendance/recognize` can only be accessed from the school's internal network (Campus Network), we will use AWS WAF. Because HTTP API type API Gateway does not support direct WAF attachment, we will create a Web ACL (Global), then deploy a CloudFront Distribution in front of API Gateway to attach WAF for protection.

**Step 1: Create an IP Set containing Campus IP addresses**
1. Search for and select the **WAF & Shield** service on the toolbar.
> ![Search WAF](/aws-image/setupWAF/waf1.png)
2. On the left menu, select **IP sets**. In the *Region scope* section, choose **CloudFront (Global)** and click **Create IP address set**.
> ![Manage IP Sets](/aws-image/setupWAF/waf2.png)
3. Fill in the IP Set information:
   - **IP set name**: `SmartCampusIPSet`
   - **Scope**: `CloudFront`
   - **IP version**: `IPv4`
   - **IP addresses**: Enter the Campus Public IP address (e.g., `113.22.28.228/32`), then click **Save**.
> ![Create IP Set](/aws-image/setupWAF/waf3.png)
> ![IP Set Success](/aws-image/setupWAF/waf4.png)

**Step 2: Create a Web ACL with Custom Rule to block external access**
4. Select **Protection packs (web ACLs)** on the left menu and click **Create protection pack (web ACL)**.
> ![Protection Packs](/aws-image/setupWAF/waf5.png)
5. On the configuration page:
   - **App categories**: Select `Other`
   - **App focus**: Select `API`
   - **Select resources to protect**: Click `Skip for now` (we will attach to CloudFront later).
> ![Web ACL Details](/aws-image/setupWAF/waf6.png)
6. Under the *Choose initial protections* section, select **Build your own pack...** and click **Next** -> **Custom rule** -> **Next**.
> ![Choose Custom Rule](/aws-image/setupWAF/waf7.png)
> ![Add Custom Rule](/aws-image/setupWAF/waf8.png)
7. Configure a Custom rule to block attendance access from the outside:
   - **Action**: `Block`
   - **Rule name**: `BlockAttendanceOutsideCompany`
   - **If a request**: `matches all the statement (AND)`
   - **Statement 1**: Choose Inspect `URI path`, Match type `Starts with string`, String to match `/api/attendance/recognize`.
> ![Rule Statement 1](/aws-image/setupWAF/waf9.png)
   - Click **Add another statement**.
   - **Statement 2**: Choose Inspect `Originates from an IP address in`, IP address list `SmartCampusIPSet`. Check **Negate statement results** and choose **Source IP address**. Then click **Add rule**.
> ![Rule Statement 2](/aws-image/setupWAF/waf10.png)
8. Name the Web ACL `SmartCampusAPIWebACL`. Expand the *Customize protection pack* section to enable logging:
   - **Logging destination type**: `Amazon CloudWatch Logs`
   - Click **Create new** to create a new log group.
> ![Web ACL Name](/aws-image/setupWAF/waf11.png)
9. Name the Log group `aws-waf-logs-smartcampus`, set Retention to `Never expire`, Log class to `Standard` and click **Create**.
> ![Create Log Group](/aws-image/setupWAF/waf12.png)
10. Return to the WAF creation page, select the newly created Log group and click **Create protection pack (web ACL)**.
> ![Select Log Group](/aws-image/setupWAF/waf13.png)

**Step 3: Create CloudFront Distribution to protect HTTP API**
11. Search for and access the **CloudFront** service.
> ![Search CloudFront](/aws-image/setupWAF/waf14.png)
12. Select **Distributions** and click **Create distribution**.
> ![Create Distribution](/aws-image/setupWAF/waf15.png)
13. Declare information:
   - **Distribution name**: `smart-campus-api-cf`
   - **Distribution type**: Choose `Single website or app`
   - Click **Next**.
> ![Distribution Options](/aws-image/setupWAF/waf16.png)
14. Configure Origin and Settings:
   - **Origin type**: Select `API Gateway`.
   - **Origin**: Select the Invoke URL of API Gateway.
   - **Origin settings**: Select `Use recommended origin settings`.
> ![Origin Config](/aws-image/setupWAF/waf17.png)
   - **Cache settings**: Select `Use recommended cache settings tailored to serving API Gateway content`.
> ![Cache Config](/aws-image/setupWAF/waf18.png)
   - **Web Application Firewall (WAF)**: Select `Enable security protections`, select `Use existing WAF configuration` and choose the Web ACL `SmartCampusAPIWebACL` you just created.
> ![WAF Config](/aws-image/setupWAF/waf19.png)
   - *(Optional)* **Connectivity**: Under IPv6, choose `Off`.
> ![Connectivity Config](/aws-image/setupWAF/waf22.png)
15. Double-check the information on the **Review and create** page and click **Create distribution**.
> ![Review and Create](/aws-image/setupWAF/waf20.png)
16. Wait for the *Deploying* process to complete. You can use the **Distribution domain name** as the new Endpoint to call the API instead of the direct Invoke URL!
> ![Distribution Success](/aws-image/setupWAF/waf21.png)


> Because HTTP API of API Gateway does not support direct integration with AWS WAF, putting CloudFront as an intermediary buffer layer helps accelerate speed via the Edge Network while also acting as an attachment point for WAF to block unauthorized access.
