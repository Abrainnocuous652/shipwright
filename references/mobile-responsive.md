# Mobile Responsiveness & Cross-Device QA Checklist

## Breakpoint Verification
- Test every page and component at: **320px** (small phones), **375px** (standard phones), **390px** (modern iPhones), **428px** (large phones), **768px** (tablets portrait), **1024px** (tablets landscape / small laptops), **1280px** (laptops), **1440px** (desktops), **1920px** (large screens).
- At every breakpoint: no horizontal overflow, no cut-off content, no overlapping elements, no text truncated without indication.
- Resize continuously from 320px to 1920px — layouts should transition smoothly with no breakage at any intermediate width.

## Layout & Navigation (Mobile)
- Navigation collapses properly: hamburger menu, slide-out drawer, or bottom tab bar. Test open/close, backdrop dismiss, scroll behavior while nav is open.
- Mobile nav includes all routes from desktop nav — nothing hidden.
- Sidebar collapses to drawer on mobile or is replaced by appropriate mobile pattern. Must not simply shrink to unusable size.
- Fixed/sticky elements (headers, CTAs, floating buttons) don't overlap content or each other. Test with keyboard open — fixed elements must not cover active input.
- Modals and drawers are full-screen or near-full-screen on mobile (not tiny centered box). Must be scrollable if content exceeds viewport.
- Dropdowns and popovers reposition or become bottom sheets on mobile.

## Touch & Interaction
- All interactive elements: minimum touch target **44×44px**. If visual element is smaller, expand tappable area with padding.
- Minimum **8px gap** between adjacent tappable items.
- Replace hover-dependent interactions with tap equivalents on touch devices.
- Swipe gestures don't conflict with browser back-swipe or scroll.
- Long-press doesn't trigger unintended browser context menus.
- Keyboard type matches input (`inputmode="numeric"` for phone/zip, `type="email"` for email). `autocomplete` attributes set.
- Input focus scrolls field into view above keyboard.

## Typography & Readability
- Body text **≥ 16px** on mobile (prevents iOS auto-zoom on input focus).
- Line length ≤ ~80 characters (minimum 16px horizontal margins).
- Headings scale down appropriately — no awkward single-word lines.

## Images, Media & Tables
- All images responsive (`max-width: 100%`, `height: auto`), no overflow.
- `srcset` / responsive images served — don't send 2000px images to 375px screens.
- Images lazy-loaded.
- Data tables have mobile strategy: horizontal scroll + sticky first column, card layout, or collapsible rows. Never just shrink to unreadable.
- Charts legible on mobile — simplified mobile version or pinch-to-zoom if needed.

## Mobile-Specific UI Patterns
- Bottom sheets have drag handles, snap points, dismiss behavior.
- Toast notifications properly positioned and dismissible.
- Loading states appropriately sized for mobile.
- Empty states have mobile-appropriate sizing.
- Pagination/infinite scroll works with rapid scrolling.

## Viewport & Meta Tags
- `<meta name="viewport" content="width=device-width, initial-scale=1">` set. No `maximum-scale=1` or `user-scalable=no` (harms accessibility).
- `theme-color` meta tag set for light and dark mode.
- Safe area insets handled (`env(safe-area-inset-*)`).
- Portrait and landscape both work without breaking.

## Performance (Mobile)
- No layout shift (CLS) during load. Reserve space for dynamic content.
- Fast first meaningful paint on 3G/4G.
- Immediate touch response — no perceptible delay.
- Animations use `transform` and `opacity` only (not `width`, `height`, `top`, `left`).
- `prefers-reduced-motion` respected.

## PWA & Offline
- If PWA: manifest correct, service worker caches critical assets, install prompt works.
- If not PWA: offline/poor connectivity shows clear offline state.
- Mobile browser chrome handled: address bar show/hide doesn't cause layout jumps, `100dvh` used for viewport height.
