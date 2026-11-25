# Internationalization, Localization, and Text Direction in React

## Learning goals

After this lesson, you should be able to:

- Distinguish internationalization (i18n) from localization (l10n).
- Keep translated messages separate from React component structure.
- Format dates, numbers, currencies, lists, and plural-sensitive values for a locale.
- Set document language and text direction independently.
- Build layouts that work in left-to-right and right-to-left languages.
- Isolate mixed-direction text safely and support translated accessible names.
- Choose a locale consistently across browser rendering, server rendering, and user preferences.

## 1. What internationalization means

**Internationalization** prepares an application so it can support different languages, writing systems, number conventions, calendars, currencies, and reading directions. **Localization** adapts the application for a particular locale, including its translated content and conventions.

A locale is more specific than a language in many cases. `en-US` and `en-GB` are both English, but they can use different date, number, and currency conventions. Locale tags follow BCP 47 conventions; use the tags that match the content and formatting data your app supports.

React does not include a full translation catalog or choose a localization architecture. It renders the messages and values the application provides. The JavaScript `Intl` APIs handle many locale-sensitive formatting rules; a message catalog or localization library supplies translated sentences and plural forms.

## 2. Keep messages separate from components

Avoid scattering translated strings through conditional expressions in JSX. Keep messages in a catalog indexed by the active locale, then have components render the selected messages:

```tsx
type Locale = "en-US" | "fr-FR";

type Messages = {
  heading: string;
  save: string;
  cancel: string;
};

const messages: Record<Locale, Messages> = {
  "en-US": {
    heading: "Account settings",
    save: "Save changes",
    cancel: "Cancel",
  },
  "fr-FR": {
    heading: "Paramètres du compte",
    save: "Enregistrer les modifications",
    cancel: "Annuler",
  },
};

function Settings({ locale }: { locale: Locale }) {
  const copy = messages[locale];
  return <button type="button">{copy.save}</button>;
}
```

The message catalog is independent from the JSX layout. Larger apps usually load catalogs by feature or route and use a translation system that supports interpolation, plural messages, fallback behavior, and translator context.

### Do not build translated sentences by concatenation

Different languages use different word order, grammar, punctuation, and gender agreements. Avoid composing a sentence from fragments such as `"Hello " + name + "!"`. Use a complete translatable message with a named placeholder so the translator can move the value as needed.

```text
"welcome": "Welcome, {name}!"
```

A translation system should interpolate the `name` into the message safely. React escapes text rendered as a value; do not turn translations or user-provided text into HTML with `dangerouslySetInnerHTML` unless it has been carefully sanitized and that behavior is required.

## 3. Format data with the `Intl` APIs

Translated labels and locale-formatted values are related but different jobs. Use `Intl` rather than manually inserting separators, currency symbols, or month names.

### Numbers

```js
const number = new Intl.NumberFormat("de-DE");
number.format(1234567.89); // locale-appropriate grouping and decimal mark
```

### Currency

```js
const price = new Intl.NumberFormat("en-GB", {
  style: "currency",
  currency: "GBP",
});
price.format(1234.5);
```

The locale controls how the currency is displayed; the currency code identifies which currency the amount represents. Do not assume that changing the locale should silently change the underlying currency.

### Dates and times

```js
const dateTime = new Intl.DateTimeFormat("ja-JP", {
  dateStyle: "long",
  timeStyle: "short",
  timeZone: "Asia/Tokyo",
});
```

Choose the time zone from the meaning of the date:

- An instant such as a server timestamp needs an explicit display time zone, often the user's selected or device time zone.
- A date-only value such as a birthday or due date may not represent an instant. Avoid converting it through UTC in a way that moves it to another calendar day.
- A business event may need the business's time zone even when the visitor is elsewhere.

A JavaScript `Date` stores an instant, not a time zone or a date-only calendar value. Keep that distinction in the application's data model.

### Lists, collation, and relative time

Use `Intl.ListFormat` to join a list according to a locale:

```js
const list = new Intl.ListFormat("en-US", {
  style: "long",
  type: "conjunction",
});
list.format(["React", "TypeScript", "CSS"]);
```

Use `Intl.Collator` to compare and sort user-visible text according to a locale rather than comparing strings by raw code-point order:

```js
const collator = new Intl.Collator("sv-SE", {
  sensitivity: "base",
});
const sortedNames = [...names].sort(collator.compare);
```

`Intl.RelativeTimeFormat` can format values such as “in 2 days” or “yesterday.” Use a message system for full application sentences; a formatter handles a particular value, not all grammar around it.

## 4. Plural forms need locale rules

A common mistake is to choose between singular and plural by checking whether a count equals one:

```js
// Too limited for many languages
count === 1 ? "1 item" : `${count} items`;
```

Languages can have more than two plural categories, and their rules vary. `Intl.PluralRules` can select a category for a locale:

```js
const rules = new Intl.PluralRules("ar-EG");
const category = rules.select(3);
```

Plural rules return a category such as `one`, `few`, or `other`; they do not translate a message. Store the appropriate translated forms in a catalog or use a message-formatting system that supports locale-aware plural messages.

```text
English:
  one:   "{count} saved item"
  other: "{count} saved items"

Arabic:
  zero, one, two, few, many, other: locale-specific translated forms
```

Interpolate a locale-formatted count into the chosen message. Do not assume every language uses only `one` and `other`.

## 5. Put locale and direction into document semantics

