# Unit Price: build spec

Unit Price is a phone-first web page that tells a supermarket customer which of two or more items is the cheapest. This file is the single source of truth for building it with a coding CLI. Every product decision is already made here.

To start a build: put this file in an empty folder, start the CLI in that folder, and tell it "Build the app described in user_stories.md."

## 1. Rules for the builder

1. Build everything in this file, in the order given in section 10. Do not start a step before the check of the previous step passes.
2. Do not stop to ask questions. Sections 3 to 9 answer every product question.
3. If you find a gap, choose the simplest behaviour that fits section 3, write it under "Assumptions made during the build" in README.md, and continue.
4. The exact text in section 6.3 and the numbers in section 5.9 are requirements, not illustrations.
5. Do not add features, files, libraries or tools that this file does not name. Section 14 lists what is out of scope.
6. You may use throwaway scripts and a local web server to check your work. Remove them before you finish. Leave no test code, package files or build output in the project.
7. Do not create a git repository, commit, push or deploy (decision D14).
8. When you finish, go through section 13 line by line and report each line as done or not done.

## 2. What to build

The customer types a price and a quantity for each item, picks each item's unit, and taps Compare. The app shows each item's price for the same standard amount and marks the cheapest. It works with no internet after the first load, and it can be added to the phone's home screen.

## 3. Hard constraints

- **Technology:** HTML, CSS and JavaScript only. No framework, library, CDN, web font, package manager, bundler, transpiler or build step. The only other files allowed are the ones listed in section 9.
- **Mobile first:** every screen is designed for a phone first. Section 6.1 has the rules.
- **Offline:** after the first load the app needs no network. Section 7 has the rules.
- **No outside requests:** the app only ever loads its own files. No analytics, no external fonts, images or scripts.
- **Privacy:** everything the customer types stays on the phone.
- **Language:** English only, written directly in the page.
- **Browsers:** current Safari on iPhone and current Chrome on Android. It must also work in current desktop Chrome, Safari and Firefox. Older browsers are not supported.

## 4. Decisions

### 4.1 Decided by the product owner (2026-11-14)

| ID | Question | Decision |
|---|---|---|
| D1 | Units a customer can enter | g, oz, ml and fl oz. No kg, lb or litres. |
| D2 | Items sold by count | Yes. A "pieces" unit, priced per 1 piece. |
| D3 | When results appear | Only after the customer taps Compare. |
| D4 | An item is missing its price or quantity | Compare the complete items. Flag the incomplete one and leave it out. A row never touched is ignored. |
| D5 | Preferred unit | One switch: Metric or Imperial. |
| D6 | Amount a price is shown for | Metric: per 100 g and per 100 ml. Imperial: per 1 oz and per 1 fl oz. |
| D7 | Which fluid ounce | The US fluid ounce. |
| D8 | Weight, volume and pieces mixed in one comparison | Not compared. A message asks for one kind at a time. |
| D9 | Decimals | 2, and more (3, then 4) only when two items would otherwise look the same. |
| D10 | Extras in the first build | Remove an item, Clear all, optional item names, remember items between visits. |
| D11 | Offline delivery | A hosted page with a service worker, plus a manifest and icons so it can be added to the home screen. |
| D12 | Tests | No test code. The builder writes manual test cases, including edge cases, in Test.md. |
| D13 | App name | Unit Price |
| D14 | Where the build stops | A local build with a README. The owner publishes it. |
| D15 | Currency | A dollar sign before the number. |
| D16 | Language | English only. |

### 4.2 Defaults set by the spec author

These were not in the brief and were not asked. They are choices a builder would otherwise make silently. The owner can change any of them by editing this table.

| ID | Default |
|---|---|
| F1 | The unit converter (story 7) is in scope. The brief said "only do if it is easy", and D1 with D5 already need the same conversion. |
| F2 | A typed number is digits with at most one "." or ",". It must be more than 0. A quantity in pieces must be a whole number. |
| F3 | A row is "started" as soon as its name, price or quantity holds any text. |
| F4 | In a tie, every item at the lowest price is marked. |
| F5 | There is no maximum number of rows, and never fewer than two. |
| F6 | A new row copies the unit of the row above it. The first two rows start in g (Metric) or oz (Imperial). |
| F7 | Metric is selected on first use. |
| F8 | Any edit after a result clears the result. Switching between Metric and Imperial re-shows it at once. |
| F9 | Clear all asks for no confirmation. |
| F10 | When kinds are mixed, no unit price is shown at all. |
| F11 | A line "Ready to use offline" appears once the offline copy is saved. |
| F12 | Light theme only, system fonts, and the colours in section 6.7. |
| F13 | Larger screens show the same layout in a centred column. |
| F14 | Field limits: 30 characters for a name, 9 characters for a price or quantity. |
| F15 | Items and the Metric or Imperial choice are remembered between visits. Results are not. |
| F16 | Remove on one of only two rows empties that row instead of deleting it. |
| F17 | "Mobile first" is turned into the testable rules of section 6.1: a 320 px base width, 16 px input text, 44 px controls and a 480 px column on larger screens. |
| F18 | The wording of every text in section 6.3, and the accessibility rules in section 6.6. |
| F19 | The file layout in section 9, the storage format in section 6.5 and the service worker strategy in section 7.1. |

