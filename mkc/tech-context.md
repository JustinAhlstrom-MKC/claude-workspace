# MKC Restaurants — Technical Work Context (for Claude chat / Cowork)

*Snapshot as of 2026-10-05, written from the code repos on Justin's WSL machine (`~/dev`). Claude Code works in those repos; this file tells chat-side Claude what exists so plans and drafts match reality. When this file and a newer conversation disagree, trust the conversation.*

---

## 1. The business

- **MKC Restaurants**, Minnesota (use MN law for anything regulatory).
  - **Margie's Kitchen & Cocktails**, Andover MN
  - **Grackle**, Maple Grove MN
  - **Mini Margie's**, a food truck (minimargies.com)
- Margie's and Grackle are **separate businesses for vacation and sick-time accrual**.
- **Justin Ahlstrom** builds all the systems below (Python, Airtable, Wix, InDesign). He also does menu design and print, and he's the de facto maintenance manager.

### People who show up in the work

| Person | Role in these systems |
|---|---|
| **Becky** | Employee development and team relations (not general HR). Manages the music program at Margie's. Uses Airtable only, never the Python side. Co-edits the handbook. Collects supplemental-material requests for menu rollouts. |
| **Gigi** | HR Specialist / admin support. Enters jobs and rates in Toast. |
| **Diego** | Chef. Owns xtraCHEF recipes and pricing (with Justin). |
| **Jourdan** | Food descriptions, ingredients, allergens, flags and server notes. Gives final sign-off on food copy. |
| **Ashley** | Same as Jourdan, for bar and drink items. |
| **Ben** | Toast POS menu setup: items, GUIDs, photos, online ordering, final prices. |
| **Lou Armitage** (Armitage Accounting) | The "accountant" throughout this doc. Processes payroll from the biweekly hours Excel: checks, direct deposit, taxes. Also gets emails from Airtable automations. MKC is fully off ADP. |

---

## 2. Architecture in one picture

```
Toast (POS: hours, jobs, rates, menu)   7shifts (scheduling)   xtraCHEF (recipes/costing, no API)
            │                                   │                          │ (PDF export)
            └──────────────┬────────────────────┴──────────────────────────┘
                           ▼
           Python scripts (mkc-automations repo, run from WSL)
                           │  write via API
                           ▼
        AIRTABLE — "MKC" base appVWDqRGdBqhvfEj  ◄── the hub; the team works only here
          │ native automations (emails, checklists)    │ interfaces (manager UIs)
          ▼                                            ▼
   Accountant / managers          Outputs: payroll Excel, InDesign menu XML, training
                                   workbooks, Wix website (menu snapshot, event form)
```

**The rules behind this design:**

- **Airtable is the hub.** Every output (print menus, website, training docs, Toast descriptions) comes from Airtable records, not from side spreadsheets or documents.
- **Hybrid model.** Airtable handles reactive work ("when X happens, do Y"). Python handles API syncs, batch math and detection.
- **Source of truth by data type:**
  - **Toast** is the source of truth for hours worked and for hourly employees' jobs and rates. Changes are made in Toast and then sync to Airtable.
  - **Airtable** owns everything else: exempt staff roles, HR records and menus.
- Python scripts are idempotent and support `--dry-run`.

### Airtable bases

| Base | ID | Contents |
|---|---|---|
| **MKC (HR / Operations)** | `appVWDqRGdBqhvfEj` | HR, menus, maintenance, event requests, cash, reviews (~29+ tables) |
| **Margie's Music** | `app7PHkKUrAmuKWmK` | Thursday music series booking |

Location record IDs in the MKC base: Margie's `recVizuxcnbanjQGX`, Grackle `recbCJRMfmQXjAOmy`.

---

## 3. Repos in `~/dev`

