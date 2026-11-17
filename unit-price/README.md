# Unit Price

Unit Price is a phone-first web page that tells a supermarket customer which of two or more items is the cheapest. The customer types each item's price and quantity, picks its unit (g, oz, ml, fl oz or pieces) and taps Compare; the app shows each item's price for the same standard amount (per 100 g, per 100 ml, per oz, per fl oz or per piece) and marks the cheapest. It works with no internet after the first load and can be added to the phone's home screen.

It is plain HTML, CSS and JavaScript with no libraries and no build step. Everything the customer types stays on the phone.

## Files

| File | Purpose |
|---|---|
| index.html | The page |
| styles.css | All styles |
| calc.js | The calculation rules, as pure functions |
| app.js | The screen, storage and service worker registration |
| sw.js | The service worker (offline copy) |
| manifest.webmanifest | Home-screen settings |
| icons/ | App icons (192 px, 512 px and 180 px for iPhone) |
| Test.md | Manual test cases |

## Run it locally

Serve this folder with any static file server, then open it in a browser. For example, from inside this folder:

```
python3 -m http.server 8000
```

and open http://localhost:8000.

Opening index.html straight from the file system (a `file://` address) does not work: the browser blocks the JavaScript modules and the service worker there. A service worker needs HTTPS, or http://localhost while testing.

## Test it offline

1. Open the app from http://localhost:8000 with the network on.
2. Wait until "Ready to use offline" appears at the bottom of the page.
3. Switch the network off (in the browser's developer tools choose "Offline" on the Network tab, or turn on airplane mode on a phone).
4. Reload. The app opens and Compare works.

The full manual test script is in Test.md.

## Publish it

Upload the folder as it is to any static host that serves HTTPS. Every URL in the project is relative, so it also works from a sub-folder of a site, for example `https://example.com/tools/unit-price/`. There is nothing to build or install.

## Release an update

When any file changes, change the cache name at the top of sw.js before publishing:

```
const CACHE_NAME = 'unit-price-v1';   // change to 'unit-price-v2', then v3, and so on
```

Without a new cache name, phones keep serving the old files from their offline copy. With it, the new service worker saves the new files and deletes the old cache.

## iPhone notes

- In Safari on iPhone, a site's stored data, including its service worker and offline copy, can be deleted after seven days of Safari use in which the customer did not interact with the site. Web apps added to the home screen are exempt (WebKit, "Full Third-Party Cookie Blocking and More": https://webkit.org/?p=10218).
- A home-screen app keeps its data separate from Safari. After adding the app to the home screen, the customer must open it once from the icon while online (WebKit, "CNAME Cloaking and Bounce Tracking Defense": https://webkit.org/blog/11338/cname-cloaking-and-web-privacy/).
- WebKit published these rules in 2020. Check that they still apply before relying on them.

## Assumptions made during the build

1. **Accessible names.** Each field keeps its visible label from the spec ("Price ($)" and so on) and gets an aria-label that adds the row label, for example "Price ($), Item 2" and "Remove Item 2". Each card is a group named after its row label. No visible card heading is shown, because section 6.3 lists no such text.
2. **Metric or Imperial control.** Built as two radio buttons ("Show results in" is the group's legend), styled as a two-part switch. Each option is 44 px high.
3. **A name of only spaces.** The row counts as started (F3: "any text"), but its label falls back to "Item N". Names are trimmed when used as labels.
4. **Field messages after other edits.** A field message only disappears when its own field is edited (section 6.4), so for example "Use a whole number of pieces" stays if only the unit is changed. The next tap on Compare checks every row again and replaces all messages.
5. **Remove with two rows** keeps the row's unit; only name, price and quantity are emptied.
6. **Focus after Remove** moves to the next row's Remove button (three or more rows) or to the emptied row's name field (two rows), so keyboard and screen reader users do not lose their place.
7. **Saved data.** If the saved data has the wrong shape or an unknown unit, the app starts in the first-visit state. If it holds fewer than two rows, empty rows are added (copying the unit above). Saved text longer than the field limits is cut to the limit.
8. **Tie summary price.** The summary shows the unit price text of the first tied row. Tied rows are equal within the tolerance of section 5.6, so they show the same text.
9. **Manifest icons** are marked `"purpose": "any maskable"`. The dollar sign sits well inside the central safe zone, so it survives a circular mask.
10. **Navigation fallback.** For a page navigation that is not in the cache, the service worker answers with the cached index.html before trying the network, as section 7.1 says.
11. **Icons** were drawn from geometric shapes (two arcs and a line), not from a font, so they contain no third-party artwork.

## Left to do

Nothing in the build. Checks that need a real phone, Safari, Firefox or a published HTTPS site are listed at the end of Test.md.
