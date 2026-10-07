# Calendar Frontend Application

A calendar application built with React, TypeScript, Vite, and UnoCSS, designed to be consumed via Module Federation.

## Features

- 📅 Full-featured calendar component
- 🎨 Styled with UnoCSS utility classes
- 🔌 Module Federation ready
- 🚀 Optimized for remote consumption
- 💅 Pre-built CSS export for host apps

## Quick Start

### Development

```bash
pnpm install
pnpm dev
```

### Build

```bash
pnpm build
```

### Preview

```bash
pnpm preview
```

## Module Federation Integration

This app exposes components via Module Federation for consumption by host applications.

### For Host App Developers

**Quick integration in 2 steps:**

1. Import the calendar styles in your `main.tsx`:

```typescript
import "http://localhost:3001/assets/calendar-styles.css";
```

2. Use the calendar component:

```typescript
import { lazy, Suspense } from 'react';

const Calendar = lazy(() => import('calendar/CalendarModule'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <Calendar />
    </Suspense>
  );
}
```

### Documentation

- **[QUICK_START.md](./QUICK_START.md)** - Get started in 2 minutes
- **[HOST_APP_EXAMPLE.md](./HOST_APP_EXAMPLE.md)** - Step-by-step example
- **[MODULE_FEDERATION_GUIDE.md](./MODULE_FEDERATION_GUIDE.md)** - Complete guide
- **[IMPLEMENTATION_SUMMARY.md](./IMPLEMENTATION_SUMMARY.md)** - Technical details

## Exposed Modules

- `calendar/CalendarModule` - Calendar with styles (recommended)
- `calendar/Calendar` - Calendar component only
- `calendar/App` - Full app component
- `calendar/styles` - Styles only

## Tech Stack

- **React 19** - UI framework
- **TypeScript** - Type safety
- **Vite** - Build tool
- **UnoCSS** - Utility-first CSS
- **Module Federation** - Micro-frontend architecture

## CSS Variables

The calendar uses CSS custom properties for theming. Override these in your host app:

```css
:root {
  --primary: #f9bf3b;
  --text: #333333;
  --bg: #ffffff;
  /* ... see calendar-styles.css for full list */
}
```

## Development Notes

- Port: 3001 (configurable via `.env`)
- Module Federation entry: `http://localhost:3001/remoteEntry.js`
- Styles: `http://localhost:3001/assets/calendar-styles.css`

## Build Output

After building, the following files are generated:

- `dist/assets/calendar-styles.css` - All calendar styles
- `dist/remoteEntry.js` - Module Federation entry
- `dist/assets/` - Component bundles and assets

## License

See LICENSE file in the root directory.
