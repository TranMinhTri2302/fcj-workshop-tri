---
title: "Blog 2"
date: 2024-01-02
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# PREVENTING FACE RECOGNITION SPOOFING WITH AMAZON REKOGNITION FACE LIVENESS

---

When building face recognition systems, we often run into a classic security vulnerability: how do you stop someone from holding up a pre-captured photo or a pre-recorded video on their phone in front of the camera to trick the system? If you only use Amazon Rekognition's **SearchFacesByImage** API, the AI will simply try to match the face in the image against your database — it has no way of knowing whether that face came from a live person or from a photo of a phone screen. **Amazon Rekognition Face Liveness** solves this problem at its root by verifying that the person in front of the camera is a real, live human being *before* proceeding to identity recognition.

---

## The Real Problem I Ran Into

This article comes from hands-on experience building face recognition systems on AWS. During implementation, I realized that if you only rely on the **SearchFacesByImage** API for face matching, your system is left with a serious security gap.

Specifically, the SearchFacesByImage API does exactly one thing: it takes an input image, detects a face in it, and matches that face against the faces already stored in a **Rekognition Collection**. The API has absolutely no way of knowing where that image actually came from — it could be a live camera capture of a real person, or it could just as easily be a camera pointed at a photo displayed on a phone screen, a printed ID photo, or even a pre-recorded video playing on an iPad.

This means that anyone holding a clear photo of another person's face could hold that photo up to the camera and bypass the system entirely. For use cases like **eKYC** (electronic Know Your Customer), **face-based attendance tracking**, or **facial login authentication**, this is a risk that simply cannot be tolerated.

Some common fraud techniques that a system relying solely on SearchFacesByImage cannot detect:

- Holding up a printed ID card or portrait photo in front of the camera.
- Displaying a face photo on a phone or tablet screen in front of the camera.
- Playing back a pre-recorded video of a real person on another device.
- Using a 3D mask or 3D-rendered face to fool the camera.

All of these methods can bypass standard face matching if the system lacks a **liveness detection** layer — that is, a check to confirm whether the entity interacting with the camera is a real, live person physically present in front of it.

---

## The Solution: Integrating Amazon Rekognition Face Liveness

Fortunately, AWS provides a **Face Liveness** feature within Amazon Rekognition that thoroughly addresses this spoofing problem. Rather than simply matching a static image, Face Liveness uses a live camera stream combined with **visual challenges** (randomized color flashes) to analyze the user's real-time facial response.

Here's how I integrated Face Liveness into the system.

---

## Detailed Processing Flow

### Step 1 — Backend Creates a Face Liveness Session

When the user initiates a face scan, the frontend calls the backend. The backend then calls Amazon Rekognition's **CreateFaceLivenessSession** API to start a new liveness-check session. This API returns a unique **SessionId**, which is sent back to the frontend to begin the verification process.

---

### Step 2 — Frontend Runs the Face Liveness Detector

On the frontend, I used the **@aws-amplify/ui-react-liveness** SDK. This SDK provides a ready-made UI component that handles the entire liveness verification flow.

Once the component is launched with the SessionId, the user sees an **oval-shaped frame** on screen and is prompted to position their face within it. The screen then **flashes randomized color bands** — this is the **Challenge** AWS uses to analyze the light reflection patterns on the user's face.

Throughout this process, the camera **streams video directly to AWS Rekognition**, which analyzes the footage in real time to determine whether the subject is a live person.

---

### Step 3 — Backend Retrieves the Liveness Check Results

Once the frontend verification process completes, the frontend notifies the backend. The backend then calls the **GetFaceLivenessSessionResults** API, passing in the **SessionId** created in Step 1, to retrieve the evaluation results from AWS.

Amazon Rekognition returns a result that includes:

- **Confidence**: a trust score indicating the likelihood that this is a real, live person (ranging from 0% to 100%).
- **ReferenceImage**: the best-quality frame of the user's face selected by AWS during the session.

---

## Evaluating Results Using the Confidence Score

Based on the Face Liveness results, I use the **Confidence** score to decide how to proceed:

- **If Confidence < 90%**: The likelihood of spoofing is high — the user may be using a mask, a 3D-rendered face, an iPad screen, or a pre-recorded video. The system **immediately rejects** the attempt and does not proceed further.

- **If Confidence >= 90%**: The person is confirmed to be real. At this point, AWS also returns the **Reference Image** — the best captured frame of the user's face. I use this image to call **SearchFacesByImage**, which identifies who the person actually is within the Rekognition Collection.

In other words, the correct processing flow is:

**Face Liveness Check → If confirmed real → SearchFacesByImage → Identity Resolution**

For systems with strict security requirements, the liveness step should never be skipped, nor should this order be reversed.

---

## Key Takeaways

- **SearchFacesByImage is only for face recognition** (identity matching) — it has no capability to detect spoofing.