| Repo | What it is | Activity |
|---|---|---|
| **mkc-automations** | Main Python automation repo: HR syncs, payroll, menu pipeline, training docs | Very active (38 commits since June 2026) |
| **handbook** | Employee handbook (MkDocs), live at team.margies-kitchen.com | Active: Time Off v2.0 in late Sept 2026 |
| **mkc-websites** | Website-overhaul specs, brand assets and design system for all three sites. No runnable code. | Active through 2026-09-18 |
| **grackle-website** | Live Velo code for the Grackle Wix Studio site (gracklegrove.com) | Active through 2026-09-18 |
| **automation-projects** | Older standalone automations (Dec 2025 – Feb 2026). Two are still live inside Airtable. | Dormant |
| `assets/`, `docs/`, `scripts/`, `test-projects/`, `web-projects/` | Empty folders, or a non-MKC Next.js learning scaffold | Ignore |

---

## 4. mkc-automations (main repo)

### HR / people systems

**Built and in use:**

- **Hiring and onboarding pipeline.**
  - A public application form feeds the Applicant pipeline.
  - A Hire button creates the Employee, EE_Location, EE History and a 14-task Checklist.
  - The HR Task Dashboard and Employee Detail interfaces support the process.
- **Change requests.** Managers submit pay, job and location changes through the "Request a Change" form. The workflow is EE History Requested → Approved → task checklist → Complete, and the accountant gets an email at the end.
- **Time Off.** Employees use a request form, and managers approve in the Time Off Requests interface.
- **Policy acknowledgement.** An e-signature form is embedded in the handbook.
- **11 native Airtable automations**, covering:
  - hire
  - Toast job/rate sync
  - change-request steps
  - logging terminations and leaves
  - the accountant email
  - event-request routing
- **Python scripts:**
  - `sevenshifts_connect.py`: daily matching of employees to 7shifts, plus contact sync.
  - `payroll.py`: biweekly, run by hand. Toast hours → Excel for the accountant, and it upserts Weekly Hours.
  - `job_rate_audit.py`: checks Toast jobs and rates against Airtable.
  - `roster_audit.py`, `comp_analysis.py`, `legacy_vacation_accrual.py`: ad-hoc reports.
  - `engines/quarterly_eligibility.py`: quarterly FT/PT classification. Built 2026-09-25 but not scheduled yet.
- **Check printing.** JSON → Illustrator template → PDF.

> ⚠️ **Scheduling gap (found 2026-10-05):** `crontab.txt` documents two daily jobs (7shifts connect at 6:15am, job/rate audit at 7:00am), but **no crontab is installed** and the logs folder has been empty since Feb 2026. The "daily" jobs are not running. Nothing is on a VPS yet; production is planned for a cheap VPS.

**The 8-system HR automation plan** (derived from the handbook):

| System | Status |
|---|---|
| 1. Classification (FT/PT, 30-day + quarterly) | Partial: quarterly engine exists, 30-day check not built |
| 2. Vacation / sick accruals and balances | Partial: request form only, no balance tracking |
| 3. Onboarding | Complete |
| 4. Benefits enrollment | Not started |
| 5. Separation (no-call/no-show, 60-day inactivity) | Partial: Terminate button and logging only |
| 6. Incident / discipline | Not started |
| 7. Attendance / overtime alerts | Not started |
| 8. Leave management (FMLA, MN Paid Leave) | Not started |

**Stated next build:** a daily Toast hours sync (`sync/toast_hours.py`), which unblocks the classification engine, which in turn unblocks accruals and benefits.

### Menu production pipeline (the most active area, Sept–Oct 2026)

```
xtraCHEF (recipes, costs, prices) ─PDF→ Airtable Menu → Menu Section → Menu Placement → Menu Items
                                          │
       ┌──────────────────────────────────┼──────────────────────────────┬────────────────────────┐
       ▼                                  ▼                              ▼                        ▼
  InDesign print                    Toast To-Do worklist             Training workbooks       Website menu
  (menu_xml.py → XML →              (menu_toast_check.py →           (food, bar, leader       (Grackle: static
   place_menu_xml.jsx fills          fields Ben works from)           guide; PDF + artifact)   snapshot via script)
   labeled frames)
```

- **Process:** each refresh clones the previous menu version (`setup/clone_menu.py`), then aligns it to the xtraCHEF dish report.
  - Claude drafts descriptions, allergens, flags and EN/ES build cards from the recipes.
  - Team members review in the **"Menu Manager"** Airtable interface and record sign-offs: Flags Verified, Pricing, Toast Verified.
