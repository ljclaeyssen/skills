---
name: wowhead-tooltips
description: "Integrate Wowhead item tooltips, icons, and colored links into any web project (Angular, React, Vue, vanilla). Use this skill whenever the user wants to add Wowhead tooltips, display WoW item icons, link to Wowhead items, or integrate the Wowhead power script. Also triggers for: 'wowhead hover', 'item tooltip', 'wow item icon', 'wowhead script', 'data-wowhead', 'iconizeLinks', 'whTooltips', or any mention of displaying World of Warcraft item information with hover tooltips on a website."
---

# Wowhead Tooltips Integration

This skill captures battle-tested patterns for integrating Wowhead's tooltip system into web applications, including the exact DOM structure Wowhead injects, CSS overrides needed, and SPA-specific refresh patterns.

## How the Wowhead Script Works

Wowhead provides a free JS script that auto-detects links to `wowhead.com/item=XXXXX` and enriches them with:
- **Tooltip on hover** — full item card (stats, effects, set bonuses)
- **Inline icon** — item icon injected as a `<span>` inside the link
- **Colored link text** — colored by item quality (green, blue, purple, orange)

## Step 1: Add the Script

Add these two script tags in `<head>`, **before** any app scripts:

```html
<script>const whTooltips = {colorLinks: true, iconizeLinks: true, iconSize: 'small', renameLinks: false};</script>
<script src="https://wow.zamimg.com/js/tooltips.js"></script>
```

### Config Options

| Option | Type | Description |
|--------|------|-------------|
| `colorLinks` | boolean | Color links by item quality (q0-q5) |
| `iconizeLinks` | boolean | Add item icon next to link |
| `iconSize` | `'tiny'`\|`'small'`\|`'medium'`\|`'large'` | Icon size. **Use `'small'` to get the `<span class="iconsmall">` structure**. `'tiny'` falls back to background-image on the `<a>` itself, which is harder to style. |
| `renameLinks` | boolean | Replace link text with item name. Set `false` to keep custom labels. |
| `hide` | object | Hide tooltip sections: `{droppedby, sellprice, ilvl, extra}` |

## Step 2: Wowhead URL Format

Standard link format (the script auto-detects these):

```html
<a href="https://www.wowhead.com/item=12345">My Item</a>
```

With bonus IDs (for upgrade tracks, difficulty variants):
```html
<a href="https://www.wowhead.com/item=12345?bonus=1234:5678">My Item</a>
```

With locale:
```html
<a href="https://www.wowhead.com/fr/item=12345">Mon Item</a>
```

### Per-link Attributes

```html
<a href="..." data-wowhead="ench=1234&gems=5678:9012&bonus=1234:5678">Item</a>
<a href="..." data-wh-icon-size="large">Item with large icon</a>
<a href="..." data-wh-rename-link="false">Keep my custom text</a>
```

## Step 3: Understand the Injected DOM

This is critical. After the script processes a link, the DOM becomes:

```html
<a class="q4" data-wh-icon-added="true" href="https://www.wowhead.com/item=12345">
  <span class="iconsmall" data-env="live" data-tree="live" data-game="wow" data-type="item">
    <ins style="background-image: url('https://wow.zamimg.com/images/wow/icons/small/inv_xxx.jpg')"></ins>
    <del></del>
  </span>
  <span>Item Name</span>
</a>
```

Key points:
- `q4` class = Epic quality color (q0=Poor, q1=Common, q2=Uncommon, q3=Rare, q4=Epic, q5=Legendary)
- `<span class="iconsmall">` is a 26x26 inline-block container
- `<ins>` inside carries the icon as `background-image` (18x18, `background-size: contain`)
- `<del>` is empty, used internally by Wowhead
- The original text gets wrapped in a second `<span>`
- `data-wh-icon-added="true"` marks processed links

**This structure only appears with `iconSize: 'small'` or larger.** With `iconSize: 'tiny'` (default), Wowhead adds a `background-image` directly on the `<a>` tag with `padding-left` — much harder to style.

## Step 4: CSS Overrides

Wowhead injects inline styles and adds classes. You'll likely need overrides.

### Show icon inline (default behavior, just needs room)

```css
/* Let the icon span sit naturally in a flex container */
.my-item-link {
  display: inline-flex;
  align-items: center;
  gap: 2px;
}

.my-item-link .iconsmall del {
  display: none !important;
}
```

### Hide the icon on specific elements

```css
/* For small elements where icons don't fit */
a.small-cell .iconsmall {
  display: none !important;
}

a.small-cell {
  background-image: none !important;
  padding-left: 0 !important;
}
```

### Prevent Wowhead from overriding your colors

Wowhead adds quality color classes (`q0`-`q5`) that override `color`. If you have custom colors:

```css
a.my-custom-link,
a.my-custom-link:hover {
  color: inherit !important;
  background-image: none !important;
  padding-left: 0 !important;
}
```

### Icon URL pattern (for manual usage)

```
https://wow.zamimg.com/images/wow/icons/{size}/{iconname}.jpg
```
Sizes: `tiny` (18px), `small` (36px), `medium` (56px), `large` (56px+)

## Step 5: SPA Refresh (Angular, React, Vue)

