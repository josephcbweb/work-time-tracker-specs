# Frontend Constitution - Work Time Tracker

This document defines the architecture, styling principles, directory structure, and coding standards for the React + Vite frontend application.

---

## 1. Tech Stack & Library Configurations

* **Framework:** React 18+ (bootstrapped with Vite, utilizing TypeScript).
* **Styling:** Vanilla CSS (no CSS frameworks like Tailwind or Bootstrap).
* **Icons:** Standard SVG icons or `lucide-react`.
* **State Management:** React Context API or standard hooks (`useState`, `useReducer`) for simple global state (e.g., auth session).
* **Routing:** Custom state-based routing (for simplicity) or `react-router-dom`.

---

## 2. Design System & Aesthetics (CSS Guidelines)

To deliver a premium, high-fidelity UX, adhere to the following styling principles:
* **Theme:** Sleek dark-mode-first aesthetic with a curated color palette (harmonious deep slates, clean whites, and vibrant accent colors like cyan/emerald).
* **Layout:** Flexbox and Grid layouts. Hardcoded margins and paddings must be replaced by structured CSS custom properties.
* **UI Features:** Use glassmorphism where appropriate (`backdrop-filter: blur(12px)`), smooth border-radii, and subtle shadows.
* **Micro-Animations:** Interactive elements (buttons, inputs, day cards) must have subtle transitions (e.g., `transition: all 0.2s ease`).

---

## 3. Directory Structure

```text
work-time-tracker-frontend/
├── src/
│   ├── assets/         # Static assets (images, global SVGs)
│   ├── components/     # Reusable UI components (Card, Button, ProgressBar)
│   ├── context/        # Auth Context / Settings Context
│   ├── hooks/          # Custom hooks (e.g., useAuth)
│   ├── pages/          # Page level components (Login, Signup, Dashboard)
│   ├── services/       # API services / Fetch wrappers
│   ├── styles/         # Global styles and CSS variables
│   ├── App.tsx         # Root component
│   └── main.tsx        # Vite entrypoint
```

---

## 4. API Client & Networking Guidelines

* All HTTP communication must go through a dedicated service layer (`src/services/api.ts`).
* Use the browser's native `fetch` API.
* Manage authentication tokens (JWT) by storing them securely in `localStorage` or `sessionStorage`.
* Automatically attach the `Authorization: Bearer <token>` header to all request endpoints if the token is present.
* Gracefully handle network failures and API errors by displaying user-friendly alerts or toast notifications.
