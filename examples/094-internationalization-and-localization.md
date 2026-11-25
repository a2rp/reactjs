# React Internationalization and RTL: Locale Dashboard Example

This example switches among US English, French, and Egyptian Arabic. It formats a date, an amount, a list, and a plural-sensitive count with the active locale; sets language and direction semantics; and uses CSS logical properties for an RTL-ready layout.

Use it in an existing React + TypeScript project. The page markup starts in English, matching the initial locale. The root document metadata is updated when the selected locale changes. For a server-rendered app, provide the initial document language and direction from the server or framework as well.

## File structure

```text
index.html
src/
├── App.tsx
├── App.css
└── main.tsx
```

## `index.html`

Set initial document language and direction to match the initial locale:

```html
<!doctype html>
<html lang="en-US" dir="ltr">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Locale dashboard</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

The script path shown is for a common module-based development setup; use the entry path expected by the project's existing toolchain.

## `src/main.tsx`

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";
import "./App.css";

const rootElement = document.getElementById("root");
if (!rootElement) {
  throw new Error("The root element was not found.");
}

createRoot(rootElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
);
```

## `src/App.tsx`

```tsx
import { useEffect, useMemo, useState } from "react";
import type { ChangeEvent } from "react";

type Locale = "en-US" | "fr-FR" | "ar-EG";
type TextDirection = "ltr" | "rtl";
type PluralCategory = "zero" | "one" | "two" | "few" | "many" | "other";
type PluralForms = Partial<Record<PluralCategory, string>> & {
  other: string;
};

type LocaleMessages = {
  title: string;
  intro: string;
  localeLabel: string;
  dateLabel: string;
  amountLabel: string;
  savedItemsLabel: string;
  topicsLabel: string;
  studentLabel: string;
  directionLabel: string;
  itemCount: PluralForms;
};

const supportedLocales = ["en-US", "fr-FR", "ar-EG"] as const;

const directionByLocale: Record<Locale, TextDirection> = {
  "en-US": "ltr",
  "fr-FR": "ltr",
  "ar-EG": "rtl",
};

const messages: Record<Locale, LocaleMessages> = {
  "en-US": {
    title: "Welcome back",
    intro: "This dashboard formats the same information for your selected locale.",
    localeLabel: "Language",
    dateLabel: "Course date",
    amountLabel: "Example amount in euros",
    savedItemsLabel: "Saved items",
    topicsLabel: "Topics",
    studentLabel: "Student",
    directionLabel: "Text direction",
    itemCount: {
      one: "{count} saved item",
      other: "{count} saved items",
    },
  },
  "fr-FR": {
    title: "Bon retour",
    intro: "Ce tableau de bord adapte les mêmes informations à la langue choisie.",
    localeLabel: "Langue",
    dateLabel: "Date du cours",
    amountLabel: "Montant indicatif en euros",
    savedItemsLabel: "Éléments enregistrés",
    topicsLabel: "Sujets",
    studentLabel: "Élève",
    directionLabel: "Sens du texte",
    itemCount: {
      one: "{count} élément enregistré",
      other: "{count} éléments enregistrés",
    },
  },
  "ar-EG": {
    title: "مرحبًا بعودتك",
    intro: "تعرض لوحة المعلومات هذه البيانات وفقًا للغة التي تختارها.",
    localeLabel: "اللغة",
    dateLabel: "تاريخ الدورة",
    amountLabel: "مبلغ تجريبي باليورو",
    savedItemsLabel: "العناصر المحفوظة",
    topicsLabel: "الموضوعات",
    studentLabel: "الطالب",
    directionLabel: "اتجاه النص",
    itemCount: {
      zero: "لا توجد عناصر محفوظة",
      one: "عنصر محفوظ واحد",
      two: "عنصران محفوظان",
      few: "{count} عناصر محفوظة",
      many: "{count} عنصرًا محفوظًا",
      other: "{count} عنصر محفوظ",
    },
  },
};

const localeNames: Record<Locale, string> = {
  "en-US": "English",
  "fr-FR": "Français",
  "ar-EG": "العربية",
};

const sampleInstant = new Date(Date.UTC(2026, 10, 25, 12));
const savedItemCount = 3;
const sampleAmount = 12849.5;
const topicNames = ["React", "TypeScript", "CSS"];
const displayName = "Maya Haddad · مايا حداد";

function isLocale(value: string): value is Locale {
  return value === "en-US" || value === "fr-FR" || value === "ar-EG";
}

export default function App() {
  const [locale, setLocale] = useState<Locale>("en-US");
  const direction = directionByLocale[locale];
  const copy = messages[locale];

  const formatters = useMemo(
    () => ({
      number: new Intl.NumberFormat(locale),
      currency: new Intl.NumberFormat(locale, {
        style: "currency",
        currency: "EUR",
      }),
      date: new Intl.DateTimeFormat(locale, {
        dateStyle: "long",
        timeZone: "UTC",
      }),
      list: new Intl.ListFormat(locale, {
        style: "long",
        type: "conjunction",
      }),
      collator: new Intl.Collator(locale, { sensitivity: "base" }),
      pluralRules: new Intl.PluralRules(locale),
    }),
    [locale],
  );

  useEffect(() => {
    document.documentElement.lang = locale;
    document.documentElement.dir = direction;
  }, [locale, direction]);

  function handleLocaleChange(event: ChangeEvent<HTMLSelectElement>) {
    const nextLocale = event.currentTarget.value;
    if (isLocale(nextLocale)) setLocale(nextLocale);
  }

  const pluralCategory = formatters.pluralRules.select(savedItemCount);
  const countTemplate =
    copy.itemCount[pluralCategory] ?? copy.itemCount.other;
  const formattedCount = countTemplate.replace(
    "{count}",
    formatters.number.format(savedItemCount),
  );
  const sortedTopics = [...topicNames].sort(formatters.collator.compare);
  const formattedTopics = formatters.list.format(sortedTopics);

  return (
    <main className="page" lang={locale} dir={direction}>
      <div className="dashboard">
        <header className="page-header">
          <div>
            <p className="eyebrow">React locale example</p>
            <h1>{copy.title}</h1>
          </div>
          <label className="locale-picker">
            <span>{copy.localeLabel}</span>
            <select value={locale} onChange={handleLocaleChange}>
              {supportedLocales.map((supportedLocale) => (
                <option key={supportedLocale} value={supportedLocale}>
                  {localeNames[supportedLocale]}
                </option>
              ))}
            </select>
          </label>
        </header>

        <p className="intro">{copy.intro}</p>

        <section className="summary-card" aria-labelledby="summary-title">
          <div className="summary-heading">
            <div>
              <p className="eyebrow">{copy.studentLabel}</p>
              <h2 id="summary-title">
                <bdi dir="auto">{displayName}</bdi>
              </h2>
            </div>
            <span className="direction-badge">
              {copy.directionLabel}: <bdi dir="ltr">{direction.toUpperCase()}</bdi>
            </span>
          </div>

          <dl className="detail-list">
            <div className="detail-row">
              <dt>{copy.dateLabel}</dt>
              <dd>
                <time dateTime={sampleInstant.toISOString()}>
                  {formatters.date.format(sampleInstant)}
                </time>
              </dd>
            </div>
            <div className="detail-row">
              <dt>{copy.amountLabel}</dt>
              <dd>{formatters.currency.format(sampleAmount)}</dd>
            </div>
            <div className="detail-row">
              <dt>{copy.savedItemsLabel}</dt>
              <dd>{formattedCount}</dd>
            </div>
            <div className="detail-row">
              <dt>{copy.topicsLabel}</dt>
              <dd>
                <bdi dir="auto">{formattedTopics}</bdi>
              </dd>
            </div>
          </dl>
        </section>

        <p className="footnote">
          <span lang="en" dir="ltr">Formatting uses the selected locale.</span>
        </p>
      </div>
    </main>
  );
}
```

