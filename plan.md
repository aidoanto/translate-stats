# Translation Tracking Implementation Plan

## Overview

Track translation usage on lifeline.org.au using GTM and GA4 to answer:
- What pages are being translated?
- At what rate?
- What languages are being translated into?
- Is the translate feature available on each page?
- Are users using our on-site feature or browser extensions?

## What We'll Track

| Data Point | Example Value |
|------------|---------------|
| Event name | `page_translated` |
| Language code | `zh-CN`, `ar`, `vi` |
| Language name | `Mandarin 普通话`, `Arabic (عربى)` |
| Page URL | `/crisis-support/` |
| Page title | `Crisis Support - Lifeline Australia` |
| Translation source | `click` (on-site feature) or `mutation` (browser extension) |
| Feature available | `true` or `false` (tracked via `translate_feature_loaded` event) |

---

## Step 1: Deploy Listener Script (Drupal)

**Location:** Drupal → Content Block (global script block)

**File:** `translation-tracker-listener.html`

The listener script is deployed via Drupal's content block system rather than GTM to keep more logic in Drupal.

**What it does:**
- On page load: Fires `translate_feature_loaded` event with `translate_feature_available` boolean
- On translation: Fires `page_translated` event with language info and `translation_source`

---

## Step 2: Create Data Layer Variables

**Location:** GTM → Variables → User-Defined Variables → New

Create these 6 variables:

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

### Variable 5
- **Name:** `DLV - translation_source`
- **Type:** Data Layer Variable
- **Data Layer Variable Name:** `translation_source`

### Variable 6
- **Name:** `DLV - translate_feature_available`
- **Type:** Data Layer Variable
- **Data Layer Variable Name:** `translate_feature_available`

---

## Step 3: Create Custom Event Triggers

**Location:** GTM → Triggers → New

### Trigger 1 (Translation Event)
- **Name:** `CE - page_translated`
- **Trigger Type:** Custom Event
- **Event name:** `page_translated`
- **This trigger fires on:** All Custom Events

### Trigger 2 (Feature Availability Event)
- **Name:** `CE - translate_feature_loaded`
- **Trigger Type:** Custom Event
- **Event name:** `translate_feature_loaded`
- **This trigger fires on:** All Custom Events

---

## Step 4: Create GA4 Event Tags

**Location:** GTM → Tags → New → Google Analytics: GA4 Event

### Tag 1: Page Translated Event

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
| `translation_source` | `{{DLV - translation_source}}` |

**Triggering:** `CE - page_translated`

### Tag 2: Feature Availability Event

- **Name:** `GA4 - Translate Feature Loaded`
- **Configuration Tag:** (select your existing GA4 configuration tag)
- **Event Name:** `translate_feature_loaded`

**Event Parameters:**

| Parameter Name | Value |
|---------------|-------|
| `feature_available` | `{{DLV - translate_feature_available}}` |

**Triggering:** `CE - translate_feature_loaded`

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

### Dimension 4
- **Dimension name:** Translation Source
- **Scope:** Event
- **Event parameter:** `translation_source`
- **Description:** `click` = on-site feature, `mutation` = browser extension

### Dimension 5
- **Dimension name:** Translate Feature Available
- **Scope:** Event
- **Event parameter:** `feature_available`
- **Description:** Whether the translate dropdown was present on the page

---

## Step 6: Test with GTM Preview

1. In GTM, click **Preview** button (top right)
2. Enter `https://www.lifeline.org.au` and click Connect
3. Verify the `translate_feature_loaded` event fires immediately on page load
4. Click the **Translate** dropdown and select a language (e.g., Mandarin)
5. In the GTM debug panel (Tag Assistant), verify:
   - `translate_feature_loaded` event appears (on page load)
   - `page_translated` event appears (after selecting language)
   - Both GA4 tags show as "Fired"

**Expected data layer output (on page load):**
```json
{
  "event": "translate_feature_loaded",
  "translate_feature_available": true
}
```

**Expected data layer output (after translation - on-site feature):**
```json
{
  "event": "page_translated",
  "translation_language_code": "zh-CN",
  "translation_language_name": "Mandarin 普通话",
  "translated_page_url": "/",
  "translated_page_title": "Lifeline Australia - 13 11 14 - Crisis Support. Suicide Prevention.",
  "translation_source": "click"
}
```

**Expected data layer output (browser extension translation):**
```json
{
  "event": "page_translated",
  "translation_language_code": "es",
  "translation_language_name": "es",
  "translated_page_url": "/",
  "translated_page_title": "...",
  "translation_source": "mutation"
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
- **On-site vs browser extension split** (translations by `translation_source`)
- **Feature availability rate** (pages where feature is on vs off)

### Dimensions to Break Down By
- `language_code` / `language_name` - Which languages are most popular?
- `page_url` - Which pages are translated most?
- `translation_source` - On-site feature (`click`) vs browser extension (`mutation`)
- `feature_available` - Was the translate dropdown present?
- `date` - Trend over time

### Suggested Visualisations
1. **Pie chart:** Translations by language
2. **Pie chart:** Translations by source (on-site vs browser extension)
3. **Bar chart:** Top 10 translated pages
4. **Line chart:** Translations per day/week
5. **Scorecard:** Feature availability rate (% of page views with feature available)
6. **Table:** Full breakdown with all dimensions

---

## Checklist

- [ ] Deploy listener script to Drupal content block
- [ ] Create 6 Data Layer Variables in GTM
- [ ] Create 2 Custom Event triggers in GTM
- [ ] Create 2 GA4 Event tags in GTM
- [ ] Register 5 custom dimensions in GA4
- [ ] Test with GTM Preview
- [ ] Publish GTM container
- [ ] Verify data in GA4 Realtime (after publishing)
- [ ] Verify data in GA4 Events (24-48 hours later)
- [ ] Build Looker dashboard with source segmentation
