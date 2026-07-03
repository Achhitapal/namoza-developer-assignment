# OrthoNow - Developer Assignment (Web Dev & Martech)

This repository contains the deliverables for the OrthoNow Digital Growth and Strategy Assignment by Namoza. 

## Project Structure

This project consists of the following components:

1.  **TASK_1_GTM_SCHEMA.md**: A complete GTM event tracking schema for OrthoNow. It outlines custom events, trigger rules, parameters, and GA4 configurations, along with a detailed multi-step booking funnel setup and standard GA4 Funnel Exploration setup. It also answers key questions regarding dataLayer ownership and Google Ads conversion choice.
2.  **index.html**: The rebuilt landing page for the OrthoNow "Book a Consultation" campaign. Written in clean semantic HTML, custom responsive CSS (embedded), and vanilla JS. It is zero-dependency, self-contained, optimized for 90+ Core Web Vitals mobile score, features an animated inline thank-you state, and fires the required `consultation_form_submitted` dataLayer push.
3.  **TASK_3_INTEGRATION.md**: A detailed technical architecture documentation for integrating the landing page with HubSpot CRM and Karix WhatsApp Business API. It details how to solve the HubSpot phone number deduplication challenge, Karix SLA targets, monitoring alerts, and Google Ads conversions.

---

## How to Test and Run the Landing Page

1.  **Open in Browser**:
    Simply double-click the `index.html` file or open it in your browser of choice. Since it contains zero external dependencies (no script imports, frameworks, or stylesheet links), it executes cleanly offline.
2.  **Verify the GTM dataLayer Push**:
    - Open the browser Developer Tools console (`F12` or `Cmd + Option + I`).
    - Fill in the **Name** and **Phone Number** in the form and click **Book Consultation**.
    - Watch the console output! A custom visual logger prints the exact `window.dataLayer` push parameters in a structured layout.
    - Inspect the `window.dataLayer` array directly in your console to verify the push object:
      ```javascript
      console.log(window.dataLayer);
      ```
3.  **Verify Form Transition**:
    - Once submitted, the form container smoothly transitions into the checkmarked **Thank You!** success panel without page reload or layout shift.


## Submission Links

- **GitHub Repository:** https://github.com/Achhitapal/namoza-developer-assignment
- **Live Website:** https://achhitapal.github.io/namoza-developer-assignment/
- **Loom Walkthrough:** https://www.loom.com/share/8d3a9f1839e34b06add034f0e6d5c5d1    