## 5. Calculation rules

### 5.1 Units

| Unit as shown | Kind | Base unit | One unit in base units |
|---|---|---|---|
| g | weight | g | 1 |
| oz | weight | g | 28.349523125 |
| ml | volume | ml | 1 |
| fl oz | volume | ml | 29.5735295625 |
| pieces | count | piece | 1 |

The two factors are exact by definition: 1 oz = 453.59237 g / 16, and 1 US fl oz = 3785.411784 ml / 128. Use every digit. Reference P3 in section 15 is the place to validate them.

### 5.2 Reading a typed number

1. Remove spaces at both ends.
2. If nothing is left, the field is empty.
3. The text must match this pattern, otherwise it is not a number:

```
^(\d+([.,]\d*)?|[.,]\d+)$
```

4. Replace "," with "." and convert to a number. So 4,99 is 4.99, "4." is 4 and ".5" is 0.5.
5. The number must be more than 0.
6. If the row's unit is pieces, the quantity must be a whole number.

A minus sign, a currency symbol, a thousands separator, letters, a second separator or a space inside the text all fail step 3. Check in this order, and show only the first failure for a field.

### 5.3 Which rows count

- **Untouched row:** name, price and quantity are all empty. It is ignored and never shows a message.
- **Started row:** any other row.
- **Complete row:** a started row whose price and quantity both pass section 5.2. The name does not matter.

### 5.4 Base price

```
base price = price / (quantity * factor of the row's unit)
```

This is the price per g, per ml or per piece.

### 5.5 Displayed unit price

| Kind | Metric | Imperial |
|---|---|---|
| weight | base price * 100, label "per 100 g" | base price * 28.349523125, label "per oz" |
| volume | base price * 100, label "per 100 ml" | base price * 29.5735295625, label "per fl oz" |
| count | base price, label "per piece" | base price, label "per piece" |

Format: a dollar sign, the number, a space, the label. Example: `$0.80 per 100 g`. Always a decimal point, never a thousands separator.

### 5.6 Cheapest and ties

- Compare base prices. Never compare rounded or displayed numbers.
- Two base prices a and b are equal when `abs(a - b) <= 1e-9 * max(a, b)`. A plain `===` is wrong here; example C7 shows why.
- The cheapest row has the lowest base price. Every row equal to the lowest is a joint cheapest: a tie.

### 5.7 Rounding and decimals

- Round half up.
- Remove binary floating-point noise before rounding. This method passes every example:

```
rounded = Math.round(Number((value * 10 ** d).toPrecision(12))) / 10 ** d
text    = rounded.toFixed(d)
```

- A plain `value.toFixed(d)` is wrong; examples C3 and C8 show why.
- The number of decimals d is chosen once per result and used for every row in it. Start with 2. If any two rows that are not tied would show the same text, use 3. If that still happens, use 4. Never use more than 4.
- Choose d again whenever the result is re-shown in the other system.

### 5.8 What a tap on Compare does

1. Check every started row against sections 5.2 and 5.3. Show a field message on each field that fails.
2. Take the complete rows.
   - If there are none, show the summary "Enter a price and quantity for at least two items." and stop.
   - If they are not all one kind, show the summary "Compare one kind at a time: weight (g, oz), volume (ml, fl oz) or pieces." Show no unit price. Stop.
3. Show the unit price on each complete row.
4. If exactly one row is complete, show the summary "Add another item to compare." If two or more are complete, mark the cheapest row or rows and show the cheapest or tie summary.
5. If step 3 was reached and one or more started rows failed step 1, also show the note "1 item left out. Check the highlighted fields." (or "N items left out. ...").

### 5.9 Worked examples

Rows are written as price / quantity unit. "Item N" is the row's position. Every example was run through a reference implementation of sections 5.1 to 5.8 and cross-checked with exact arithmetic.

