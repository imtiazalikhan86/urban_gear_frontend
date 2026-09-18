# Urban Gear Frontend

React + Vite PWA for the Urban Gear reseller platform. The API it talks to lives in the
separate `urban_gear` repository.

## Stack

- React 19 with TypeScript and Vite
- React Router for screen routing
- Redux Toolkit for cross-screen state
- Axios for HTTP, with transparent access-token refresh
- Bootstrap 5 and Material Design Icons from the purchased template

## Commands

```bash
npm install
npm run dev      # http://localhost:5173
npm run build    # tsc -b && vite build
npm run preview
```

`VITE_API_URL` points at the backend and defaults to `http://localhost:3000/api/v1`.

## API Contract

Every call goes through `src/services`. The backend enforces these rules, so the UI must not
work around them:

- Access tokens last 15 minutes; `services/http.ts` refreshes once per 401 behind a
  single-flight guard, and rotation makes a refresh token single use.
- Never present a product's `price` as a customer price. It is the reseller cost; customer
  pricing only comes from a quote preview.
- Admin-only endpoints (`/users`, order status changes) are guarded server side as well; the
  UI hides what the API would refuse.

The maintainable frontend lives in this repository and is a React/Vite PWA built on the purchased template's own stylesheets, copied into `src/theme/` from the purchased template download (`WB095FRJM-v1-0-0/template/`, kept outside this repository):

- `theme/css/bootstrap.min.css` (the template's Bootstrap 5.3 build), `theme/css/materialdesignicons.min.css` with `theme/fonts/`, and `theme/css/template.css` (the template's own `css/style.css`). `bootstrap-icons` comes from npm for the few `bi` classes the template uses. They are imported in that order in `main.tsx`, before the small `styles.css` that holds only app-specific additions.
- Build screens from template classes, not hand-written CSS: page shell `osahan-page` / `osahan-page-header` / `osahan-page-body`, cards `bg-white rounded-4 shadow-sm`, product cards `osahan-card-2`, buttons `btn btn-primary rounded-4 btn-lg`, inputs `input-group bg-white rounded-4 shadow p-1` with an `mdi` icon, filters `nav nav-pills rounded-pill` + `btn btn-outline-primary`, round icon buttons `icon-sm shadow-sm`, small print `little-text`.
- Icons are Material Design Icons (`<i className="mdi mdi-..." />`). Do not add an icon component library.
- `components/ui` keeps only what the template does not provide directly: `Button`, `Field`, and `Toast`. Cards and badges use template markup inline.
- The template is phone-first, so `.osahan-page` is capped at 560px and centred on wide screens, with a fixed bottom navigation and an off-canvas drawer.

- Routing uses `react-router-dom`. `src/App.tsx` declares the routes and the session bootstrap; `src/layouts/AppLayout.tsx` holds the chrome (sidebar, drawer, header, bottom navigation, toast) and renders an `<Outlet />`.
- The search field lives once, in the layout header: typing navigates to `/search?q=` after a 300ms debounce, and `SearchPage` renders results for that param. Do not add a second search input, and do not sync the URL back into the field while it has focus.
- Navigation is role-aware in `AppLayout`: each nav entry declares `roles` and `bottomNavFor`. Resellers get Cart and Alerts; admins get Users and Orders in the bottom bar instead. Admins never see cart controls, the header bell, the margin panel, or the notification stream, and `RequireRole` in `App.tsx` redirects a wrong-role URL to `/`.
- Screens live in `src/pages`: `HomePage` (catalog and the admin product form), `SearchPage`, `CartPage` (quote preview and order placement), `AlertsPage`, `ProfilePage` (margin, password, orders, sign out), plus the signed-out `LoginPage` and `ResetPasswordPage`.
- `/` `/search` `/products/:id` `/profile` `/orders` require a token; `/cart` and `/alerts` are reseller-only and `/users` is admin-only; `/login` and `/reset-password` are public. Tapping a notification marks it read and opens `/products/:id` for the product it announced; a deleted product returns 404 and the page shows a 'no longer in the catalog' state. Static hosting needs an SPA fallback for these paths.
- Use Redux Toolkit for cross-screen state such as authentication and session data. Current slices: `auth`, `cart` (lines plus the last quote preview), `notifications`, and `ui` (the toast). Anything two screens share belongs in a slice, not in a page.
- Use the shared Axios client in `src/services/http.ts` for common `get`, `post`, `put`, `patch`, and `delete` calls.
- Keep domain API calls in `src/services`, not inside components.
- Keep reusable presentation primitives in `src/components/ui`.
- Use RTK Query later only if server-cache complexity grows; the current Axios service layer is intentionally explicit and matches the backend contracts.
- Keep customer pricing derived through quote preview; never display reseller cost as a customer price.
