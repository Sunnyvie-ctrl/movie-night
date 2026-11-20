## Lab 1 Questions

1. **In which file are the `<html>` and `<body>` tags?**
   In `app/layout.tsx` (the required root layout).

2. **Which file renders the page at `/`?**
   `app/page.tsx`.

3. **Which folder is for images and other static files, and how do you reference them?**
   The `public/` folder at the root of the project. Files inside it are served from the root URL `/`, so `public/next.svg` is referenced as `"/next.svg"`.

4. **What is `tsconfig.json` doing in your project?**
   It configures the TypeScript compiler (strict type-checking, JSX transform, module resolution, and Next.js TypeScript plugin) and defines path aliases.

5. **What does `@/*` mean in `tsconfig.json`? Write an import that uses it.**
   `@/*` maps the `@/` prefix to the project root directory (`./*`), avoiding relative paths like `../../`.
   Example:
   ```ts
   import { getMovies } from "@/lib/movies";
   ```

6. **What does `app/layout.tsx` import from `next/font/google`, and what does Next.js do with it at build time?**
   It imports Google Fonts (`Geist` and `Geist_Mono`). At build time, Next.js downloads the font files and self-hosts them with your static assets so the browser never makes external requests to Google Fonts and avoids layout shift.

7. **What’s the difference between `layout.tsx` and `page.tsx`?**
   `page.tsx` renders the unique content for a specific route URL. `layout.tsx` defines shared wrapper UI around pages in its folder and subfolders, preserving state and avoiding re-renders when navigating between sibling pages.

8. **Where would you put a navbar that appears on every page?**
   Inside `app/layout.tsx` (inside `<body>`, above `{children}`).

9. **What is `globals.css` for, and where is it imported?**
   It defines global CSS rules and loads Tailwind CSS (`@import "tailwindcss";`). It is imported once in `app/layout.tsx` (`import "./globals.css";`).

10. **What is `AGENTS.md`, and does your app need it to run?**
    `AGENTS.md` provides up-to-date Next.js 16 documentation and conventions for AI coding agents. Your app does **not** need it to build or run.