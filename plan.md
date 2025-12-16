# Translation Tracking Implementation Plan

## Overview

Track translation usage on lifeline.org.au using GTM and GA4 to answer:
- What pages are being translated?
- At what rate?
- What languages are being translated into?

## What We'll Track

| Data Point | Example Value |
|------------|---------------|
| Event name | `page_translated` |
| Language code | `zh-CN`, `ar`, `vi` |
| Language name | `Mandarin 普通话`, `Arabic (عربى)` |
| Page URL | `/crisis-support/` |
| Page title | `Crisis Support - Lifeline Australia` |

---

## Step 1: Create Custom HTML Tag (Listener)

**Location:** GTM → Tags → New → Custom HTML

**Tag Name:** `translation-tracker-listener.html`


**Triggering:** All Pages

---

## Step 2: Create Data Layer Variables

**Location:** GTM → Variables → User-Defined Variables → New

Create these 4 variables:

### Variable 1
- **Name:** `DLV - translation_language_code`
- **Type:** Data Layer Variable
- **Data Layer Variable Name:** `translation_language_code`

### Variable 2
- **Name:** `DLV - translation_language_name`
- **Type:** Data Layer Variable
- **Data Layer Variable Name:** `translation_language_name`

### Variable 3
- **Name:** `DLV - translated_page_url`
- **Type:** Data Layer Variable
- **Data Layer Variable Name:** `translated_page_url`

### Variable 4
- **Name:** `DLV - translated_page_title`
- **Type:** Data Layer Variable
- **Data Layer Variable Name:** `translated_page_title`

---

## Step 3: Create Custom Event Trigger

**Location:** GTM → Triggers → New

- **Name:** `CE - page_translated`
- **Trigger Type:** Custom Event
- **Event name:** `page_translated`
- **This trigger fires on:** All Custom Events

---

## Step 4: Create GA4 Event Tag

**Location:** GTM → Tags → New → Google Analytics: GA4 Event

- **Name:** `GA4 - Page Translated`
- **Configuration Tag:** (select your existing GA4 configuration tag)
- **Event Name:** `page_translated`

**Event Parameters:**

| Parameter Name | Value |
|---------------|-------|
| `language_code` | `{{DLV - translation_language_code}}` |
| `language_name` | `{{DLV - translation_language_name}}` |
| `page_url` | `{{DLV - translated_page_url}}` |
| `page_title` | `{{DLV - translated_page_title}}` |

**Triggering:** `CE - page_translated`

---

## Step 5: Register Custom Dimensions in GA4

**Location:** GA4 → Admin → Custom definitions → Custom dimensions → Create custom dimension

### Dimension 1
- **Dimension name:** Translation Language Code
- **Scope:** Event
- **Event parameter:** `language_code`

### Dimension 2
- **Dimension name:** Translation Language Name
- **Scope:** Event
- **Event parameter:** `language_name`

### Dimension 3
- **Dimension name:** Translated Page URL
- **Scope:** Event
- **Event parameter:** `page_url`

---

## Step 6: Test with GTM Preview

1. In GTM, click **Preview** button (top right)
2. Enter `https://www.lifeline.org.au` and click Connect
3. On the site, click the **Translate** dropdown
4. Select a language (e.g., Mandarin)
5. In the GTM debug panel (Tag Assistant), verify:
   - `page_translated` event appears in the left sidebar
   - Click on it to see the data layer values
   - The `GA4 - Page Translated` tag shows as "Fired"

**Expected data layer output:**
```json
{
  "event": "page_translated",
  "translation_language_code": "zh-CN",
  "translation_language_name": "Mandarin 普通话",
  "translated_page_url": "/",
  "translated_page_title": "Lifeline Australia - 13 11 14 - Crisis Support. Suicide Prevention."
}
```

---

## Step 7: Publish

Once testing confirms everything works:

1. In GTM, click **Submit** (top right)
2. Add a version name: `Add translation tracking`
3. Click **Publish**

---

## Step 8: Verify in GA4 (24-48 hours later)

**Location:** GA4 → Reports → Realtime

1. Trigger a translation on the live site
2. Check Realtime report for `page_translated` event
3. Click into the event to see parameter values

**Location:** GA4 → Reports → Engagement → Events

After 24-48 hours, you'll see `page_translated` in your events list with full data.

---

## Reporting in Looker

Once data is flowing, create a Looker dashboard with:

### Key Metrics
- **Total translations** (count of `page_translated` events)
- **Translation rate** (`page_translated` / `page_view` as percentage)
- **Unique users translating** (users with at least one translation event)

### Dimensions to Break Down By
- `language_code` / `language_name` - Which languages are most popular?
- `page_url` - Which pages are translated most?
- `date` - Trend over time

### Suggested Visualisations
1. **Pie chart:** Translations by language
2. **Bar chart:** Top 10 translated pages
3. **Line chart:** Translations per day/week
4. **Table:** Full breakdown with all dimensions

---

## Checklist

- [ ] Create Custom HTML tag with listener code
- [ ] Create 4 Data Layer Variables
- [ ] Create Custom Event trigger
- [ ] Create GA4 Event tag
- [ ] Register 3 custom dimensions in GA4
- [ ] Test with GTM Preview
- [ ] Publish GTM container
- [ ] Verify data in GA4 Realtime (after publishing)
- [ ] Verify data in GA4 Events (24-48 hours later)
- [ ] Build Looker dashboard
