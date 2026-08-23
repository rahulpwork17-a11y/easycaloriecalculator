# Custom Error Pages Implementation Guide

## Overview

This guide documents the implementation of custom 404 (Not Found) and 500 (Internal Server Error) pages for the Easy Calorie Calculator Astro application.

## Files Created

1. **`src/pages/404.astro`** - Custom 404 Not Found page
2. **`src/pages/500.astro`** - Custom 500 Internal Server Error page

## How Astro Handles Error Pages

Astro automatically recognizes files named `404.astro` and `500.astro` in the `src/pages/` directory as error pages:

- **404.astro**: Served when a route is not found (client-side navigation or direct URL access)
- **500.astro**: Served when an unhandled server error occurs during rendering

## Page Features

### 404 Page (`src/pages/404.astro`)

- **Clear error code display** (404)
- **Friendly message** explaining the page wasn't found
- **Primary CTA**: "Go to Homepage" - returns users to the main page
- **Secondary CTA**: "Try Calorie Calculator" - directs to the main calculator
- **Support link**: "Contact Support" for further assistance
- **Consistent branding** using the existing design system (colors, typography, shadows)
- **Dark mode support** via the existing `.dark` class system
- **Entrance animations** using the existing `animate-fade-in-up` keyframe

### 500 Page (`src/pages/500.astro`)

- **Clear error code display** (500)
- **Non-technical message** that doesn't expose internal details
- **Primary CTA**: "Refresh Page" - attempts to recover from transient errors
- **Secondary CTA**: "Go to Homepage" - safe fallback
- **Support link**: "Contact Support" for persistent issues
- **Collapsible technical details** section (hidden by default)
- **SessionStorage integration** for capturing error details (for debugging)
- **Consistent branding** matching the 404 page and site design

## Design System Integration

Both pages use the existing design tokens from `src/styles/global.css`:

### Colors Used
- `--color-error` / `--color-error-soft` / `--color-error-deep` - Error state colors
- `--color-primary` / `--color-on-primary` - Primary button colors
- `--color-canvas` / `--color-canvas-soft` / `--color-canvas-soft-2` - Background colors
- `--color-ink` / `--color-body` / `--color-mute` - Text colors
- `--color-hairline` / `--color-hairline-strong` - Border colors

### Typography Utilities
- `font-display-lg` - Large display heading (48px)
- `font-display-md` - Medium display heading (32px)
- `font-body-lg` - Large body text (18px)
- `font-body-md-strong` - Medium body, semibold (16px)
- `font-body-sm` / `font-body-sm-strong` - Small body text
- `font-caption-mono` - Monospace for technical details

### Shadow System
- `shadow-level-1` - Subtle border
- `shadow-level-2` - Elevated cards/buttons

### Animations
- `animate-fade-in-up` - Staggered entrance animation

## Best Practices Implemented

### 1. User Experience
- **Non-technical language** - Avoids jargon, explains in plain terms
- **Clear next steps** - Multiple actionable options (refresh, homepage, calculator, contact)
- **Minimal cognitive load** - Single primary action, secondary alternatives
- **Brand consistency** - Matches site aesthetic, maintains trust

### 2. Accessibility
- **Semantic HTML** - Proper heading hierarchy (h1 → h2)
- **ARIA attributes** - `aria-hidden="true"` on decorative SVGs
- **Focus management** - Native button/link focus styles
- **Color contrast** - Meets WCAG AA standards in both themes
- **Reduced motion** - Animations respect `prefers-reduced-motion` (via Tailwind)

### 3. Error Handling
- **No stack traces exposed** - 500 page hides technical details by default
- **Error capture mechanism** - SessionStorage for optional error details
- **Graceful degradation** - Works without JavaScript (except refresh button)

### 4. SEO Considerations
- **Proper status codes** - Astro automatically returns 404/500 status
- **Canonical URLs** - Inherited from Layout component
- **Meta tags** - Title and description set appropriately
- **Noindex** - Consider adding `<meta name="robots" content="noindex">` to prevent indexing

## Testing Error Pages

### Local Development

1. **Test 404 page**:
   ```bash
   # Navigate to a non-existent route
   http://localhost:4321/non-existent-page
   ```

2. **Test 500 page** (requires triggering a server error):
   ```astro
   ---
   // Temporary test route - remove after testing
   throw new Error('Test 500 error');
   ---
   ```

### Production Verification

1. **404**: Visit any invalid URL on the deployed site
2. **500**: Monitor error tracking (Sentry, LogRocket, etc.) for 500 responses

### Preview Mode
```bash
npm run build && npm run preview
```
Then test at `http://localhost:4321/404` and `http://localhost:4321/500`

## Configuration Notes

### Astro Config (`astro.config.mjs`)
No additional configuration needed. Astro 7+ automatically handles:
- `404.astro` → 404 responses
- `500.astro` → 500 responses

### Middleware (Optional Enhancement)
For advanced error tracking, create `src/middleware/index.ts`:

```typescript
import { defineMiddleware } from 'astro:middleware';

export const onError = defineMiddleware(async ({ error, locals, url }, next) => {
  // Log to error tracking service
  console.error(`[${new Date().toISOString()}] Error at ${url.pathname}:`, error);
  
  // Store error details for 500 page (client-side retrieval)
  if (error instanceof Error) {
    // This would need a custom adapter for sessionStorage on server
  }
  
  return next();
});
```

### Static Hosting (Netlify, Vercel, Cloudflare Pages)

These platforms automatically serve the generated `404.html` and `500.html` files. Ensure your build output includes them (Astro does this by default).

#### Netlify
Add to `netlify.toml`:
```toml
[[redirects]]
  from = "/*"
  to = "/404.html"
  status = 404
```

#### Vercel
Automatic - no configuration needed.

#### Cloudflare Pages
Automatic - no configuration needed.

## Customization Checklist

When adapting for your brand:

- [ ] Update error messages to match brand voice
- [ ] Customize SVG icons if using custom illustrations
- [ ] Adjust CTA buttons for your primary user flows
- [ ] Add analytics events for error page views
- [ ] Consider adding a search link on 404 page
- [ ] Add "Report this error" button on 500 page (links to contact with pre-filled info)

## Analytics Integration (Optional)

Add to both pages for error tracking:

```astro
<script is:inline>
  // Track error page views
  if (typeof gtag !== 'undefined') {
    gtag('event', 'page_view', {
      page_title: document.title,
      page_location: window.location.href,
      error_code: '404' // or '500'
    });
  }
</script>
```

## Maintenance

- **Review periodically** - Check error page analytics quarterly
- **Update links** - Ensure all CTAs point to valid routes
- **Test after deployments** - Verify both pages render correctly
- **Monitor 500 frequency** - Alert if 500 errors spike

## Troubleshooting

| Issue | Solution |
|-------|----------|
| 404 page not showing | Ensure `src/pages/404.astro` exists and builds to `dist/404.html` |
| 500 page not showing | Check Astro dev server logs; 500 pages only show in production/preview |
| Styles not applying | Verify `@import '../styles/global.css'` in Layout is correct |
| Dark mode not working | Ensure theme script in Layout runs before page content |
| Animations not playing | Check `prefers-reduced-motion` browser setting |

## Related Files

- `src/layouts/Layout.astro` - Base layout with theme script
- `src/styles/global.css` - Design tokens and utilities
- `package.json` - Astro and Tailwind dependencies