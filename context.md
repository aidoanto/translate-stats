# Translation Tracking Project - Context

## The Request

From the boss (via message):

> "Hey Aidan, got something I'd love your big brain on if you have some capacity. Graham (CEO) is very interested in data around the 'translate' feature usage on our new Drupal site. Wondering if you could work out the best way to understand:
>
> - what pages are being 'translated', at what rate
> - what languages are being translated into
>
> I'm guessing this would be a custom GTM tag but not tied to any particular solution"

## Initial Context

- **Site:** lifeline.org.au (Drupal 11)
- **Analytics stack:** GTM, GA4, Looker
- **GTM Container ID:** GTM-MG3BGD7X
- **GTM is loaded via:** Drupal content block (global script block)

## Investigation Findings

### Drupal Multilingual Modules - NOT in use

Checked the Drupal Extend menu. All multilingual modules are **disabled**:
- Configuration Translation (unchecked)
- Content Translation (unchecked)
- Interface Translation (unchecked)
- Language (unchecked)

This confirmed that translation is NOT happening server-side via Drupal.

### Translation Mechanism - Google Translate Website Translator

The site uses **Google Translate's Website Translator** with a custom UI overlay.

**Key code found in page source:**

```html
<!-- Google Translate loads here (hidden) -->
<div id="google_translate_element" class="hidden"></div>

<!-- Google Translate script -->
<script src="//translate.google.com/translate_a/element.js?cb=googleTranslateElementInit" defer></script>
```

**Custom language selector UI:**

```html
<div class="switcher">
  <button class="switcher-inner" aria-expanded="false" aria-controls="language-switcher-menu">
    <span class="label-translate notranslate">Translate</span>
  </button>

  <div class="switcher-item">
    <ul class="selector-language">
      <li><a href="#" class="language-link" hreflang="en"><span class="notranslate"><strong>English</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="zh-CN"><span class="notranslate"><strong>Mandarin 普通话</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="zh-TW"><span class="notranslate"><strong>Cantonese (廣東話)</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="ar"><span class="notranslate"><strong>Arabic (عربى)</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="vi"><span class="notranslate"><strong>Vietnamese</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="pa"><span class="notranslate"><strong>Punjabi</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="hi"><span class="notranslate"><strong>Hindi</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="el"><span class="notranslate"><strong>Greek</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="it"><span class="notranslate"><strong>Italian</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="es"><span class="notranslate"><strong>Spanish</strong></span></a></li>
      <li><a href="#" class="language-link" hreflang="ne"><span class="notranslate"><strong>Nepali</strong></span></a></li>
    </ul>
  </div>
</div>
```

### Supported Languages

| Language | Code |
|----------|------|
| English | `en` |
| Mandarin | `zh-CN` |
| Cantonese | `zh-TW` |
| Arabic | `ar` |
| Vietnamese | `vi` |
| Punjabi | `pa` |
| Hindi | `hi` |
| Greek | `el` |
| Italian | `it` |
| Spanish | `es` |
| Nepali | `ne` |

### How Translation Works

1. User clicks the "Translate" dropdown button
2. User selects a language from the list
3. JavaScript triggers Google Translate via the hidden `#google_translate_element`
4. Google Translate modifies the `<html>` element:
   - Adds class like `translated-ltr` or `translated-rtl`
   - Changes the `lang` attribute to the target language
5. Page content is translated client-side (URL does not change)
6. Yellow "Translation notice" bar appears at top of page

### Translation Notice

When translated, a notice appears:

> "Translation notice: This page was translated automatically and may not be fully accurate. Spot an issue? Let us Know"

Users can provide feedback via a Tally form embedded in a modal.

## Tracking Approach

Since:
- The URL doesn't change on translation
- The `<html>` element's class and lang attributes DO change
- Each language link has an `hreflang` attribute

We can track translations by:
1. **Primary method:** Listen for clicks on `.language-link` elements and capture the `hreflang` value
2. **Backup method:** Use MutationObserver to detect when Google Translate adds `translated-` class to `<html>`

## Resources

- [Track Page Translations with GTM - Analytics Mania](https://www.analyticsmania.com/post/track-page-translations-with-google-tag-manager-and-google-analytics/)
- [Page Translation GTM Recipe - Analytics Mania](https://www.analyticsmania.com/google-tag-manager-recipes/page-translation/)
- [Page Translation Tracking with GA4 - EZSegment](https://ezsegment.com/page-translation-tracking-using-google-tag-manager-and-ga4/)
