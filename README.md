# EcoPulse

Multi-page site. Start at `index.html`.

```
ecopulse/
├── index.html            Home: hero, live stats, feature preview
├── features.html         All eight features
├── how-it-works.html     Five-step flow
├── about.html            Mission, vision, impact, SDGs
├── contact.html          Contact form
├── login.html            Sign in / create account (also how the admin signs in — see Admin below)
├── app/                  Workspace (one page per tool)
│   ├── dashboard.html    Citizen profile and wallet summary
│   ├── report.html       Report an issue
│   ├── qr.html           Citizen collection QR
│   ├── rewards.html      Wallet and rewards
│   ├── leaderboard.html  Top citizens
│   ├── worker.html       Tasks and collection
│   ├── analytics.html    Charts, complaints table, export/reset
│   ├── map.html          City map (Leaflet)
│   ├── community.html         Join or create a community; list of yours
│   └── community-detail.html  Report a problem, view reports, rewards, and (for the head) manage/resolve
├── admin/
│   └── index.html        Hidden admin console — only renders for a signed-in admin (see below)
├── server.mjs             Express app: accounts, communities, issues, points, rewards, admin API
├── community.mjs          Communities, issues, points ledger, rewards and admin routes
└── assets/
    ├── css/
    │   ├── style.css        All styles for the marketing/workspace pages (unchanged)
    │   └── community.css    Styles for the community and admin pages
    └── js/
        ├── core.js       Shared: helpers, theme, nav, data store, CFG
        ├── site.js       Marketing pages: features, reveal, stats
        ├── app.js        Workspace: sidebar, click routing
        └── pages/        One script per page
```

- Data is stored in the browser (localStorage) under the same key as before, so existing data carries over. Export from Analytics > Export data to back up.
- Set `CFG.email` near the top of `assets/js/core.js` to enable email from the contact form.
- Requires Node.js 24 or newer. Run `npm install`, then `npm start` and open `http://localhost:8000`.
- Create an account from the Sign in page. Passwords are hashed with scrypt and sessions use HttpOnly, SameSite cookies; sign-in endpoints are rate-limited. Set `NODE_ENV=production` behind HTTPS to enable Secure cookies.
- In account-creation mode, the first GPS tap fills State, City, and a mapped Ward/Village/Locality for review; a second tap verifies those values before signup. If map data has no local area, users can enter their ward/village manually; State and City are GPS-verified and the locality is marked unverified. Manually entered details can also be checked directly. GPS coordinates are sent to OpenStreetMap Nominatim, cached for up to 30 days, and are not stored on the account profile. Review the [Nominatim usage policy](https://operations.osmfoundation.org/policies/nominatim/). Set `NOMINATIM_CONTACT` to a project contact address when deploying.
- Account credentials and sessions are stored in `data/ecopulse.sqlite`. The existing complaints, collections, and rewards demo data still lives in browser localStorage and is not yet account-synced.
- Run backend tests with `npm test`.
- Needs internet for fonts, icons, QR library and map tiles.

## Communities

Anyone can create a community for a city ward, school, college, hostel or society from **Dashboard → Community** (`app/community.html`). Creating one makes you its **head** and gives you a join code to share; joining with a code makes you a member.

Members can, from the community page:
- Submit a cleanliness report with a title, category, optional photo (JPEG/PNG/WebP, resized client-side, capped at 900 KB), and a location that's either typed or captured live via the browser's GPS.
- See their own community's reports, their EcoCredit balance, and the reward catalogue.
- Redeem rewards once they have enough points; a redemption request goes to the head to fulfil.

The community **head** additionally gets a Manage tab to:
- Respond to a report — mark it **solving on the way** (in progress), **solved** (resolved), or **not a valid report** (rejected), with a message shown to the reporter.
- Award EcoCredits when marking a report solved (0–100 points, editable per report to match how much the report was worth).
- Add, edit, deactivate or remove rewards and set their point cost.
- See a per-community leaderboard and mark redemption requests as fulfilled.

Points are tracked per community in a ledger (`points_ledger`), so a citizen's balance is scoped to each community they belong to. A reporter can't award themself points, and points are only awarded once per report (editing a resolved report's message doesn't re-award).

## Admin

There's no separate admin login page — this is intentional so the admin route stays hidden. The admin account signs in through the same **Sign in** form as everyone else, with:

- Email: `hallosaini3@gmail.com`
- Password: `teambreak`

On first run in development, the server seeds this account automatically (role `admin`). Signing in as this user and visiting `/admin` opens the console; anyone else requesting `/admin` gets a plain 404, and the app workspace itself redirects this account straight to `/admin`. For a real deployment, don't rely on the built-in defaults — set `ADMIN_EMAIL` and `ADMIN_PASSWORD` environment variables (see below) and consider changing them after the first login.

The admin console (`admin/index.html`) shows:
- **Overview** — total citizens, communities, EcoCredits issued, and a breakdown of reports by status and category.
- **Issues** — every report across every community, searchable and filterable by status, with the reporter's name and email, photo, and the head's response.
- **Communities** — every community, its head, member count, open reports, and a toggle to mark it "Verified" (e.g. once you've confirmed it's a real school/ward/hostel).
- **Users** — every registered citizen, their location, and how many reports they've filed.

Admin is read/oversight-focused: it doesn't resolve reports on a community's behalf (that stays with the community head), it verifies communities and gives a cross-community view.

### Environment variables for the admin account
```
ADMIN_EMAIL=hallosaini3@gmail.com
ADMIN_PASSWORD=teambreak
```
Set these before running in production; without them, in production mode no admin account is auto-created and you'll need to promote a user's `role` column to `admin` directly in `data/ecopulse.sqlite`.