| ID | Rows typed | Metric shows | Imperial shows | Summary and notes |
|---|---|---|---|---|
| C1 | 4.00 / 500 g; 3.00 / 300 g | $0.80 and $1.00 per 100 g | $0.23 and $0.28 per oz | Cheapest: Item 1 |
| C2 | 2.49 / 750 ml; 1.99 / 500 ml | $0.33 and $0.40 per 100 ml | $0.10 and $0.12 per fl oz | Cheapest: Item 1 |
| C3 | 3.99 / 12 pieces; 5.49 / 18 pieces | $0.33 and $0.31 per piece | $0.33 and $0.31 per piece | Cheapest: Item 2. The exact value 0.305 rounds half up to 0.31. |
| C4 | 3.79 / 340 g; 3.99 / 12 oz | $1.11 and $1.17 per 100 g | $0.32 and $0.33 per oz | Cheapest: Item 1 |
| C5 | 1.29 / 500 ml; 1.29 / 16.9 fl oz | $0.2580 and $0.2581 per 100 ml | $0.0763 and $0.0763 per fl oz | Cheapest: Item 1, with 4 decimals. In Imperial both rows still show the same text; only Item 1 gets the badge. |
| C6 | 2.00 / 200 g; 4.00 / 400 g | $1.00 and $1.00 per 100 g | $0.28 and $0.28 per oz | Tie for cheapest: Item 1 and Item 2 |
| C7 | 0.30 / 3 g; 0.10 / 1 g | $10.00 and $10.00 per 100 g | $2.83 and $2.83 per oz | Tie for cheapest: Item 1 and Item 2. A plain === comparison wrongly picks Item 1. |
| C8 | 2.01 / 200 g; 3.00 / 300 g | $1.01 and $1.00 per 100 g | $0.285 and $0.283 per oz | Cheapest: Item 2. The exact value 1.005 rounds half up to 1.01. Imperial needs 3 decimals. |
| C9 | 1.99 / 250 g; 3.18 / 400 g | $0.796 and $0.795 per 100 g | $0.226 and $0.225 per oz | Cheapest: Item 2, with 3 decimals |
| C10 | 4.00 / 500 g; (empty price) / 300 g; 3.00 / 300 g | Item 1 $0.80, Item 3 $1.00 per 100 g | Item 1 $0.23, Item 3 $0.28 per oz | Cheapest: Item 1. Item 2 shows "Enter a price" and no unit price. Note: "1 item left out. Check the highlighted fields." |
| C11 | 4.00 / 500 g; 2.49 / 750 ml | no unit price | no unit price | "Compare one kind at a time: weight (g, oz), volume (ml, fl oz) or pieces." |
| C12 | 4,99 / 500 g; 5,39 / 600 g | $1.00 and $0.90 per 100 g | $0.28 and $0.25 per oz | Cheapest: Item 2. A comma is read as a decimal point. |
| C13 | 3.99 / 1.5 pieces; 5.49 / 18 pieces | Item 2 $0.31 per piece | Item 2 $0.31 per piece | Item 1 shows "Use a whole number of pieces". Summary: "Add another item to compare." Note: "1 item left out. Check the highlighted fields." |
| C14 | 4.00 / 500 g; (untouched row) | Item 1 $0.80 per 100 g | Item 1 $0.23 per oz | "Add another item to compare." No note about items left out. |
| C15 | abc / 500 g; 0 / 300 g; 3.00 / 1.2.3 g; -1 / 0 g | no unit price | no unit price | Item 1 price: "Use numbers only, like 4.99". Item 2 price: "Must be more than 0". Item 3 quantity: "Use numbers only, like 4.99". Item 4 price: "Use numbers only, like 4.99", quantity: "Must be more than 0". Summary: "Enter a price and quantity for at least two items." |
| C16 | C1 with the names "Store brand" and "Name brand" | $0.80 and $1.00 per 100 g | $0.23 and $0.28 per oz | "Cheapest: Store brand at $0.80 per 100 g." |
| C17 | 2.00 / 200 g; 4.00 / 400 g; 5.00 / 400 g | $1.00, $1.00 and $1.25 per 100 g | $0.28, $0.28 and $0.35 per oz | Tie for cheapest: Item 1 and Item 2 |
| C18 | 3.99 / 12 oz; 4.49 / 14 oz | $1.17 and $1.13 per 100 g | $0.33 and $0.32 per oz | Cheapest: Item 2 |
| C19 | two untouched rows | no unit price | no unit price | "Enter a price and quantity for at least two items." No field message. |
| C20 | 4. / 500 g; .5 / 50 g | $0.80 and $1.00 per 100 g | $0.23 and $0.28 per oz | Cheapest: Item 1. "4." and ".5" are valid numbers. |

## 6. Screen and behaviour

### 6.1 Mobile-first rules

These apply to every story.

1. The page has `<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">`.
2. The base CSS is written for a screen 320 px wide. Wider layouts are added only with min-width media queries.
3. At 320 px wide there is no horizontal scrolling and nothing is cut off.
4. From 600 px wide, the content sits in a centred column no wider than 480 px.
5. Price and quantity are `<input type="text" inputmode="decimal">`, which gives a numeric keypad with a decimal key and lets a comma be typed.
6. Text inside inputs and selects is at least 16 px.
7. Every button, input and select is at least 44 px high, and every button at least 44 px wide.
8. Nothing depends on hover.
9. Page padding uses `env(safe-area-inset-*)` so content clears the notch and the home indicator in home-screen mode.

### 6.2 Layout, top to bottom

1. **Header:** the app name, then the control "Show results in" with the two options Metric and Imperial.
2. **Item list:** one card per row. A card holds the name field, the Remove button, the price field, the quantity field, the unit selector, a line for the row's unit price and badge, and the field messages.
3. **Add item** button.
4. **Compare** button: full width and the most prominent control on the page.
5. **Summary area:** the summary line and the note. After every tap on Compare it is scrolled into view.
6. **Clear all** button: visually secondary.
7. **Footer:** the line "Ready to use offline", when it applies.

### 6.3 Exact text

