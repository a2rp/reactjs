# Internationalization, Localization, and Text Direction: Questions and Answers

## Core concepts and messages

### 1. What is internationalization?
Internationalization (i18n) is preparing an application to support different languages, writing systems, formats, calendars, currencies, and reading directions.

### 2. What is localization?
Localization (l10n) adapts an application for a selected locale, including translated messages and locale-specific presentation.

### 3. Is a locale the same as a language?
No. A locale can include a language and region, such as `en-US` or `en-GB`, which may use different date or number conventions.

### 4. Does React provide a complete i18n framework?
No. React renders the messages the app supplies. Use catalogs and, if needed, a localization library for message loading, interpolation, plural forms, and fallback behavior.

### 5. How should translated messages be organized?
Keep them in message catalogs indexed by a supported locale. Components should retrieve complete messages from the active catalog rather than embedding locale conditionals throughout JSX.

### 6. Why should I avoid concatenating translated sentence pieces?
Word order, grammar, punctuation, and agreement vary between languages. Translate a whole sentence with named placeholders so translators can place values correctly.

### 7. Is it safe to render a translated string as HTML?
Render it as a normal React text value whenever possible. Do not use `dangerouslySetInnerHTML` for translations or user content unless HTML is required and safely sanitized.

### 8. What should happen when a translation is missing?
Use an explicit, predictable fallback such as the source locale or a key-specific fallback. Do not leave important controls or accessible messages empty.

## Locale-sensitive formatting

### 9. How do I format a number for a locale?
Use `Intl.NumberFormat(locale).format(value)` instead of manually choosing grouping and decimal separators.

### 10. How do I format currency?
Use `Intl.NumberFormat` with `style: "currency"` and the actual currency code. The locale controls display conventions; it should not silently change the underlying currency.

### 11. How do I format a date or time?
Use `Intl.DateTimeFormat` with the active locale and explicit options. Choose a time zone according to the meaning of the value.

### 12. Why should I specify a time zone?
The same instant can fall on different local calendar dates in different zones. Explicitly choose the viewer's, business's, or event's intended time zone.

### 13. Is every date a JavaScript `Date`?
No. `Date` represents an instant. A date-only value, such as a birthday, may be a calendar date with no time zone; converting it as an instant can shift the displayed day.

### 14. How do I format a list of values?
Use `Intl.ListFormat` to produce locale-appropriate conjunction or disjunction punctuation.

### 15. How do I sort names for a locale?
Use `Intl.Collator(locale).compare` rather than assuming raw string order matches the language's sorting rules.

### 16. What is `Intl.RelativeTimeFormat` for?
It formats relative values such as “in two days” or “yesterday.” It formats a relative-time expression, not an entire application message.

### 17. What does `Intl.PluralRules` return?
It selects a plural category for a numeric value and locale, such as `one`, `few`, or `other`. It does not return a translated sentence.

### 18. Why is `count === 1` not a general plural rule?
Many languages use more than two plural categories or have rules that differ from English. Select the locale's category and provide the corresponding translated form.

### 19. Should an app cache `Intl` formatter instances?
For formatters reused repeatedly, it can be useful to create them once per locale or memoize them. Recreating a few formatters in a small example may be acceptable; avoid unnecessary work in large render loops.

### 20. Does changing the locale necessarily change currency?
No. Locale is a presentation choice. Currency is usually a separate value defined by the product, account, or transaction.

## Language and direction semantics

### 21. What does the HTML `lang` attribute declare?
It declares the language of an element's content using a language tag. This helps assistive technologies and other tools process the text appropriately.

### 22. What does the HTML `dir` attribute declare?
It sets the base text direction, usually `ltr`, `rtl`, or `auto`. It affects bidirectional text and should express content direction rather than be treated only as a visual alignment rule.

### 23. Does `lang="ar"` automatically make the page RTL?
No. Language and direction are separate. Set both, for example `lang="ar"` and `dir="rtl"` for Arabic content.

### 24. Where should a page's primary `lang` and `dir` go?
Set them on the document's root `<html>` element. A page with mixed-language sections can override them on the relevant elements.

### 25. How should a React single-page app change the document language?
Keep the chosen locale in a single source of truth, then update the document root metadata through the app or framework's document mechanism. If server rendering, send the correct initial `lang` and `dir` in the response.