The `footnote` is intentionally English, so its element declares `lang="en"` and `dir="ltr"` even when the surrounding page is Arabic.

## `src/App.css`

```css
@layer reset, base;

@layer reset {
  *,
  *::before,
  *::after {
    box-sizing: border-box;
  }

  body {
    min-width: 320px;
    min-height: 100vh;
    margin: 0;
  }
}

@layer base {
  body {
    color: #182033;
    background: #f3f6fb;
    font-family: Inter, ui-sans-serif, system-ui, sans-serif;
    line-height: 1.6;
  }

  button,
  input,
  select {
    font: inherit;
  }
}

.page {
  min-height: 100vh;
  padding-block: clamp(2rem, 7vw, 5rem);
  padding-inline: 1rem;
  color: #182033;
  background: #f3f6fb;
}

.dashboard {
  width: min(100%, 760px);
  margin-inline: auto;
}

.page-header,
.summary-heading,
.detail-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 1rem;
}

.page-header {
  align-items: end;
  flex-wrap: wrap;
}

.eyebrow {
  margin: 0 0 0.35rem;
  color: #3157d5;
  font-size: 0.75rem;
  font-weight: 850;
  letter-spacing: 0.1em;
  text-transform: uppercase;
}

h1,
h2,
p {
  margin-block: 0;
}

h1 {
  font-size: clamp(2rem, 7vw, 3.5rem);
  letter-spacing: -0.055em;
  line-height: 1.1;
}

.locale-picker {
  display: grid;
  gap: 0.35rem;
  color: #596579;
  font-size: 0.85rem;
  font-weight: 700;
}

.locale-picker select {
  min-height: 2.75rem;
  min-width: 10rem;
  padding-block: 0.5rem;
  padding-inline: 0.75rem;
  border: 1px solid #cbd4e2;
  border-radius: 0.65rem;
  color: #182033;
  background: #fff;
}

.intro {
  max-width: 42rem;
  margin-block: 1rem 2rem;
  color: #596579;
}

.summary-card {
  padding: clamp(1.1rem, 4vw, 1.75rem);
  border: 1px solid #d9e0eb;
  border-radius: 1rem;
  border-inline-start: 0.3rem solid #3157d5;
  background: #fff;
  box-shadow: 0 16px 45px rgb(24 42 82 / 8%);
}

.summary-heading {
  align-items: start;
  flex-wrap: wrap;
  padding-block-end: 1.2rem;
  border-block-end: 1px solid #e3e8f0;
}

.summary-heading h2 {
  font-size: clamp(1.2rem, 4vw, 1.65rem);
  overflow-wrap: anywhere;
}

.direction-badge {
  padding-block: 0.35rem;
  padding-inline: 0.65rem;
  border-radius: 999px;
  color: #2445b5;
  background: #e8edff;
  font-size: 0.8rem;
  font-weight: 750;
}

.detail-list {
  display: grid;
  gap: 0;
  margin: 0;
}

.detail-row {
  align-items: baseline;
  padding-block: 0.85rem;
  border-block-end: 1px solid #edf0f5;
}

.detail-row:last-child {
  border-block-end: 0;
}

.detail-row dt {
  color: #596579;
}

.detail-row dd {
  margin: 0;
  font-weight: 750;
  text-align: end;
  overflow-wrap: anywhere;
}

.footnote {
  margin-block-start: 1rem;
  color: #596579;
  font-size: 0.9rem;
}

:focus-visible {
  outline: 3px solid #7898ff;
  outline-offset: 3px;
}

@media (max-width: 520px) {
  .page-header,
  .detail-row {
    align-items: start;
    flex-direction: column;
  }

  .detail-row dd {
    text-align: start;
  }
}
```

