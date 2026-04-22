# svg-shadow-scope

A [minimal reproduction](https://Serrulien.github.io/svg-shadow-scope/svg-shadow-dom-test.html) showing how Shadow DOM can be used to isolate duplicate IDs when the same SVG is rendered multiple times on a page.

## The problem

Inline SVGs commonly reference internal paint servers (gradients, patterns, filters, masks, clip paths) by `id`:

```html
<linearGradient id="grad1">...</linearGradient>
<circle fill="url(#grad1)" />
```

If the same SVG markup is injected into the page more than once, every copy ends up with the same `id`. `getElementById` returns the first match, so all `url(#grad1)` references resolve to the first instance — and some browsers historically lost the reference entirely if that first instance lived inside a `display: none` subtree.

## The fix

Attach each SVG to its own shadow root. Shadow DOM scopes IDs to the root, so `url(#grad1)` inside a shadow tree resolves only within that tree and no cross-instance collision is possible.

```js
const host = document.querySelector('.svg-host');
host.attachShadow({ mode: 'open' }).innerHTML = svgMarkup;
```

## Running it

Open [`svg-shadow-dom-test.html`](https://Serrulien.github.io/svg-shadow-scope/svg-shadow-dom-test.html) in any modern browser. No build step, no dependencies.

- **Toggle First SVG** — hides/shows SVG #1 and confirms SVG #2 keeps its gradient.
- **Switch to Light DOM** — re-injects both SVGs directly (no shadow root) so you can compare behavior.

## Files

- [`svg-shadow-dom-test.html`](./svg-shadow-dom-test.html) — the demo (single file, vanilla JS + CSS).

## Notes

The Light DOM reproduction was most visible in older Chrome versions. Current Chrome, Firefox, and Safari generally resolve both references correctly even with duplicate IDs, so the two modes may look identical in an up-to-date browser. The Shadow DOM approach is still the right pattern — it makes correctness independent of the browser's current behavior.
