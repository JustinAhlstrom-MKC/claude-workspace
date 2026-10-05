# CLAUDE.md — Justin's workspace

This folder is Justin's Claude workspace and the source of truth for MKC Restaurants operating docs. Read this file first, then check the repo before asking Justin for facts.

## Who / what
- Justin co-owns MKC Restaurants with his wife Becky: **Margie's Kitchen & Cocktails** (Andover, MN, plus the Mini Margie's food truck) and **Grackle** (Maple Grove, MN). Two LLCs, ~170 employees.
- **Brand vs. operations:** Margie's and Grackle are two distinct guest-facing brands (look, voice, menu, marketing). Underneath, they're one company: shared processes, policies, admin, and systems. Keep brand work separate; treat operations as unified, noting location-specific exceptions.
- No GMs. A crew of managers with specific roles: Emily O'Dell (senior manager), Diego (Culinary Director), Lou Armitage (bookkeeping).
- Minnesota law and the Twin Cities market apply (labor, tips/service charges, liquor, THC beverages, minors in FOH).
- Stack: Toast, 7shifts, Airtable, Google Workspace, GoTo Connect, UniFi, Wix Studio, GitHub, Todoist.

## Key files
- `mkc/tech-context.md` — what's built: Airtable bases, code repos, pipelines, status, known gaps. Read it before planning or drafting anything that touches Airtable, payroll/HR, menus, the handbook, or the websites. Claude Code regenerates it; when it's older than a recent conversation, trust the conversation.

## Related repos (WSL, `~/dev`, worked on by Claude Code — not this folder)
- `mkc-automations` — main Python repo: HR syncs, payroll, menu pipeline, training docs
- `handbook` — MkDocs employee handbook. **A push to `main` publishes to staff.**
- `mkc-websites` — website specs, brand assets, shared design system
- `grackle-website` — live Velo code for gracklegrove.com
- `automation-projects` — older, mostly dormant

API credentials exist only in WSL `.env` files. Work that needs them, or edits these repos, goes to Claude Code with a short brief: goal, decisions made, open questions, Airtable tables/fields involved.

## Folders
- `mkc/` — restaurant work. Ops reference docs live here (e.g. `mkc/facilities/`, `mkc/brand/`, `mkc/vendors/`, `mkc/people/`), each created only when its first real file arrives.
- `home/` — personal and family.
- `scratch/` — throwaway drafts. Safe to delete.

## How to work
- Lead with the answer or recommendation, then options and trade-offs. Be direct; push back when something's a bad idea.
- Give real specs, part numbers, and procedures. Flag genuine code, safety, or licensing lines — don't default to "call a professional."
- Anything staff-facing: plain reading level, short sentences, ready to translate to Spanish for BOH.
- Prefer markdown. Edit existing files instead of creating near-duplicates.
- Tasks and follow-ups: phrase as short Todoist-ready lines.

## Guardrails
- **Don't build structure ahead of content.** No new folders, templates, or taxonomies until a real file needs them.
- No new tools, automations, or bases without evidence the simple version worked first. Point it out if Justin is drifting into system-building instead of doing the work.
- Canonical facts (door/key schedule, network map, vendors, equipment, brand palettes) live in this repo. Update the file when a fact changes; don't leave the current version only in chat.
- Never put work files in `C:\Users\justi\.claude` (that's Claude's settings).
- End of session: remind Justin to commit and push.
