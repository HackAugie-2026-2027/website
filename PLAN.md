# HackAugie 2026–2027 Website — Plan

The site has two jobs:

1. **Share information.** Tell students what HackAugie is, when and where it happens, who can come, and what to expect.
2. **Sign people up.** Collect registrations from participants, and later from mentors, judges, volunteers and sponsors, in a form organizers can actually manage.

Everything below follows from those two jobs. Sections marked **❓ Decision** need input from the organizing team.

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
/                 Home: hero, date/location, primary "Register" button, short pitch
/about            What a hackathon is, who it's for, beginner-friendly messaging
/schedule         Event timeline (published as it firms up)
/tracks           Themes / prize categories (optional, can live on /about at first)
/faq              Eligibility, cost, teams, food, overnight, accessibility, what to bring
/register         Participant sign-up form
/register/thanks  Confirmation and next steps (join Discord, add to calendar)
/get-involved     Mentor / judge / volunteer sign-up
/sponsors         Sponsor logos and a link to the sponsorship packet / contact
/code-of-conduct  Required for any hackathon (adapt the MLH Code of Conduct)
/contact          Email plus socials
```

Phase 1 needs only **Home, FAQ, Register, Code of Conduct**. Everything else can follow.

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

❓ **Decision:** Is the event open to other schools, or Augustana students only? Is there an age minimum (18+)? Is there a capacity cap with a waitlist?

**Privacy:** Collect only what's needed, state on the form how the data is used, and delete it after the event. If the event is affiliated with MLH, their data-sharing checkboxes are required.

## 4. Technical approach

### Recommendation

- **Site:** [Astro](https://astro.build), a static site generator. Content lives in Markdown (FAQ, schedule, about), so non-developers on the team can edit it through GitHub's web editor. It produces fast, plain HTML.
- **Styling:** Tailwind CSS, or plain CSS with a small set of design tokens (colors, fonts) using Augustana-adjacent branding.
- **Hosting:** GitHub Pages, deployed with GitHub Actions on every push to `main`. It's free and the repo is already on GitHub. Netlify or Vercel are equally fine if preview deploys for PRs are wanted.
- **Domain:** ❓ Custom domain (e.g. `hackaugie.com`) or a subdomain from the college?

### Registration backend: phased

A static site can't store form submissions by itself, so:

| Phase | Approach | Pros | Cons |
|---|---|---|---|
| **1 (launch)** | Embedded or linked **Google Form** / **Tally** form; responses go to a Google Sheet | Live in an hour, organizers already know Sheets, no code to secure | Looks less integrated, limited validation and custom logic |
| **2 (if needed)** | Custom form on `/register` posting to **Supabase** (Postgres + auth) or a small serverless function | Fully branded, check-in / waitlist / dedupe logic, participant dashboard possible | Needs someone to maintain it, plus security and privacy responsibility |

**Recommendation:** Launch with Phase 1. Move to Phase 2 only if a concrete need appears, such as QR check-in, waitlist automation, or applicant acceptance emails.

### Other integrations
- **Email updates:** Mailchimp / Buttondown free tier, or mail merge from the Sheet.
- **Calendar:** "Add to calendar" `.ics` link on Home and on the thanks page.
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
├── .github/workflows/    # deploy.yml → GitHub Pages
├── PLAN.md
└── README.md             # how to run locally and how to edit content
```

## 6. Design notes

- Mobile first: most students will open the link from Instagram or a group chat.
- One obvious call to action ("Register") above the fold on every page.
- Show the date, location and "free" in the hero.
- Use beginner-friendly language. Many students have never been to a hackathon.
- Meet WCAG AA contrast, use semantic HTML, label every form field, and make everything usable by keyboard.
- Support a dark mode if the brand allows it.

## 7. Milestones

| # | Milestone | Contents |
|---|---|---|
| M0 | Decisions | Answer the ❓ items, settle date/venue, get a logo |
| M1 | Skeleton | Astro project, layout, nav/footer, deploy pipeline live |
| M2 | Launch (registration open) | Home, FAQ, Register (Google Form/Tally), Code of Conduct |
| M3 | Build-up | About, Get Involved, Sponsors, email updates |
| M4 | Pre-event | Schedule, tracks/prizes, participant info page, Discord |
| M5 | Event & after | Live schedule, judging info, then winners/gallery and photos |

❓ **Decision:** Event date. Registration should open about 6–8 weeks before it.

## 8. Open questions for the team

1. Event date(s), venue, and whether it's overnight.
2. Augustana-only or open to other schools? Age limit?
3. Capacity and waitlist policy.
4. MLH member event or independent?
5. Domain name, and who owns the DNS.
6. Branding: logo, colors, fonts.
7. Who on the team will maintain content and registrations?
8. Google Form vs. Tally for Phase 1 registration.
9. Do mentors, judges and volunteers need sign-up at launch, or later?

## 9. Next steps

1. Review this plan and answer the open questions.
2. Scaffold the Astro project and set up the GitHub Pages deploy (M1).
3. Write the Home and FAQ copy, create the registration form, and launch (M2).
