# PropertyPal Plan Addendum — shadcn/ui

This addendum updates the PropertyPal implementation plan to explicitly use **shadcn/ui** for the frontend UI component system.

## Changes

### Frontend (Vercel)
- Next.js (App Router) + TypeScript
- **shadcn/ui** (Radix UI primitives + Tailwind) for reusable, accessible UI components
- Tailwind CSS for styling

### UI scope impacted
- Address input table (1–50 rows), CSV upload, validation errors
- Batch progress list with status badges
- Portfolio viewer layout (tabs/accordion/cards)
- Override edit forms (inputs/selects/date pickers)
- Export/download dialogs and toast notifications

## Notes
- We will generate shadcn components into `apps/web/components/ui/*` (default shadcn pattern).
- Radix primitives will be used via shadcn/ui; no separate component library will be added unless required.