| Where | Text |
|---|---|
| Page title and header | Unit Price |
| System control label | Show results in |
| System options | Metric, Imperial |
| Name field label | Name (optional) |
| Name field placeholder | Item N |
| Price field label | Price ($) |
| Quantity field label | Quantity |
| Unit selector label | Unit |
| Unit options, in this order | g, oz, ml, fl oz, pieces |
| Buttons | Add item, Compare, Clear all, Remove |
| Badges | Cheapest, Tie |
| Unit price labels | per 100 g, per 100 ml, per oz, per fl oz, per piece |
| Summary, one cheapest | Cheapest: [label] at [unit price]. |
| Summary, tie of two | Tie for cheapest: [label] and [label] at [unit price]. |
| Summary, tie of three or more | Tie for cheapest: [label], [label] and [label] at [unit price]. |
| Summary, one complete row | Add another item to compare. |
| Summary, no complete row | Enter a price and quantity for at least two items. |
| Summary, mixed kinds | Compare one kind at a time: weight (g, oz), volume (ml, fl oz) or pieces. |
| Note, one row left out | 1 item left out. Check the highlighted fields. |
| Note, several rows left out | N items left out. Check the highlighted fields. |
| Field message, empty price | Enter a price |
| Field message, empty quantity | Enter a quantity |
| Field message, not a number | Use numbers only, like 4.99 |
| Field message, zero | Must be more than 0 |
| Field message, pieces not whole | Use a whole number of pieces |
| Footer | Ready to use offline |

A row's label is its name if one is typed, otherwise "Item N", where N is the row's current position starting at 1.

### 6.4 Behaviour rules

- **First visit:** Metric is selected and two empty rows are shown, both set to g.
- **Before Compare:** nothing is calculated or shown while the customer types.
- **After an edit:** once a result or a summary is on screen, any change to a name, price, quantity or unit, and any added or removed row, clears every unit price, badge, summary and note. A field message disappears when its own field is edited.
- **Switching system:** when a result is on screen, switching between Metric and Imperial re-shows the same result in the other system at once, with the decimals chosen again. It never changes which row is cheapest and never changes a row's typed unit.
- **Add item:** adds an empty row after the last one, with the same unit as the row above it.
- **Remove:** with three or more rows it deletes the row. With exactly two rows it empties the row's name, price and quantity and keeps the row.
- **Clear all:** returns to two empty rows set to g (Metric) or oz (Imperial), and clears every result and message. It keeps the Metric or Imperial choice. It asks for no confirmation.

### 6.5 Remembering

- Save to localStorage under the key `unit-price:v1`, as JSON in this shape:

```
{"system": "metric", "items": [{"name": "", "price": "", "quantity": "", "unit": "g"}]}
```

- "system" is "metric" or "imperial". Prices and quantities are saved exactly as typed.
- Save after every change. Load once when the app starts.
- If nothing is saved, or the saved data cannot be read, start with the first-visit state.
- Put every localStorage call inside try/catch. If storage is not available, the app works normally and simply does not remember.
- Results, summaries, notes and field messages are never saved.

### 6.6 Accessibility

- Every input and select has a visible label or an aria-label that includes the row label, for example "Price, Item 2".
- A field message is linked to its field with aria-describedby, and the field gets aria-invalid="true".
- The summary area is an aria-live="polite" region.
- The cheapest row is marked with the word "Cheapest" or "Tie", never by colour alone.
- Every control is a real button, input or select element and can be reached and used with a keyboard.
- The page has `<html lang="en">`.

### 6.7 Look

- System font stack. No web fonts. No images other than the icons.
- Light theme only.
- Colours, defined once as CSS custom properties:

| Property | Value | Used for |
|---|---|---|
| --bg | #FFFFFF | page and cards |
| --text | #1A1A1A | text |
| --muted | #5F6368 | secondary text |
| --primary | #0B6E4F | Compare button background with white text, badges |
| --tint | #E6F4EA | background of the cheapest row |
| --error | #B00020 | field messages and the border of a field with a message |
| --border | #8A8A8A | field and card borders |

Each text pair above has a contrast ratio of at least 4.5:1, and the border colour at least 3:1 on white.

## 7. Offline and home screen

### 7.1 Service worker (sw.js)

- Register it from app.js with `navigator.serviceWorker.register('./sw.js')`, only when the browser supports service workers. If registration fails, the app keeps working online: log the error and show nothing to the customer.
- Use one cache with a versioned name: `unit-price-v1`.
- **Install:** add to the cache `./`, index.html, styles.css, app.js, calc.js, manifest.webmanifest and the three icon files. Do this inside event.waitUntil, so the service worker only becomes active once every file is cached. Then call skipWaiting().
- **Activate:** delete every cache whose name is not the current one. Then call clients.claim().
- **Fetch:** answer same-origin GET requests from the cache first, ignoring the query string, and fall back to the network. If a page navigation is not in the cache, answer with the cached index.html.
- Every URL in the project is relative (`./...`), so the app works from a sub-folder of a site.
- A service worker needs HTTPS, or http://localhost while testing (reference P2). Do not open index.html straight from the file system.
- In a later release, when any cached file changes, change the cache name (v2, v3 and so on). Say so in README.md.

### 7.2 "Ready to use offline"

Show the footer line when `navigator.serviceWorker.ready` resolves. In every other case show nothing.

### 7.3 Manifest and page links

manifest.webmanifest contains: name "Unit Price", short_name "Unit Price", start_url "./", scope "./", display "standalone", background_color "#FFFFFF", theme_color "#0B6E4F", and the 192 px and 512 px icons.

