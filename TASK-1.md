# Task 01 - GTM Event Schema & Funnel Tracking Spec

This document details the Google Tag Manager (GTM) event tracking schema and funnel drop-off configuration designed for the OrthoNow website. 

---

## 1. GTM Event Tracking Schema

| Event Name | GTM Trigger Type | Key Parameters (Min. 3) | GA4 Report or Audience Destination |
| :--- | :--- | :--- | :--- |
| `booking_step_complete` | Custom Event | 1. `step_number` (Integer)<br>2. `step_name` (String)<br>3. `clinic_location` (String)<br>4. `specialty` (String)<br>5. `booking_id` (String - Step 3 only) | **Reports**: Funnel Exploration, Conversions<br>**Audiences**: High Intent Drop-offs, Booking Converters |
| `call_initiated` | Click - Just Links (Regex: `tel:.*`) | 1. `click_location` (header/footer/clinic_card)<br>2. `phone_number` (clicked number)<br>3. `page_path` (source page) | **Reports**: Events, Engagement (Conversions)<br>**Audiences**: Call-in Leads |
| `whatsapp_initiated` | Click - Just Links (Regex: `.*wa\.me.*\|.*api\.whatsapp\.com.*`) | 1. `click_location` (widget/footer)<br>2. `whatsapp_number` (destination)<br>3. `page_path` (source page) | **Reports**: Events, Engagement (Conversions)<br>**Audiences**: WhatsApp Inquirers |
| `patient_guide_form_submitted` | Custom Event | 1. `form_id` (String)<br>2. `guide_name` (String)<br>3. `page_path` (source page) | **Reports**: Events, Conversions<br>**Audiences**: Informational Leads |
| `patient_guide_downloaded` | Click - Just Links (Regex: `.*\.pdf$`) | 1. `file_name` (String)<br>2. `file_extension` (pdf)<br>3. `page_path` (source page) | **Reports**: Engagement -> Events (File Downloads)<br>**Audiences**: Highly Engaged Leads |
| `clinic_location_viewed` | Page View (Page Path starts with `/clinics/`) | 1. `clinic_name` (clinic location identifier)<br>2. `clinic_city` (Bengaluru/Hyderabad/Chennai)<br>3. `page_referrer` (traffic source path) | **Reports**: Engagement -> Pages and Screens<br>**Audiences**: Location-Specific Retargeting (e.g. "Bengaluru Clinic Leads") |
| `blog_scroll_depth` | Scroll Depth (Thresholds: 25, 50, 75, 90) | 1. `scroll_depth_threshold` (Integer)<br>2. `blog_title` (String)<br>3. `blog_category` (String) | **Reports**: Engagement -> Events<br>**Audiences**: Content Consumers (Audience: Users with depth >= 75%) |

---

## 2. Multi-step Booking Form Tracking & Funnel Analysis

To analyze user friction and drop-offs in the 3-step appointment booking form, we implement step-level tracking via custom front-end JavaScript calls to push details to the `window.dataLayer`.

### DataLayer Push JSON Payloads

#### Step 1: clinic location + specialty selected (User clicks "Next" on Step 1)
```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Replacement"
}


#### Step 2: enter name/phone/preferred date (User clicks "Next" on Step 2)
```json


{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "contact_details_entered",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Replacement",
  "preferred_date": "2026-07-15"
}

#### Step 3: confirm booking (User views final confirmation state/success message)

{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Indiranagar",
  "specialty": "Knee Replacement",
  "preferred_date": "2026-07-15",
  "booking_id": "BK-90827"
}

