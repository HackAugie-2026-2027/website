# HackAugie 2026–2027 Website — Plan

The site has two jobs:

1. **Share information.** Tell students what HackAugie is, when and where it happens, who can come, and what to expect.
2. **Sign people up.** Collect registrations from participants, and later from mentors, judges, volunteers and sponsors, in a form organizers can actually manage.

Everything below follows from those two jobs. Sections marked **❓ Decision** need input from the organizing team.

## 0. Decisions so far (updated 2026-11-17)

| Topic | Decision |
|---|---|
| Eligibility | Open to students from other schools, not only Augustana |
| Capacity | No cap and no waitlist |
| MLH | MLH member event. The MLH Code of Conduct, MLH data-sharing checkboxes and MLH trust badge are required |
| Cloud | **Google Cloud (GCP)**. A credit grant from Google is **being requested** (not yet received). The GCP project is not created yet |
| Hosting | **Cloud Run** runs the site and its server code (see §4) |
| Sign-up | A form on the site that writes to a GCP database (see §4) |
| Database | **Cloud SQL for PostgreSQL** (SQL) |
| Email | **Resend** free tier (3,000/month, 100/day), sending from the club domain. Decision emails go out in daily batches |
| Team inbox | **Cloudflare Email Routing** (free): `hello@` etc. forward to organizers' inboxes; replies sent as `hello@` through Gmail "Send mail as" using Resend SMTP |
| Domain / DNS | Registered at or moved to **Cloudflare** (needed for Email Routing). Name ❓ |
| Repo/org admin | The project lead controls the GitHub org |

### Key dates (from the HackAugie 2027 Timeline spreadsheet; weeks start on Sunday)

| When | Milestone |
|---|---|
| Oct 25 – Nov 7, 2026 | Branding |
| Nov 1 – Nov 14 | Date + venue decided |
| Nov 22 | MLH |
| **Nov 8 – Jan 2** | **Main website + application form built** |
| Dec 6 | Media posting timeline; mentor outreach begins |
| **Jan 3, 2027** | **Website live, registration opens** |
| Jan 31 – Feb 20 | Applicant review |
| Mar 14 – Mar 28 | Applications close |
| Apr 11 | Setup, live media |
| **Week of Apr 25, 2027** | **Event day** (exact date TBD) |
| May | Sponsor thank-you, recap post/video, debrief |

The "Applicant review" step means sign-up is an **application** that organizers accept or reject, not first-come registration. **Confirmed:** organizers accept or reject each applicant, and the site sends acceptance and rejection emails.

---

## 1. Audiences

| Audience | What they need from the site |
|---|---|
| Prospective participants (students) | What, when, where, eligibility, cost (free?), how to sign up, what to bring |
| Registered participants | Schedule, rules, team formation, Discord/Slack link, updates, day-of logistics |
| Mentors / judges / volunteers | What they'd do, time commitment, sign-up |
| Sponsors | Why sponsor, tiers, contact |
| Organizers (internal) | An easy way to edit content and view or export registrations |

## 2. Sitemap

```
/                 Home: hero, date/location, primary "Apply" button, short pitch
/about            What a hackathon is, who it's for, beginner-friendly messaging
/schedule         Event timeline (published as it firms up)
/tracks           Themes / prize categories (optional, can live on /about at first)
/faq              Eligibility, cost, teams, food, overnight, accessibility, what to bring
/apply            Hacker application form
/apply/thanks     Confirmation and next steps (join Discord, add to calendar)
/get-involved     Mentor / judge / volunteer sign-up
/sponsors         Sponsor logos and a link to the sponsorship packet / contact
/code-of-conduct  Required for any hackathon (adapt the MLH Code of Conduct)
/contact          Email plus socials
```

Phase 1 needs only **Home, FAQ, Apply, Code of Conduct**. Everything else can follow.

## 3. Registration: what to collect

Keep the form short. Each extra field costs sign-ups.

**Required**
- Full name
- School email (to verify eligibility)
- School / university
- Year (First-year, Sophomore, Junior, Senior, Grad, Other)
- Major
- Programming experience level (None / Beginner / Intermediate / Advanced)
- T-shirt size
- Dietary restrictions or allergies
- Code of Conduct agreement (checkbox)
- Photo/media release (checkbox)