index.html contains: a link to the manifest, `<link rel="icon" href="icons/icon-192.png">`, `<link rel="apple-touch-icon" href="icons/apple-touch-icon.png">` and `<meta name="theme-color" content="#0B6E4F">`.

### 7.4 Icons

- Draw one simple, original icon: a solid #0B6E4F square with a white dollar sign in the centre, with enough margin to survive a rounded or circular mask. Use no third-party artwork.
- Deliver icons/icon-192.png (192 x 192), icons/icon-512.png (512 x 512) and icons/apple-touch-icon.png (180 x 180).
- Produce the PNG files with whatever is available on the build machine, using a throwaway script if needed. Leave neither the script nor any dependency in the project.
- If PNG files cannot be produced, deliver icons/icon.svg instead, point the manifest and the page links at it, list the missing PNG files under "Left to do" in README.md, and continue.

### 7.5 iPhone notes

Put these in README.md.

- In Safari on iPhone, a site's stored data, including its service worker and offline copy, can be deleted after seven days of Safari use in which the customer did not interact with the site. Web apps added to the home screen are exempt (reference P1).
- A home-screen app keeps its data separate from Safari. After adding the app to the home screen, the customer must open it once from the icon while online (reference P1).
- WebKit published these rules in 2020. Check that they still apply before relying on them.

## 8. User stories

Epics:

- **E1** - Find the cheapest of two or more items (stories 1 to 5 and 9 to 12)
- **E2** - See results in the customer's preferred measure unit (stories 6 and 7)
- **E3** - Use the app in the store with no internet (stories 8 and 13)

### Story 1: Enter an item's price and quantity

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item entry

As a supermarket customer, I want to enter the price and the quantity (with its measure unit) of an item, so that the app has what it needs to compare it.

**Acceptance criteria**

1. Given nothing is saved on the phone, when the app opens, then two empty rows are shown, each with a price field, a quantity field and a unit selector set to g.
2. Given the customer opens a unit selector, when the options are shown, then they are g, oz, ml, fl oz and pieces, in that order, and nothing else.
3. Given the customer taps a price or quantity field on a phone, when the keyboard opens, then it is a numeric keypad with a decimal key.
4. Given the customer types 4,99 in a price field, when Compare is tapped, then it is read as 4.99.

**Source:** Brief: goal; Brief: mandatory fields; Brief: lower measures; Follow-up: mobile first; D1; D2; F17

### Story 2: Compare two or more items together

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item entry

As a supermarket customer, I want to enter two or more items in the same comparison, so that I can compare all the options in front of me.

**Acceptance criteria**

1. Given the item list is shown, when the customer taps "Add item", then a new empty row is added after the last row, with the same unit as the row above it.
2. Given any number of rows, when the customer taps "Add item", then a row is added. There is no maximum.
3. Given exactly one complete row, when Compare is tapped, then that row's unit price is shown, no row has a badge, and the summary reads "Add another item to compare."
4. Given no complete row, when Compare is tapped, then the summary reads "Enter a price and quantity for at least two items."

**Source:** Brief: goal; D3; F5; F6

### Story 3: Be told when a price or quantity is missing

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item entry

As a supermarket customer, I want to be told when an item is missing its price or its quantity, so that I never get a result based on incomplete information.

**Acceptance criteria**

1. Given a started row with an empty price, when Compare is tapped, then "Enter a price" is shown under the price field.
2. Given a started row with an empty quantity, when Compare is tapped, then "Enter a quantity" is shown under the quantity field.
3. Given a price or quantity that is not a number by section 5.2 (for example abc, 1.2.3 or -1), when Compare is tapped, then "Use numbers only, like 4.99" is shown under that field.
4. Given a price or quantity of 0, when Compare is tapped, then "Must be more than 0" is shown under that field.
5. Given a quantity in pieces that is not a whole number, when Compare is tapped, then "Use a whole number of pieces" is shown under the quantity field.
6. Given one row has a field message and two other rows are complete, when Compare is tapped, then the complete rows are compared, the row with the message gets no unit price, and the note "1 item left out. Check the highlighted fields." is shown.
7. Given a row whose name, price and quantity are all empty, when Compare is tapped, then the row is ignored and shows no message.
8. Given a field shows a message, when the customer edits that field, then the message disappears.

**Source:** Brief: mandatory fields; D4; F2; F3

### Story 4: See each item's unit price

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Unit price comparison

As a supermarket customer, I want to see each item's price for the same standard amount (for example per 100 g), so that I can compare packages of different sizes fairly.

**Acceptance criteria**

1. Given Metric is selected and a row holds 4.00 for 500 g, when Compare is tapped, then the row shows "$0.80 per 100 g".
2. Given Metric is selected and a row holds 2.49 for 750 ml, when Compare is tapped, then the row shows "$0.33 per 100 ml".
3. Given Imperial is selected, when Compare is tapped, then weight rows show a price "per oz" and volume rows a price "per fl oz".
4. Given a row holds 3.99 for 12 pieces, when Compare is tapped, then the row shows "$0.33 per piece" in either system.
5. Given two rows whose unit prices differ but would look the same with 2 decimals, when Compare is tapped, then every row in that result shows 3 decimals, or 4 if 3 are still not enough (examples C9 and C5).
6. Given any result, then every unit price starts with a dollar sign and uses a decimal point.