The HTML `lang` attribute identifies the language of content. The `dir` attribute specifies its base direction. They answer different questions: a language tag does not imply text direction.

Set both on the root document for the primary language and direction:

```html
<html lang="ar-EG" dir="rtl">
```

A left-to-right document can contain an explicitly right-to-left section, and the reverse is also possible:

```html
<section lang="ar" dir="rtl">
  <h2>...</h2>
  <p>...</p>
</section>
```

Use the correct `lang` on passages in a different language so assistive technology can select appropriate pronunciation. Also translate accessible names, form labels, error messages, alternative text, and status messages; visible strings are not the only user-facing text.

For a client-rendered app that lets a person change languages, keep the selected locale in one source of truth and update document metadata at the application boundary. For example:

```tsx
useEffect(() => {
  document.documentElement.lang = locale;
  document.documentElement.dir = direction;
}, [locale, direction]);
```

For server-rendered pages, provide the correct `lang` and `dir` in the HTML response on the first render. Updating them only after hydration can leave the initial content with the wrong language or direction. Keep the document title and other metadata localized through the framework's document or head mechanism as well.

## 6. Build layouts for both directions

Avoid assuming that left is always the start of a line or that right is always the end. CSS logical properties adapt to writing direction:

```css
.card {
  padding-inline: 1rem;
  margin-block: 1rem;
  border-inline-start: 0.25rem solid royalblue;
  text-align: start;
}
```

`inline-start` and `inline-end` follow the writing direction. For horizontal text, start is left in LTR and right in RTL. `block-start` and `block-end` refer to the block flow direction.

Prefer logical properties such as `margin-inline-start`, `padding-inline-end`, and `inset-inline-start` when the meaning is about reading order. Use physical properties like `left` or `right` when the design really means that physical screen edge.

Not every symbol should be mirrored. Some icons indicate a physical object or a universal action; others, such as a back arrow or reading progression, may need direction-aware treatment. Test each icon and navigation affordance in the intended direction rather than flipping every image.

## 7. Isolate mixed-direction text

Right-to-left text can contain Latin names, IDs, dates, or numbers. Their punctuation can appear in surprising positions if their direction interacts with surrounding text.

Use `<bdi>` to isolate an inline value whose direction is unknown, or `dir="auto"` for a block of user-provided text with unknown direction:

```tsx
<p>
  {messages[locale].greeting} <bdi dir="auto">{displayName}</bdi>
</p>

<p dir="auto">{userComment}</p>
```

`dir="auto"` uses the content to determine a base direction. It is useful for unknown-direction input, but it does not detect the language. For known content, provide the known `lang` and `dir` explicitly.

Avoid manually reversing strings or using Unicode direction-control characters as a substitute for semantic markup. Use `<bdi>`, `dir`, or a semantic element with the correct direction.

## 8. Locale selection and fallback

A product can suggest a starting locale from the browser's language preferences, an account setting, or a URL. A visitor should still be able to select a supported language when the product offers that choice.

Keep the policy explicit:

1. Determine the locale from a deliberate source, such as the route or saved profile.
2. Match it to a locale the application actually supports.
3. Load the corresponding messages and formatting rules.
4. Provide a predictable fallback if a message or locale is unavailable.
5. Preserve the selected locale in navigation and persistence according to the app's product requirements.

Do not let the server choose one locale and the client silently choose another during hydration. Pass the same initial locale to both renders. Avoid relying on a browser-only API before the initial server-rendered markup is created.

## 9. Test localized interfaces

Test more than whether translated words appear:

- Long strings fit without clipping or overlapping controls.
- Number, date, currency, list, and plural output match the selected locale.
- The time zone is correct for the value's meaning.
- The document and translated passages have correct `lang` attributes.
- RTL layout, punctuation, numbers, and mixed-script text render correctly.
- Keyboard focus and reading order remain sensible after direction changes.
- Accessible labels, error messages, and status announcements use the selected language.
- The server and client begin with the same locale and direction.
- A missing translation uses a clear fallback rather than an empty control.

Localization can change content length and direction, so use flexible layouts and avoid fixed widths for translated text.

## 10. Common mistakes

- **Treating language and direction as the same setting:** set `lang` and `dir` independently.
- **Writing custom date or currency punctuation:** use an `Intl` formatter for the selected locale.
- **Changing currency when changing language:** keep currency as business data and format it for the locale.
- **Using only `count === 1` for plurals:** select locale-aware forms and translate each required category.
- **Concatenating translated sentence fragments:** translate the complete message with placeholders.
- **Using `left` and `right` for every layout:** use CSS logical properties where the meaning follows reading order.
- **Assuming `dir="auto"` detects a language:** it determines base direction from text, not its language.
- **Ignoring mixed-direction inline values:** isolate them with `<bdi>` or a semantic element with a direction.
- **Updating document language after a wrong server render:** supply correct document metadata in the server response.
- **Translating only visible headings:** include labels, alternative text, errors, titles, and status messages.
- **Hard-coding locale-formatted values in translations:** format dynamic data with `Intl` using the active locale.

## Summary

Internationalization makes an application ready for different languages, formats, and directions; localization supplies the actual translated content and locale choices. Keep messages in catalogs, use `Intl` for locale-sensitive formatting and plural categories, and set `lang` and `dir` deliberately. Use logical CSS properties and isolate unknown-direction text with `<bdi>` or `dir="auto"`. Match server and client locale settings, translate accessible content, and test long strings, RTL, dates, numbers, and plural forms.