- **Consumer advisory rule (asterisk):** anything cooked to temperature or possibly underdone gets the asterisk. That covers burgers, fish, pork, steak and eggs.
  - Chicken doesn't need it.
  - Hollandaise gets it. Caesar and the commercial-mayo aiolis don't.
- **Current state, Margie's Fall/Winter 2026:**
  - Dinner: 53 dishes, launched 9/30.
  - Brunch: the full new brunch is delayed. An *interim* brunch launched 10/3, using the Feb 2026 breakfast items plus a subset of dinner dishes.
  - Kids menu: 10 items.
  - Drink menu: fall cocktails named after Prince songs.
  - Training workbooks and the leader guide were built for the rollout.
- **Open items for the Margie's rollout:**
  - Jourdan's review of the drafted copy and allergens
  - new dishes still to create in Toast
  - photos (almost none exist)
  - Ranchero Bowl and Ranchero Burrito prices
  - drink allergens (Ashley)
  - fun section names for brunch Egg Favorites and Something Sweet
- **Grackle's menu refresh** comes next, on the same pipeline. Grackle's print template is "V4" (Roslindale type).

### Other operations systems in the MKC base

- **Maintenance & Repair.**
  - Records: Equipment (named by a formula, "Margie's | Bar | Ice Machine"), Equipment Type, Repair Order (the ticket), Work Order (each visit, with cost), Vendor, Part, Inspection.
  - Phase A (data model) is done. Phase D (interfaces and automations) is in progress.
  - QR-code intake and the inspection migration are deferred.
- **Margie's Music base.** The data model is done: Lead, Artist, Event, Payment, Band Members. **Next: Phase 2 interfaces** (Booking Board, Lead Pipeline, Artist Directory). Standard rates are $400 solo and $500 band. Becky is the primary user.

---

## 5. handbook

- **Format and editing.** 47 policies, one markdown file each, published with MkDocs Material to **team.margies-kitchen.com**.
  - **A push to `main` publishes straight to staff**, through a GitHub Action.
  - Word/PDF export exists for printed copies and attorney review.
  - Justin edits in VS Code. Becky edits through the GitHub web UI or synced Google Drive files.
- **Status.** All policies are active.
  - Most are v1.0, effective 2026-01-01.
  - **Time Off / Vacation Policy v2.0** took effect 2026-09-03. It's pushed and live, and the working tree is clean.
- **Open compliance items:**
  - attorney review (8-item checklist)
  - broker review of the ACA measurement periods (MKC has 50+ employees)
  - voting, jury and USERRA leave policies, deferred by ownership
  - `REVIEW-STATUS.md` still describes the old PTO rules and needs an update.
- **Key rules (the handbook is the source of truth; the Python engines implement these):**
  - **Classification:** 30+ hrs/week average = full-time; under 30 = part-time; exempt = salaried management. Initial check at 30 days, then quarterly reviews (Jan 1, Apr 1, Jul 1, Oct 1).
  - **Sick & Safe Time (MN ESST):** 1 hr per 30 hrs worked, 48 hrs/yr, all employees.
  - **Hourly vacation ("PTO" in payroll and Airtable):**
    - 24 hrs per grant if the employee worked more than 700 clocked hours in the 13 pay periods before it.
    - Summer grant is added July 1; winter grant on the last December payday.
    - The vacation year is July 1 – June 30. Everything expires June 30, and the balance tops out at 48 hrs.
    - Used in 4-hour blocks. Doesn't depend on FT/PT status.
  - **Exempt vacation:** 3.08 hrs per pay period, 80 hrs/yr cap, 40-hr carryover.
  - **Vacation and SST payout:** vacation is for planned time off only, and **neither vacation nor SST is paid out at separation.** Pay requests are due within 1 week after the time off.
  - **Benefits:** full-time and exempt only. If someone drops to part-time, they get 1 grace quarter, and benefits end after 2 consecutive part-time quarters.
  - **Separation triggers:** 3 consecutive no-call/no-shows, or 60 days inactive.
  - **Pay:** biweekly, paid Friday. Meal breaks are paid if taken on the premises. MN Paid Leave applies.