**Source:** Brief: comparison basis; Brief: preferred unit; D2; D6; D9; D15

### Story 5: See which item is the cheapest

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Unit price comparison

As a supermarket customer, I want the app to tell me which item is the cheapest, so that I can choose the best value without working it out myself.

**Acceptance criteria**

1. Given the customer is typing prices and quantities, when Compare has not been tapped, then no unit price, badge or summary is shown.
2. Given two or more complete rows of one kind, when Compare is tapped, then the row with the lowest unit price shows the badge "Cheapest" and the summary reads "Cheapest: [label] at [unit price]."
3. Given 4.00 for 500 g and 3.00 for 300 g with Metric selected, when Compare is tapped, then the summary reads "Cheapest: Item 1 at $0.80 per 100 g."
4. Given two or more rows share the lowest unit price, when Compare is tapped, then each of them shows the badge "Tie" and the summary reads "Tie for cheapest: [labels] at [unit price]." (example C6).
5. Given the complete rows are not all one kind (weight, volume or pieces), when Compare is tapped, then no unit price or badge is shown and the summary reads "Compare one kind at a time: weight (g, oz), volume (ml, fl oz) or pieces."
6. Given a result is on screen, when the customer changes a name, price, quantity or unit, or adds or removes a row, then every unit price, badge, summary and note disappears until Compare is tapped again.
7. Given 0.30 for 3 g and 0.10 for 1 g, when Compare is tapped, then the two rows are shown as a tie, because the cheapest is decided on exact values with the tolerance in section 5.6.

**Source:** Brief: goal; Brief: comparison basis; D3; D8; F4; F8; F10

### Story 6: Choose Metric or Imperial results

- **Epic:** E2 - See results in the customer's preferred measure unit
- **Feature:** Preferred measure unit

As a supermarket customer, I want to choose my preferred measure unit, so that all item results are shown in the measurement system I am used to.

**Acceptance criteria**

1. Given the app opens for the first time, then the control "Show results in" offers Metric and Imperial, with Metric selected.
2. Given Metric is selected, when a result is shown, then weight rows read "per 100 g" and volume rows read "per 100 ml", whatever unit each row was typed in.
3. Given Imperial is selected, when a result is shown, then weight rows read "per oz" and volume rows read "per fl oz", whatever unit each row was typed in.
4. Given a result is on screen, when the customer switches system, then the same result is shown in the other system at once, without a tap on Compare, and the cheapest row does not change.
5. Given the customer selected Imperial, when the app is closed and opened again, then Imperial is still selected.
6. Given rows in pieces, when the system is switched, then they still read "per piece".
7. Given the customer switches system, then no row's typed unit changes.

**Source:** Brief: preferred unit; D5; D6; F7; F8; F15

### Story 7: Compare items entered in different units

- **Epic:** E2 - See results in the customer's preferred measure unit
- **Feature:** Unit converter

As a supermarket customer, I want to enter one item in grams and another in ounces and still get a comparison, so that I can compare products labelled in different measurement systems.

**Acceptance criteria**

1. Given 3.79 for 340 g and 3.99 for 12 oz, when Compare is tapped, then Item 1 is the cheapest, and the rows show "$1.11 per 100 g" and "$1.17 per 100 g" in Metric, or "$0.32 per oz" and "$0.33 per oz" in Imperial.
2. Given 1.29 for 500 ml and 1.29 for 16.9 fl oz with Metric selected, when Compare is tapped, then Item 1 is the cheapest, and the rows show "$0.2580 per 100 ml" and "$0.2581 per 100 ml".
3. Given a weight row in oz, then 1 oz is counted as 28.349523125 g. Given a volume row in fl oz, then 1 fl oz is counted as 29.5735295625 ml.
4. Given rows of different kinds, then nothing is converted between weight, volume and pieces (story 5, criterion 5).

**Source:** Brief: nice to have; D1; D7; F1

### Story 8: Use the app with no internet after the first load

- **Epic:** E3 - Use the app in the store with no internet
- **Feature:** Offline availability

As a supermarket customer, I want the page to keep working with no internet once I have loaded it, so that I can use it at any time in the supermarket.

**Acceptance criteria**

1. Given the customer opened the app once with a connection and "Ready to use offline" was shown, when they open it again with no connection, then it opens and every feature works.
2. Given the phone has no connection, when the customer enters items and taps Compare, then the result is shown.
3. Given the app is in use, then it requests nothing from any address other than its own files.
4. Given the app is served from a sub-folder of a site, when it is opened, then it loads and works offline in the same way.

**Source:** Brief: offline; Brief: technology; D11; F11

### Story 9: Name an item

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item entry

As a supermarket customer, I want to give an item an optional name, so that I can recognise it in the result.

**Acceptance criteria**

1. Given a row, then it has a name field that accepts up to 30 characters and shows "Item N" as its placeholder, where N is the row's position.
2. Given a row with a name, when a result is shown, then the summary uses the name. Given a row without a name, then the summary uses "Item N".
3. Given a row with a valid price and quantity and no name, when Compare is tapped, then the row is complete.
4. Given a row with only a name, when Compare is tapped, then "Enter a price" and "Enter a quantity" are shown on it.