The CSS uses logical properties such as `padding-block`, `padding-inline`, `border-inline-start`, and `text-align: end`. They adapt to the element's writing direction without maintaining separate left-to-right and right-to-left stylesheets.

## How localization works

1. The locale selector updates one typed React state value.
2. `directionByLocale` keeps writing direction separate from the language tag.
3. The message catalog provides translated labels and plural forms.
4. Memoized `Intl` formatters handle dates, currency, number grouping, list joining, sorting, and plural-category selection.
5. The count category selects a translated template; `Intl.NumberFormat` supplies the localized count.
6. The page and document root receive `lang` and `dir` updates. The English footnote declares its own language and direction.
7. `<bdi dir="auto">` isolates a mixed-script display name and list from surrounding punctuation and direction.

## Try it

1. Select **Français**. Check translated labels, French number/date/list conventions, and the left-to-right layout.
2. Select **العربية**. Check that the page switches to RTL and that the embedded Latin and Arabic name remains isolated.
3. Inspect the document root. Its `lang` and `dir` should match the selected locale; the main region should expose the same values.
4. Change `savedItemCount` to `1`, `2`, `0`, and `11`. Check that the locale's plural category and translated form change appropriately.
5. Change the amount or locale. Confirm that the amount remains euros while its display conventions change.
6. Resize the page and inspect long text. It should wrap without overlapping the locale selector or summary values.
7. Use the keyboard to change the locale and confirm the focus indicator remains visible.

## Key takeaways

- Translation messages, `Intl` formatting, document language/direction, and CSS layout are related but separate concerns.
- `Intl.PluralRules` chooses a category; the catalog supplies the translated text for that category.
- A locale can change formatting without changing the underlying date instant or currency value.
- `lang` declares language, `dir` sets direction, and `<bdi>` isolates mixed-direction inline values.
- CSS logical properties make the same component layout work in LTR and RTL contexts.