### 26. Why is updating `lang` and `dir` only after hydration a problem?
The initial HTML can be announced or displayed with the wrong language or direction, and the server and client may render differently. Provide matching initial metadata and application state.

### 27. What does `dir="auto"` do?
It asks the browser to choose a base direction from the text's first strong directional character. It is useful for unknown-direction user input, but it is not language detection.

### 28. What is `<bdi>` for?
It isolates a short inline value whose direction may differ from surrounding text, such as a user name, identifier, or mixed-script value. It prevents its directional behavior from affecting adjacent punctuation.

### 29. Should I reverse strings for RTL layouts?
No. Do not reverse characters manually. Use semantic `dir` attributes, `<bdi>`, and layouts that support the correct writing direction.

### 30. Should every icon be mirrored in RTL?
No. Mirror only icons whose meaning follows reading or navigation direction. Physical objects and universal symbols may need to keep their orientation.

## CSS and React layout

### 31. Which CSS properties help layouts adapt to RTL?
Use logical properties such as `margin-inline-start`, `padding-inline-end`, `border-inline-start`, `text-align: start`, and `inset-inline-start` when the meaning follows reading order.

### 32. When are physical `left` and `right` properties still appropriate?
Use them when the design specifically refers to a physical screen edge, rather than a logical start or end that should switch with writing direction.

### 33. Does setting `dir="rtl"` solve every layout issue?
No. Review alignment, flex/grid ordering, icons, mixed text, punctuation, inputs, scrolling, and navigation in both directions. Direction affects text flow but does not translate content.

### 34. How should language selection work in React?
Store the chosen supported locale in state or a shared preference store. Render its catalog, formatters, `lang`, and `dir` from that same value.

### 35. Should locale come from browser settings automatically?
Browser preferences can suggest a default, but the product should define a supported-locale matching and fallback policy. Provide a user choice when the app supports one.

### 36. How do I avoid server/client locale mismatch?
Pass the same initial locale into both server and client rendering. Do not let the server render one catalog while the browser silently initializes another before hydration.

## Accessibility and quality

### 37. What needs translation besides visible headings?
Translate form labels, accessible names, alternative text, validation errors, status messages, document titles, and other text exposed to assistive technology.

### 38. Why test long translations?
Translated strings can be longer or shorter than the source. Flexible layouts prevent clipping, overlap, or controls becoming unusable.

### 39. What should I test for an RTL locale?
Check reading order, alignment, punctuation, mixed scripts, numbers, icons, focus order, and any directional navigation affordances.

### 40. What should I test for locale formatting?
Check dates with the intended time zone, currency and number output, list punctuation, plural categories, and fallback behavior for missing messages.

### 41. Should locale-specific formatting be hard-coded in a message catalog?
Usually no for dynamic values. Keep the translated message in the catalog and format dates, numbers, and currencies using the active locale and the intended business data.

### 42. Can CSS alone localize an interface?
No. CSS can adapt layout and direction-sensitive presentation, but it does not translate messages, format values, or provide correct language metadata.

### 43. What is a good first i18n implementation?
Choose a supported locale model, centralize translated messages, use `Intl` for dynamic formatting, set root language and direction, and test the UI in at least one RTL locale.

### 44. What is the most important separation to maintain?
Keep translation text, locale-sensitive formatting, semantic language/direction metadata, and layout presentation as related but distinct concerns. Each has a different correct mechanism.
## Chapter navigation

<table width="100%">
  <thead>
    <tr><th>Previous Q&amp;A</th><th>Chapter lesson</th><th>Chapter example</th><th>Next Q&amp;A</th></tr>
  </thead>
  <tbody>
    <tr><td><a href="093-animation-and-motion-preferences.md">093 - Animation, Motion Preferences, and View Transitions: Questions and Answers</a></td><td><a href="../lessons/094-internationalization-and-localization.md">Lesson</a></td><td><a href="../examples/094-internationalization-and-localization.md">Example</a></td><td><a href="095-frameworks-and-rendering-strategies.md">095 - Frameworks, Client Rendering, SSR, and Static Generation: Questions and Answers</a></td></tr>
  </tbody>
</table>