**Source:** D10; F3; F14

### Story 10: Remove an item

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item list

As a supermarket customer, I want to remove an item, so that I can undo a mistake or drop an item I put back on the shelf.

**Acceptance criteria**

1. Given a row, then it has a "Remove" button whose accessible name includes the row label.
2. Given three or more rows, when Remove is tapped on a row, then that row disappears and the rows after it are renumbered.
3. Given exactly two rows, when Remove is tapped on a row, then its name, price and quantity are emptied and the row stays.

**Source:** D10; F5; F16

### Story 11: Clear all and start again

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item list

As a supermarket customer, I want to clear everything in one tap, so that I can start a new comparison at the next shelf.

**Acceptance criteria**

1. Given any rows, results and messages, when "Clear all" is tapped, then the list goes back to two empty rows, and every unit price, badge, summary, note and field message disappears.
2. Given Clear all was tapped, then the Metric or Imperial choice is unchanged, and both rows are set to g (Metric) or oz (Imperial).
3. Given Clear all is tapped, then no confirmation is asked.

**Source:** D10; F6; F9

### Story 12: Find my items still there when I come back

- **Epic:** E1 - Find the cheapest of two or more items
- **Feature:** Item list

As a supermarket customer, I want the items I typed to still be there when I reopen the app, so that I do not lose them when I switch to another app in the store.

**Acceptance criteria**

1. Given rows with names, prices, quantities and units, when the app is closed and opened again on the same phone, then the same rows are shown with the same content.
2. Given the app is reopened, then no unit price or summary is shown until Compare is tapped.
3. Given the phone does not let the app store data, when the app is used, then everything works except remembering, and no error is shown.
4. Given Clear all was tapped, when the app is reopened, then it shows two empty rows.

**Source:** D10; F15

### Story 13: Add the app to my home screen

- **Epic:** E3 - Use the app in the store with no internet
- **Feature:** Offline availability

As a supermarket customer, I want to add the app to my phone's home screen, so that I can open it like any other app, with or without a connection.

**Acceptance criteria**

1. Given the app is open in the phone's browser, when the customer uses the browser's own "Add to Home Screen" or "Install" option, then an icon named "Unit Price" is added.
2. Given the customer opens the app from that icon, then it opens full screen, without the browser's address bar.
3. Given the customer opened it from the icon once with a connection, when they open it from the icon with no connection, then it works.

**Source:** D11; D13

## 9. Files to deliver

Create the project in a folder named `unit-price` next to this file. It contains exactly these files:

| File | Purpose |
|---|---|
| index.html | The page. Loads app.js with `<script type="module">`. |
| styles.css | All styles. |
| calc.js | Sections 5.1 to 5.8 as pure functions. No access to the page, to storage or to the network. |
| app.js | The screen, storage and service worker registration. Imports calc.js. |
| sw.js | The service worker (section 7.1). |
| manifest.webmanifest | The manifest (section 7.3). |
| icons/icon-192.png, icons/icon-512.png, icons/apple-touch-icon.png | The icons (section 7.4). |
| Test.md | Manual test cases (section 11). |
| README.md | How to run, test and publish (section 12). |

## 10. Build order

| Step | Build | Check before the next step |
|---|---|---|
| 1 | calc.js | All 20 examples in section 5.9 give the listed result in both systems. Use a throwaway script. |
| 2 | index.html, styles.css and the item list in app.js: sections 6.1, 6.2, 6.3 and 6.7, stories 1, 2 and 9 | At 320 px wide there is no horizontal scrolling, and rows can be added. |
| 3 | The Compare flow and its messages: stories 3, 4, 5 and 7 | Examples C1, C10, C11, C13 and C15 behave on the page as listed. |
| 4 | The Metric and Imperial switch: story 6 | Examples C4 and C5 re-show correctly when the system is switched. |
| 5 | Remove, Clear all and remembering: stories 10, 11 and 12, section 6.5 | A reload keeps the items and the system. Clear all resets the list. |
| 6 | Offline and home screen: stories 8 and 13, section 7 | Served from http://localhost, the page reloads and compares with the network switched off. |
| 7 | Accessibility pass (section 6.6), then Test.md and README.md | Section 13. |

## 11. Test.md

Test.md is a manual test script (decision D12). Write no automated tests.

Format: one table per area, with the columns ID, Steps, Expected result and Pass/Fail. Leave Pass/Fail empty. Write steps a person can follow on a phone without reading this file.

It must cover at least the following, and any other edge case found during the build.