**Optional**
- Pronouns
- Do you have a team? Teammates' names
- What are you hoping to build or learn?
- Accessibility needs
- How did you hear about HackAugie?
- GitHub / LinkedIn / portfolio
- Emergency contact (only if the event runs overnight; may be needed for school policy)

**Required for MLH events**
- Agreement to the MLH Code of Conduct
- Agreement to share application data with MLH (MLH Privacy Policy and Contest Terms)
- Optional opt-in to MLH emails
- Age, or at least an "I am 18+" confirmation, plus country of residence and level of study, which MLH asks member events to collect

Open to other schools, no cap, no waitlist (decided). ❓ **Decision:** Is there an age minimum (18+)? Do you need to collect anything about travel?

**Privacy:** Collect only what's needed, state on the form how the data is used, and delete it after the event.

## 4. Technical approach

### Recommendation

- **Site:** [Astro](https://astro.build) with the Node adapter. Content pages are pre-built as plain HTML; only the form endpoint and `/admin` run server code. Content lives in Markdown (FAQ, schedule, about), so non-developers on the team can edit it through GitHub's web editor.
- **Styling:** Tailwind CSS, or plain CSS with a small set of design tokens (colors, fonts) using Augustana-adjacent branding.
- **Domain:** ❓ Custom domain (e.g. `hackaugie.com`) or a subdomain from the college?

### Google Cloud architecture

```
GitHub (main / PRs)
   │  GitHub Actions: build container → Artifact Registry → deploy
   ▼                 (keyless auth via Workload Identity Federation)
Cloud Run "web" ──── public site + POST /api/apply ──► Cloud SQL PostgreSQL
   │                                                       ▲
   │ custom domain (Firebase Hosting rewrite               │
   │  or Cloud Run domain mapping)                         │
Cloud Run "admin" ── behind Identity-Aware Proxy ──────────┘
                     (organizer Google accounts only)
Secret Manager ───── email-provider API key
Resend (or similar) ─ confirmation + decision emails
```

| Job | Service | Notes |
|---|---|---|
| Site + form API | **Cloud Run** (`web` service) | One container. Scales to zero when idle. Can raise the minimum to 1 instance during the application window to avoid slow first loads |
| Organizer review | **Cloud Run** (`admin` service) + **Identity-Aware Proxy** | IAP limits access to a list of organizer Google accounts, so the app needs no login code. Lists, filters, sets accepted/rejected, exports CSV |
| Database | **Cloud SQL for PostgreSQL** | Smallest instance with automated backups and deletion protection. Cloud Run connects through its built-in Cloud SQL connection; no IP allowlists. See §4a |
| Container images | **Artifact Registry** | |
| PR previews | Cloud Run **revision tags** | Each PR is deployed as a tagged revision that gets its own URL and no live traffic |
| Custom domain + CDN | **Firebase Hosting** rewrite to Cloud Run *(recommended)*, or Cloud Run domain mapping | Firebase Hosting gives free SSL and caches the static pages |
| Secrets | **Secret Manager** | Email API key; never committed to the repo |
| Email | **Resend** (decided), sending from the club's own domain | GCP has no built-in transactional email service. See §4b |
| Cost safety | **Billing budget alerts** | Alert well below the credit amount |

**Estimated cost:** about $15–30/month, most of it the Cloud SQL instance, which runs around the clock. The Google credit covers it. Stop or delete the instance after the May cleanup so it doesn't keep using credit.

**Until the credit arrives:** build and test locally (Postgres in Docker) and on Cloud Run's free tier. Create the Cloud SQL instance only once the credit is applied, or at the start of M3 at the latest; without credit it costs about $15–30/month. If the credit is denied, either pay that from the club budget or switch the database to Firestore (free tier).

**Credit checklist (once granted):** Confirm the amount, the **expiry date** (it must cover the event in April and the cleanup in May), which billing account it's applied to (a club-owned one, with at least two organizers as owners), and whether it excludes any services.

**Fallback:** If the form isn't ready by Jan 3, `/apply` can link to a Tally form temporarily.

**Possible later addition:** "Log in with MLH" (MyMLH), so applicants sign in with their MLH account, which pre-fills their profile. It requires MLH OAuth approval.

### 4a. Database details (Cloud SQL for PostgreSQL)

- **Why PostgreSQL over MySQL:** both work on Cloud SQL. Postgres has better support for constraints, enums and JSON, and Astro/Node libraries default to it.
- **Instance:** the smallest shared-core instance is enough for hundreds of applications. Turn on automated daily backups, point-in-time recovery and deletion protection.
- **Connecting from Cloud Run:** use the built-in Cloud SQL connection (Unix socket) and keep the database password in Secret Manager. IAM database login is possible later.
- **Connection limits:** small instances allow few connections. Cap Cloud Run at about 5 instances and give each a pool of 2–5 connections.
- **Schema and migrations:** use [Drizzle ORM](https://orm.drizzle.team), which is TypeScript, close to plain SQL, and keeps migrations in the repo. Migrations run as a step in the deploy workflow, before the new version goes live.
- **Environments:** production and preview use **separate databases** on the same instance (`hackaugie` and `hackaugie_preview`). PR previews must never touch real applicant data.
- **Local development:** Postgres in Docker (`docker compose up`).
- **Ad-hoc queries:** organizers with access can query directly in Cloud SQL Studio in the console. The `admin` app remains the normal review tool.

**Draft `applications` table**

| Column | Type | Notes |
|---|---|---|
| `id` | uuid, primary key | |
| `created_at`, `updated_at` | timestamptz | |
| `first_name`, `last_name` | text | |
| `email` | text, unique, case-insensitive | One application per email |
| `school`, `major` | text | |
| `level_of_study` | enum | MLH list (high school, undergraduate, graduate, …) |
| `graduation_year` | int | |
| `country` | text | MLH requires country of residence |
| `is_18_or_older` | boolean | Pending the age-policy decision |
| `experience` | enum | none / beginner / intermediate / advanced |
| `shirt_size` | enum | |
| `dietary`, `accessibility` | text, nullable | |
| `has_team`, `teammates` | boolean, text | |
| `goals`, `heard_about`, `links` | text, nullable | |
| `coc_agreed_at`, `mlh_share_agreed_at` | timestamptz | When each required agreement was given |
| `mlh_email_opt_in`, `media_release` | boolean | |
| `status` | enum | `submitted` → `accepted` / `rejected` / `withdrawn`; later `checked_in` |
| `reviewed_by`, `reviewed_at`, `review_notes` | text, timestamptz, text | Organizer from IAP identity |

A small `email_log` table records each confirmation or decision email sent, so nobody gets emailed twice.

### 4b. Email

Two separate needs:

**1. Automated emails from the site** (application received, accepted, rejected, reminders)

| Provider | Free tier | Paid | Notes |
|---|---|---|---|
| **Resend** *(chosen)* | 3,000/month, **100/day** | ~$20/month | Simple API, good Node/Astro support, easy domain setup |
| Postmark | 100/month (testing) | ~$15/month | Best deliverability; account approval required |
| Brevo | 300/day | Paid plans | Also does newsletters |
| Mailgun | ~100/day | Paid plans | Mature, more complex |
| Amazon SES | Pay per use (~$0.10/1,000) | — | Cheapest, but it's AWS |

The 100/day limit matters on decision day. The admin app sends decisions in batches of up to 100/day, and the `email_log` table prevents duplicates. If a faster send is needed, pay for one month of Resend (~$20) around Feb 20.

**2. A team inbox** (e.g. `hello@` / `team@` on the domain, for sponsors and questions)

- **Cloudflare Email Routing** *(chosen)*: free forwarding to organizers' existing inboxes. Requires the domain's DNS to be on Cloudflare.
- **Zoho Mail** (free plan for a few users) or **Google Workspace** (~$7/user/month): a real shared mailbox.

**Domain setup:** Send automated mail from a subdomain (e.g. `mail.hackaugie.<tld>`) with `noreply@` as the sender and `hello@` as the reply-to. Add SPF, DKIM and DMARC DNS records, which the provider gives you; university mail filters are strict without them. Register the domain at **Cloudflare Registrar** (at-cost, about $10–15/year for `.com`), so DNS, email forwarding and the Resend records live in one place.

### Other integrations
- **Email updates:** Mailchimp / Buttondown free tier, or emails sent from the application database.
- **Calendar:** "Add to calendar" `.ics` link on Home and on the /apply/thanks page.
- **Analytics:** a privacy-friendly option (Plausible, or GoatCounter, which is free) to see where sign-ups come from.
- **Community:** Discord invite link shown after registration.

## 5. Project structure (proposed)

```
website/
├── src/
│   ├── pages/            # one file per route (index.astro, faq.astro, …)
│   ├── layouts/          # shared page shell (nav, footer, <head>)
│   ├── components/       # Hero, FAQItem, ScheduleTable, SponsorGrid, …
│   ├── content/          # Markdown/JSON that organizers edit
│   │   ├── faq.md
│   │   ├── schedule.json
│   │   └── sponsors.json
│   └── styles/
├── admin/                # Cloud Run "admin" service (application review)
├── public/               # images, logo, favicon, sponsor logos
├── Dockerfile            # container for the Cloud Run "web" service
├── .github/workflows/    # CI checks, PR preview deploys, production deploy to Cloud Run
├── PLAN.md
└── README.md             # how to run locally and how to edit content
```

## 6. Design notes

- Mobile first: most students will open the link from Instagram or a group chat.
- One obvious call to action ("Apply") above the fold on every page.
- Show the date, location and "free" in the hero.
- Use beginner-friendly language. Many students have never been to a hackathon.
- Meet WCAG AA contrast, use semantic HTML, label every form field, and make everything usable by keyboard.
- Support a dark mode if the brand allows it.

## 7. Milestones

| # | Target | Milestone | Contents |
|---|---|---|---|
| M0 | Nov 22 | Decisions + GCP setup | GCP project + billing account created, credit requested, domain bought, date/venue, logo |
| M1 | Nov 29 | Skeleton | Astro project, layout, nav/footer, Dockerfile, CI, deploy to Cloud Run, PR previews |
| M2 | Dec 13 | Content | Home, About, FAQ, Code of Conduct, Sponsors, Get Involved (placeholder copy is fine) |
| M3 | Dec 27 | Application form | `/apply` → Cloud SQL, confirmation email, `admin` service behind IAP, CSV export |
| M4 | **Jan 3** | **Launch** | Site live on the real domain, applications open |
| M5 | Mar | Pre-event | Schedule, tracks/prizes, accepted-hacker info page, Discord |
| M6 | Apr 25 | Event & after | Live schedule, judging info, then winners/gallery and recap |

## 8. Open questions for the team

1. **Google credit:** Request submitted? When is a decision expected? Once granted: amount, expiry date, billing account, excluded services.
2. **GCP owners:** Who besides the project lead gets owner access to the GCP project?
3. **Domain name:** Which domain, and who will register it? This blocks email setup.
4. **Inbox addresses:** Which addresses (`hello@`, `sponsors@`, …) and which organizers each forwards to?
5. Exact event date, venue, and whether it's overnight. The timeline puts this decision at Nov 1–14.
6. Age minimum (18+, or are minors allowed with a waiver?).
7. Travel reimbursement or buses for students from other schools? This affects the FAQ and possibly the form.
8. Branding (scheduled for Oct 25 – Nov 7): logo, colors, fonts.
9. Who reviews applications and needs access to applicant data?
10. Do mentors, judges and volunteers sign up through the site, and from when? Mentor outreach starts Dec 6.

## 9. Next steps

1. Create the GCP project and billing account (see §10), request the credit, and pick a domain.
2. Scaffold the Astro project with a Dockerfile and CI (M1).
3. Wire up GitHub Actions → Cloud Run deploys, then build content pages and the application form (M2–M3) ahead of the Jan 3 launch.

## 10. GCP project setup checklist

1. Create a **club Google account** (not a personal one) to own the project.
2. In the Google Cloud console, create a project (e.g. `hackaugie-2027`) and a **billing account**. A card is required even while usage is free; the credit is applied to this billing account once granted.
3. Add at least one other organizer as **Owner**.
4. Set a **budget alert** (e.g. $10) on the billing account.
5. Pick a region: `us-central1` recommended.
6. Share the **project ID** and region so the deploy workflow can be configured. The workflow setup (Artifact Registry, Workload Identity Federation, service accounts) can then be scripted.
