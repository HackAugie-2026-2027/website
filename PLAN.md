# HackAugie 2026–2027 Website — Plan

The site has two jobs:

1. **Share information.** Tell students what HackAugie is, when and where it happens, who can come, and what to expect.
2. **Sign people up.** Collect registrations from participants, and later from mentors, judges, volunteers and sponsors, in a form organizers can actually manage.

Everything below follows from those two jobs. Sections marked **❓ Decision** need input from the organizing team.

## 0. Decisions so far (updated 2026-11-08)

| Topic | Decision |
|---|---|
| Eligibility | Open to students from other schools, not only Augustana |
| Capacity | No cap and no waitlist |
| MLH | MLH member event. The MLH Code of Conduct, MLH data-sharing checkboxes and MLH trust badge are required |
| Hosting | **Not** settled. GitHub Pages is not assumed (see §4) |
| Sign-up | Not settled. It leans toward a form on the site that writes to a database (see §4) |
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

The "Applicant review" step means sign-up is an **application** that organizers accept or reject, not first-come registration. The site needs to support review status and acceptance emails.

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

- **Site:** [Astro](https://astro.build), a static site generator. Content lives in Markdown (FAQ, schedule, about), so non-developers on the team can edit it through GitHub's web editor. It produces fast, plain HTML.
- **Styling:** Tailwind CSS, or plain CSS with a small set of design tokens (colors, fonts) using Augustana-adjacent branding.
- **Hosting:** ❓ Not decided (GitHub Pages is not assumed). Because sign-up will probably need a server and a database, a host that runs serverless functions fits better than GitHub Pages:
  - **Vercel** or **Netlify**: free tier, a preview link for every PR, serverless functions for the form, and first-class Astro support. *Recommended.*
  - **Cloudflare Pages**: the same benefits, plus its own database (D1) if the team wants everything in one place.
- **Domain:** ❓ Custom domain (e.g. `hackaugie.com`) or a subdomain from the college?

### Sign-up / application backend

The timeline includes an applicant review step (Jan 31 – Feb 20), so organizers need to see applications, mark them accepted or rejected, and email decisions. Options:

| Option | How it works | Pros | Cons |
|---|---|---|---|
| **A. Form tool** | Tally or Google Form linked from `/apply`; responses go to a Sheet; review happens in the Sheet | No code, live in a day | Off-brand, manual acceptance emails, data lives in Google |
| **B. Site form + database** *(recommended)* | Form on `/apply` → serverless function → **Supabase** (Postgres). Organizers review in the Supabase table view or a small `/admin` page | Branded, validation, status tracking, can automate emails and later QR check-in | ~2–3 weeks of build, someone must own security and privacy |
| **C. Option B plus "Log in with MLH"** | Applicants sign in with MyMLH, which pre-fills their profile and MLH consent | Fewer fields to type, MLH-native | Requires MLH OAuth app approval, adds a dependency |

The build window (Nov 8 – Jan 2) is long enough for Option B. Option A is a fallback if the team runs short on time; the site can link to a Tally form on Jan 3 and switch later.

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
├── public/               # images, logo, favicon, sponsor logos
├── .github/workflows/    # CI checks (build, lint); deploy is handled by the host
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
| M0 | Nov 14 | Decisions | Hosting, sign-up approach, date/venue, logo |
| M1 | Nov 22 | Skeleton | Astro project, layout, nav/footer, CI, preview deploys |
| M2 | Dec 13 | Content | Home, About, FAQ, Code of Conduct, Sponsors, Get Involved (placeholder copy is fine) |
| M3 | Dec 27 | Application form | `/apply` form, database, confirmation email, organizer review view |
| M4 | **Jan 3** | **Launch** | Site live on the real domain, applications open |
| M5 | Mar | Pre-event | Schedule, tracks/prizes, accepted-hacker info page, Discord |
| M6 | Apr 25 | Event & after | Live schedule, judging info, then winners/gallery and recap |

## 8. Open questions for the team

1. **Hosting:** Vercel, Netlify or Cloudflare Pages? (Recommended: Vercel or Netlify.)
2. **Sign-up approach:** Option A, B or C in §4? (Recommended: B.)
3. **Application vs. registration:** The timeline has an applicant review step. Is that a real accept/reject decision, or just a check?
4. Exact event date, venue, and whether it's overnight. The timeline puts this decision at Nov 1–14.
5. Age minimum (18+, or are minors allowed with a waiver?).
6. Travel reimbursement or buses for students from other schools? This affects the FAQ and possibly the form.
7. Domain name, and who owns the DNS.
8. Branding (scheduled for Oct 25 – Nov 7): logo, colors, fonts.
9. Who reviews applications and needs access to applicant data?
10. Do mentors, judges and volunteers sign up through the site, and from when? Mentor outreach starts Dec 6.

## 9. Next steps

1. Answer questions 1–3 above.
2. Scaffold the Astro project with CI (M1). This doesn't depend on the hosting choice.
3. Connect the chosen host, then build content pages and the application form (M2–M3) ahead of the Jan 3 launch.