| Area | Cases |
|---|---|
| T-CALC | Every example C1 to C20, each one in Metric and in Imperial. |
| T-INPUT | Empty price. Empty quantity. Letters. Two separators. A minus sign. Zero. A comma as decimal. "4." and ".5". Spaces around a number. The 9-character limit. A decimal quantity in pieces. |
| T-ROWS | An untouched row is ignored. A row with only a name is flagged. Add item copies the unit above. Many rows can be added. Remove with three rows. Remove with two rows. Clear all. |
| T-COMPARE | Nothing is shown before Compare. The result is cleared by each kind of edit. One complete row. No complete row. Mixed kinds, for each pair of kinds. The left-out note for one row and for several. |
| T-SYSTEM | Switching with a result on screen. The choice is remembered. Pieces are unchanged by the switch. Typed units are unchanged by the switch. |
| T-DECIMALS | A result with 2 decimals (C1), with 3 (C9) and with 4 (C5 in Metric). Two rows that still look the same at 4 decimals (C5 in Imperial). |
| T-NAMES | Named and unnamed rows in the summary. The 30-character limit. A tie summary with two names and with three. |
| T-MEMORY | A reload keeps the items. Results are not restored. Storage blocked, for example in private browsing. Clear all followed by a reload. |
| T-OFFLINE | First load, then a reload with no connection. Compare with no connection. "Ready to use offline" appears. The app served from a sub-folder. A reload after a cache name change. |
| T-HOME | Add to the home screen on iPhone and on Android. The app opens full screen. First open from the icon with a connection, then without. |
| T-MOBILE | A screen 320 px wide. The numeric keypad. Landscape. A screen 600 px wide or more. Tap targets of 44 px. |
| T-A11Y | A screen reader reads every label and the summary. The whole flow works with a keyboard only. |

End Test.md with a list of the cases that could not be checked on the build machine, for example anything that needs a real phone.

## 12. README.md

README.md contains:

1. What the app does, in two or three sentences.
2. How to run it locally: serve the folder with any static file server, for example `python3 -m http.server 8000`, and open http://localhost:8000. Say that opening index.html as a file does not work.
3. How to test it offline: load it once, switch the network off, reload.
4. How to publish it: upload the folder as it is to any static host that serves HTTPS. It also works from a sub-folder.
5. How to release an update: change the cache name in sw.js.
6. The iPhone notes from section 7.5.
7. "Assumptions made during the build": every gap filled under rule 3 of section 1, or "None".
8. "Left to do": anything that could not be finished, or "Nothing".

## 13. Definition of done

- [ ] Every acceptance criterion in section 8 is met.
- [ ] All 20 examples in section 5.9 give the listed result in both systems.
- [ ] Every text on the screen matches section 6.3.
- [ ] The project holds exactly the files in section 9.
- [ ] There is no framework, library, CDN, web font, package file or build output, and no request to any other origin.
- [ ] At 320 px wide there is no horizontal scrolling.
- [ ] Served from http://localhost, the app reloads and compares with the network switched off.
- [ ] Items and the Metric or Imperial choice survive a reload.
- [ ] Test.md and README.md are complete (sections 11 and 12).
- [ ] No git repository was created, nothing was deployed, and every throwaway script is gone.

## 14. Out of scope

- kg, lb, litres, the imperial (UK) fluid ounce and any other unit.
- Converting between weight, volume and pieces.
- Other currencies, a currency setting, and locale number formats.
- Other languages.
- A dark theme.
- Showing how much more the other items cost, rankings or savings.
- Barcode scanning, product lookup, price history, shopping lists, accounts and sharing.
- Analytics, cookies and any server.
- Automated tests, git and deployment.

## 15. Source key

**Brief and follow-up.** Each label points to these words from the product owner.

- **Brief: technology** - "It is webpage that is intended to use HTML, CSS and JavaScript only."
- **Brief: goal** - "The goal of the app is to help customers in a supermarket. They will put the price, and measure of two or more items and the app will tell which one is the cheapest one."
- **Brief: comparison basis** - "Please compare the price of 100g (or 100 unit selected that makes more sense)."
- **Brief: mandatory fields** - "The price and quantity (g, ml, ect) is mandatory."
- **Brief: nice to have** - "Nice to have is a converter where you can add product A in grams and product B in pounds. Only do if it is easy."
- **Brief: lower measures** - "We can have just lower measures, so grams are enough, no need to add kilo for instance."
- **Brief: offline** - "The page should load once and be available for the user to use at anytime, no internet needed."
- **Brief: preferred unit** - "Add a measure unit of preference, so all items will show the result on that metric system."
- **Follow-up: mobile first** - "this is a mobile first app."

**Decisions D1 to D16.** Answered by the product owner on 2026-11-14 (section 4.1).

**Defaults F1 to F19.** Set by the spec author, not by the product owner (section 4.2).

**Platform references.** Checked on 2026-11-14.

- **P1** - WebKit, "Full Third-Party Cookie Blocking and More" (the seven-day cap on script-writable storage): https://webkit.org/?p=10218 and WebKit, "CNAME Cloaking and Bounce Tracking Defense" (home-screen web apps are exempt and keep their data separate from Safari): https://webkit.org/blog/11338/cname-cloaking-and-web-privacy/
- **P2** - MDN, "Secure contexts" (https and localhost URLs are secure contexts; plain http is not): https://developer.mozilla.org/docs/Web/Security/Secure_Contexts
- **P3** - NIST Handbook 44, Appendix C, "General Tables of Units of Measurement". Validate the factors in section 5.1 against it before release: https://www.nist.gov/system/files/documents/2017/12/07/appc-18-hb44-final.pdf
