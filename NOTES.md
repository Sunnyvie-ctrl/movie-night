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


## Lab 2 Questions

1. **In the page source, are the movie titles present for `/movies`? For `/movies-client`? Why?**
   * **`/movies`: Yes.** It is an `async` Server Component that awaits `getMovies()` on the server before sending the HTML response, so the rendered movie list is already included in the initial HTML.
   * **`/movies-client`: No** (only `"Loading movies..."` appears). Next.js prerenders the initial state of the Client Component (`movies === null`) on the server, whereas `useEffect` fetches `/movies.json` only in the browser after JavaScript loads and hydrates.

2. **Without the `force-dynamic` line of Part E, you run `npm run build` again tomorrow, then `npm start`, without changing anything else. What does the "Rendered at" timestamp of `/movies` show, and why?**
   It shows **tomorrow's build timestamp** (the exact time `npm run build` ran tomorrow) and stays frozen at that value across refreshes. Without `force-dynamic`, `/movies` is statically prerendered (`○ Static`) once during `next build`, so `new Date().toISOString()` runs at build time rather than per request.

3. **Part E made every request to `/movies` about one second slower. Give one situation where `force-dynamic` is still the right choice, and explain why.**
   A page showing **live, rapidly changing data**—such as real-time ticket/seat availability, live auction bids, or current inventory levels—where serving stale build-time HTML would show outdated information or cause booking conflicts. Every visitor must receive fresh data computed at request time.

4. **A classmate sees `[movies] rendered 6 movies` in the browser console and says: "So this code runs in the browser." What do you answer, and how can you prove it?**
   * **Answer:** The code ran on the **Node.js server**, not in the browser. In development (`npm run dev`), Next.js forwards server `console.log` messages to the browser DevTools console (tagged `Server`) purely as a debugging aid.
   * **Proof:** (1) The log prints in the Node.js terminal running `npm run dev`; (2) disabling JavaScript in DevTools still displays all 6 movies on `/movies`; (3) running `npm run build && npm start` with `force-dynamic` prints the log **only** in the server terminal and never in the browser console.

5. **Which version would you choose for a public movie catalogue that changes once a week, and why?**
   The **static server-rendered version** (or ISR with a long `revalidate` window) rather than `force-dynamic` or client-side fetching. Because the catalogue only changes once a week, prerendering on the server gives visitors instant page loads (avoiding the 1-second query delay on every visit), reduces server load, and provides full HTML immediately for search engines (SEO).



## Lab 3

1. **`NavLink` uses `usePathname`, a hook that only works in Client Components. Yet the page source of `/movies/2` already contains `aria-current="true"` on the Movies link. How is this possible?**
   Marking a component with `"use client"` does not mean it runs only in the browser—Next.js still prerenders Client Components on the server to generate the initial HTML. Because the server already knows the requested URL (`/movies/2`), `usePathname()` returns `"/movies/2"` during the server render as well, so `aria-current="true"` is included directly in the HTML sent before hydration.

2. **Why does `generateStaticParams` return `String(movie.id)` and not `movie.id`? And why does `/movies/42` give a 404 in Part C without running the code of the page?**
   * **Why `String(movie.id)`:** Dynamic route parameters (`params`) extracted from a URL are always strings (`{ id: string }`), whereas `movie.id` in `movies.json` is a `number`. `generateStaticParams` must return string values matching the URL segment type.
   * **Why `/movies/42` returns 404 immediately:** With `export const dynamicParams = false;`, Next.js restricts `/movies/[id]` strictly to the IDs returned by `generateStaticParams` (`1` through `6`). Any unlisted ID like `42` is rejected at the routing layer with a 404 before executing the page component.

3. **In Part D, why is the note kept between `/movies` and `/movies/2`, and lost when you go through `/about`?**
   Both `/movies` and `/movies/2` are rendered inside the shared nested layout `app/movies/layout.tsx`. In the App Router, layouts stay mounted and preserve their React state when navigating between routes inside the same folder subtree. Navigating to `/about` leaves the `/movies` layout subtree, which unmounts `app/movies/layout.tsx` and destroys its local state.

4. **In Part E, step 4, where is the real message of the error, and why does Next.js not send it to the browser?**
   * **Where it is:** The full error message and stack trace are printed in the **server terminal** (the Node.js console running `npm start`), tagged with a `digest` hash.
   * **Why it is hidden from the browser:** In production (`npm start`), Next.js strips server error details and sends only a generic message plus the `digest` ID to `error.tsx` for **security**—to avoid leaking sensitive internal information such as file paths, database queries, or backend implementation details to users.

5. **Under `npm run dev`, "Loading the movies..." appears when you open `/movies/2`. Under `npm start` it does not, but it still appears on `/movies`. Why?**
   * **Under `npm run dev`:** Next.js renders every route dynamically on every request, so opening `/movies/2` runs the simulated data-fetching delay at request time and triggers `app/movies/loading.tsx`.
   * **Under `npm start`:** `/movies/2` was prerendered at build time via `generateStaticParams` (`● SSG`), so its HTML is already built and served instantly without triggering `loading.tsx`. Meanwhile, `/movies` still has `export const dynamic = "force-dynamic"` (`ƒ Dynamic`), so it re-renders on every request, waits for `getMovies()`, and displays `loading.tsx` while streaming.