The script scans the DOM on load, but SPAs render content dynamically. You must call `$WowheadPower.refreshLinks()` when new Wowhead links appear in the DOM.

### Angular Directive (MutationObserver-based)

```typescript
import { Directive, ElementRef, AfterViewInit, OnDestroy } from '@angular/core';

declare global {
  interface Window {
    $WowheadPower?: { refreshLinks: () => void };
  }
}

@Directive({
  selector: '[appWowheadTooltips]',
  standalone: true,
})
export class WowheadTooltipsDirective implements AfterViewInit, OnDestroy {
  private timer: ReturnType<typeof setTimeout> | null = null;
  private observer?: MutationObserver;

  constructor(private el: ElementRef<HTMLElement>) {}

  ngAfterViewInit(): void {
    this.observer = new MutationObserver(() => this.scheduleRefresh());
    this.observer.observe(this.el.nativeElement, { childList: true, subtree: true });
    this.scheduleRefresh();
  }

  ngOnDestroy(): void {
    this.observer?.disconnect();
    if (this.timer) clearTimeout(this.timer);
  }

  private scheduleRefresh(): void {
    if (this.timer) clearTimeout(this.timer);
    this.timer = setTimeout(() => {
      window.$WowheadPower?.refreshLinks();
    }, 100);
  }
}
```

Usage: `<div appWowheadTooltips>` on any container with Wowhead links.

**Why MutationObserver and not ngAfterViewChecked?** `ngAfterViewChecked` fires on every change detection cycle. With zone.js, that's every mouse move, scroll, click. With 20+ instances (e.g., a roster table with one per row), that's hundreds of unnecessary DOM reads per second. MutationObserver fires only when the DOM actually changes.

### React Hook

```tsx
import { useEffect, useRef } from 'react';

function useWowheadTooltips(deps: any[] = []) {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    let timer: ReturnType<typeof setTimeout>;
    const observer = new MutationObserver(() => {
      clearTimeout(timer);
      timer = setTimeout(() => {
        (window as any).$WowheadPower?.refreshLinks();
      }, 100);
    });

    if (ref.current) {
      observer.observe(ref.current, { childList: true, subtree: true });
      timer = setTimeout(() => (window as any).$WowheadPower?.refreshLinks(), 100);
    }

    return () => {
      observer.disconnect();
      clearTimeout(timer);
    };
  }, deps);

  return ref;
}

// Usage: const ref = useWowheadTooltips(); <div ref={ref}>...</div>
```

### Vue 3 Composable

```typescript
import { onMounted, onUnmounted, ref, type Ref } from 'vue';

export function useWowheadTooltips(): Ref<HTMLElement | null> {
  const el = ref<HTMLElement | null>(null);
  let observer: MutationObserver;
  let timer: ReturnType<typeof setTimeout>;

  onMounted(() => {
    if (!el.value) return;
    observer = new MutationObserver(() => {
      clearTimeout(timer);
      timer = setTimeout(() => (window as any).$WowheadPower?.refreshLinks(), 100);
    });
    observer.observe(el.value, { childList: true, subtree: true });
    timer = setTimeout(() => (window as any).$WowheadPower?.refreshLinks(), 100);
  });

  onUnmounted(() => {
    observer?.disconnect();
    clearTimeout(timer);
  });

  return el;
}

// Usage: const el = useWowheadTooltips(); <div ref="el">...</div>
```

## Step 6: Angular View Encapsulation Gotcha

Wowhead injects DOM elements dynamically — they don't have Angular's `_ngcontent-xxx` attributes. Component-scoped styles (the default `ViewEncapsulation.Emulated`) **will not match** Wowhead-injected elements like `.iconsmall`, `ins`, `del`.

**Solution:** Put all Wowhead CSS overrides in a **global stylesheet** (e.g., `styles.scss`), not in component stylesheets.

```scss
/* styles.scss — global, not component-scoped */

/* Hide icon on small gear cells */
a.gear-cell .iconsmall {
  display: none !important;
}

/* Style icon in loot chips */
.loot-chip .iconsmall {
  flex-shrink: 0;
  margin: 0 2px 0 0 !important;
}

.loot-chip .iconsmall del {
  display: none !important;
}
```

This applies to any framework with style scoping (Angular, Vue scoped styles, Shadow DOM).

## Common Pitfalls

1. **`iconSize: 'tiny'` (default) uses background-image on `<a>`, not the `<span>` structure.** Always set `iconSize: 'small'` or larger to get the controllable `<span class="iconsmall">` DOM.

2. **`renameLinks: true` replaces your custom link text.** Set `false` if you want to keep your own labels.

3. **Styles in component files don't reach Wowhead elements.** Use global CSS.

4. **`refreshLinks()` is global** — it rescans the entire document, not just the container. With debounce, this is fine, but be aware if you have many directive instances.

5. **Wowhead adds `padding-left` and `background-image` inline on `<a>` tags.** Even with `iconSize: 'small'`, you may need `background-image: none !important; padding-left: 0 !important;` on links where you don't want the old-style icon.

6. **The `<del>` element is empty but takes space.** Hide it with `display: none !important` in your global styles.