- **Face Liveness protects against spoofing** — meaning it defends against fraud attempts using photos, videos, phone/tablet screens, masks, or other replay-based techniques.

- **CreateFaceLivenessSession** is called from the backend to create a unique liveness-check session for each authentication attempt.

- **SessionId** is used by the frontend to launch the liveness verification process through the Amplify UI component.

- **GetFaceLivenessSessionResults** is called by the backend after the verification process completes, to retrieve the evaluation results.

- **The Confidence Score** is the most critical metric for determining whether a user passes the liveness check. A threshold of 90% is generally appropriate for systems that demand high security.

- **The Reference Image** is the best frame selected by AWS after confirming the user is a real, live person — this image is then used for the identity-matching step.

- **Face Liveness should be placed before the SearchFacesByImage step** in the processing flow — you must verify liveness first, then determine identity.

- **The video stream is only used for liveness analysis** during execution and is automatically discarded afterward, ensuring compliance with data privacy requirements.

---

## Architecture Comparison: Before vs. After Integrating Face Liveness

| Criteria | SearchFacesByImage Only | Face Liveness + SearchFacesByImage Combined |
|---|---|---|
| Identity verification | Yes | Yes |
| Live-person detection | No | Yes |
| Protection against still photos | No | Yes |
| Protection against pre-recorded video | No | Yes |
| Protection against 3D masks | No | Yes |
| User experience | Simple but less secure | Still smooth, significantly more secure |
| Suitable for eKYC / attendance | Not secure enough | Suitable |
| Risk of being bypassed | High | Significantly lower |

Adding Face Liveness shifts the system's underlying logic from "who does this face look like?" to a much safer model: "is this actually a real person, and if so, who are they?"

---

## Advantages of This Architecture

**Robust security**: Completely eliminates fraud attempts using ID photos, phone-displayed images, or pre-recorded videos. The system requires the user to be a genuinely live person physically present in front of the camera.

**Smooth user experience**: Unlike older systems that force users to perform multiple manual actions like "blink three times" or "turn your head left/right," AWS Face Liveness only requires the user to hold still within the oval frame. All the challenge generation and analysis happen almost instantly in the background.

**No video storage**: The video stream is used solely for liveness analysis during execution and is automatically discarded afterward, ensuring compliance with data privacy regulations.

---

## Implementation Notes

**First**, avoid calling sensitive Rekognition APIs directly from the frontend. The backend should be responsible for creating sessions, authenticating users, checking permissions, and making the final decision.

**Second**, configure IAM permissions following the **principle of least privilege**. The backend should only be granted the necessary permissions, such as `rekognition:CreateFaceLivenessSession`, `rekognition:GetFaceLivenessSessionResults`, and `rekognition:SearchFacesByImage`. Avoid granting overly broad permissions that aren't needed.

**Third**, choose an appropriate **Confidence** threshold for your use case. A threshold that's too low increases the risk of spoofing, while one that's too high may cause legitimate users to be rejected under poor lighting or camera conditions. A threshold of 90% is AWS's recommended baseline for systems requiring high security.

**Fourth**, handle frontend error cases carefully — scenarios like users denying camera permissions, obstructed cameras, poor lighting, weak network connections, or users abandoning the flow midway.

**Fifth**, design the UX to clearly guide users: keep your face within the frame, don't wear a mask, avoid overly dark environments, and make sure only one person appears in the camera view at a time.

**Sixth**, exercise caution with biometric data. If you store face images (Reference Images), you need proper security policies, encryption, access controls, and compliance with data privacy regulations applicable to your deployment region.

---

## When Should You Use Amazon Rekognition Face Liveness?

Face Liveness is especially well-suited for systems that need to verify faces while preventing spoofing via photos, videos, or other fraudulent methods. Some real-world use cases include:

- **eKYC** for banks, e-wallets, and fintech platforms — verifying customer identity remotely.
- **Face-based attendance tracking** — ensuring employees are physically present rather than having someone else clock in with a photo.
- **Facial login authentication** — replacing or supplementing password-based login.
- **Access control** for offices, factories, or secure areas.
- **Identity verification** before executing important transactions.
- **Online examination systems** that need to verify test-takers' identities.

If your system currently relies only on SearchFacesByImage, it only answers the question "who does this face look like?" But if you need to answer the question "is this actually a live person in front of the camera?" — it's time to integrate Amazon Rekognition Face Liveness.

---

**Blog Post Link:**
[[Post Link]](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2240573343374292/)

**Main References:**

[Recommendations for Usage of Face Liveness — Amazon Rekognition Developer Guide](https://docs.aws.amazon.com/rekognition/latest/dg/recommendations-liveness.html)

[Amazon Rekognition Face Liveness — Amplify UI React Documentation](https://ui.docs.amplify.aws/react/connected-components/liveness)

**Blog Image:**  
![Blog 2-1](images/3-Blog/Blog-2.png)
![Blog 2-2](images/3-Blog/Blog-2-1.png)