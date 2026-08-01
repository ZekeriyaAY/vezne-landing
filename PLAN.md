# Plan: Outstanding Work — vezne-landing

Working list of marketing-site content and pages that are actionable
today, plus what's blocked and why. Cross-cutting business/infra
decisions (hosting, payment providers, legal engagement, final
pricing) are tracked separately in the workspace-level launch plan and
only surface here once they unblock a concrete page or section — see
"Workspace" in `AGENTS.md` for why that split exists and what stays
out of this (public) repo.

Items move out as they ship — git history keeps the long-term record.

---

## 1. Actionable now

Nothing currently unblocked — see §2 for what's next once its
dependency clears.

## 2. Blocked

- **Pricing page** — needs the payment-provider decision finalized
  (workspace `LAUNCH-PLAN.md` §3) and a confirmed TRY/USD/EUR
  geo-switch approach before publishing real numbers.
- **Official legal text** — `/terms` and `/privacy` are now live here
  (`src/pages/terms.astro`, `src/pages/privacy.astro`,
  `src/layouts/LegalLayout.astro`), migrated as-is from the draft
  English text that used to live in `vezne-web`
  (`TermsPage.tsx`/`PrivacyPage.tsx`/`LegalPageShell.tsx`). That repo
  still has the old in-app copies and internal links pending removal
  — see its `PLAN.md`. This is still the **draft, non-lawyer-reviewed**
  text, and KVKK Aydınlatma Metni, Mesafeli Satış Sözleşmesi, and
  Çerez Politikası don't exist yet at all. All of that is blocked on
  lawyer engagement (workspace
  `LAUNCH-PLAN.md` §5/§7) — swap in the official TR-reviewed text
  (and add the missing documents) once it lands.
- **Testimonials section** — no real users to quote until the beta
  cohort (workspace `LAUNCH-PLAN.md` v9) is running.
- **Status page link** (`status.<domain>`) — blocked on the status
  page itself being provisioned (workspace `LAUNCH-PLAN.md` §7).
- **Blog** — deferred until after public launch (v10); cadence
  ~1-2 posts/month, SEO-driven.

## 3. Low-priority / parking lot

- Blog topic backlog / SEO keyword research.
- Turkish (and further) locale toggle — revisit once the English
  content ships and traction justifies the localization work.