---

## 6. Websites (mkc-websites + grackle-website)

All three sites run on **Wix Studio**. The split of work:

- Claude writes specs and Velo code in git.
- Justin does all the work in the Wix editor.
- A push to `grackle-website` main syncs the code, and it goes live on the next editor Publish.

| Site | Status |
|---|---|
| **Grackle** (gracklegrove.com, rebuild in progress) | See the per-page status below |
| **Mini Margie's** (minimargies.com) | Live. Built in the editor with no code. Truck booking is an embedded Airtable form. |
| **Margie's** | Not started. Planned as phase 2, reusing the Grackle pattern with a config swap (heavier Roslindale weights). |

**Grackle page status:**

- Design system: done in the editor.
- Home: done. Email signup goes to Wix Contacts plus marketing consent.
- Events: done. The private-dining inquiry form writes to the Airtable **Event Request** table through the Velo backend, and Airtable Automation 11 then notifies the event manager.
- Menu: only 1 of 6 sections is built. Its data is a static Airtable snapshot pulled by a script.
- Not done: final publish and live sweep.

**Integrations:**

- Resy: a reservation modal via custom code.
- Toast: online-ordering link.
- Airtable: event form and menu data.

**Open items:**

- finish the menu page
- footer gift-card and careers links
- event capacities copy and hours
- spam hardening on the event form
- an About page decision

**Design references:** `shared-design-system/` (tokens, components, integrations, and "Wix Studio lessons", which must be read before editor work) and the Grackle brand starter kit (Roslindale type, brass/cream palette).

---

## 7. automation-projects (older and mostly dormant)

| Project | Status |
|---|---|
| **review-analysis** | **Live as an Airtable automation.** Nightly sync of Google Business Profile reviews into the Airtable Reviews table, with a running average. Not built yet: negative-review alerts and AI-drafted replies. |
| **cash-management** | **Live as an Airtable automation.** Twice a week, it compares the closing cash drawer counts with par, rounds up to bank pack sizes, and emails the bank order. |
| **employee-data** | Superseded. The 7shifts → Weekly Hours sync and quarterly eligibility logic now live in mkc-automations. |
| **menu-automation** | Superseded by the mkc-automations menu pipeline. |

---

## 8. How to divide work between chat and Claude Code

- **Chat / Cowork is good for:**
  - planning a system before it's built
  - drafting policies, emails, staff communications and training copy
  - menu copywriting
  - reviewing business rules against MN law
  - reading or editing Airtable data directly through the Airtable connector (same base IDs as above)
- **Claude Code (WSL) is good for:**
  - anything in the repos: Python scripts, Velo code, handbook edits, InDesign JSX
  - running syncs and reports with API credentials (those exist only in `.env` on the WSL machine)
  - committing changes
- **When handing work from chat to Code**, write a short brief: the goal, the decisions already made, the open questions, and the Airtable tables/fields involved. `docs/grackle-menu-pipeline-brief_1.md` is an example of one that worked well.
- **Things to keep in mind:**
  - Airtable's API can't create formula, rollup, lookup or count fields. Those are manual UI steps for Justin.
  - Airtable interface buttons run as the "automation" user, not as the person who clicks them.
  - Never treat a handbook push as routine, because it publishes to staff.

## 9. Known gaps and priorities (as of 2026-10-05)

1. **Cron isn't installed.** The daily 7shifts sync and job/rate audit aren't running. Either install the crontab or move the jobs to a VPS.
2. The daily Toast hours sync, then the classification engine (30-day check), then accruals and balances.
3. Finish the Margie's F/W 2026 rollout loose ends (listed above), then the Grackle menu refresh.
4. Finish the Grackle website menu page, then publish.
5. Music base interfaces (Phase 2), and the maintenance system interfaces and automations (Phase D).
6. Handbook: attorney and ACA review. Update `REVIEW-STATUS.md`.
