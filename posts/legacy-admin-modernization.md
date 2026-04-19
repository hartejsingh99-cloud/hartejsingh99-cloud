# Modernizing a Legacy Admin Panel While Ops Is Using It Live

## Thesis

Legacy admin panels accumulate implicit workflows that no one documented. Modernizing them isn't a rewrite — it's an excavation. The PM's job is to discover what ops actually does (vs. what they say they do), slice the migration so nothing breaks mid-transition, and measure success in workflow continuity, not feature parity.

## The frame

Your company has an internal admin panel. It was built years ago — maybe PHP, maybe early jQuery, maybe a CRA app that's now three major versions behind. It works. Ops uses it hundreds of times a day. Nobody wants to touch it. And now someone has decided it needs to be modernized — new stack, new design, better performance, mobile-friendly, "the whole thing."

The instinct is to treat this like a greenfield build with a spec derived from the old panel. Screenshot every page, rebuild each one in the new stack, ship it, redirect. This fails almost every time, for reasons that have nothing to do with engineering.

## Why it's harder than it looks

**1. Implicit workflows are invisible in the UI.**
An ops team using a panel for two years has developed workflows that exist in muscle memory, not in the interface. They click these three buttons in this order, then open this page in a new tab to cross-check, then go back and change a status. The panel doesn't enforce this flow — it just happens to support it. If your new panel changes the navigation structure, button placement, or page load behavior, you break workflows that nobody told you existed because nobody thinks of them as "workflows."

**2. Feature parity is a trap.**
The old panel has 80 features. Ops uses 15 of them daily, 10 weekly, and the rest are either dead code or used by one person who figured out a workaround for something five years ago. Building feature parity means rebuilding 55 features nobody needs. The PM's job is to instrument the old panel before rebuilding the new one — click tracking, page-visit frequency, and a week of shadowing ops users. The data tells you what to build; the old panel tells you what existed.

**3. The strangler pattern requires coexistence, not a cutover.**
You can't flip everyone to the new panel on day one. The migration happens in slices — one module at a time, one team at a time. During transition, ops uses both panels. This means: shared auth, shared data layer, and clear routing (which URL goes where). The PM must define the slice order, the coexistence contract, and the rollback plan per slice. If any slice breaks, you roll back that slice, not the whole migration.

**4. Power users will resist, and they're right to.**
The person who processes 200 orders a day has optimized their workflow around the old panel's quirks — keyboard shortcuts, tab order, the exact scroll position of the "approve" button. Your new panel is objectively better but subjectively slower for them. The PM must identify power users early, involve them in the slice design, and accept that "better" means "faster for them" not "prettier for us."

## A playbook for PMs running this migration

1. **Instrument before you design.** Add lightweight analytics to the old panel: page views, button clicks, session duration per page. Run it for two weeks. The heatmap of actual usage is your spec — not the feature list.

2. **Shadow three power users for a full day each.** Watch them work. Don't ask "what do you use?" — watch what they do. Document the implicit workflows: the tab patterns, the cross-checks, the copy-paste-into-spreadsheet moments. These are your migration risks.

3. **Define slices by workflow, not by page.** "Order management" is a workflow. "The orders page" is a UI surface. The workflow might touch 4 pages, 2 modals, and a status-change API. Slice by workflow so each migration unit is self-contained and testable.

4. **Ship the strangler with shared auth from day one.** Users should not have two logins. Shared SSO between old and new panel is non-negotiable before the first slice ships. Without it, ops will avoid the new panel because "I have to log in again."

5. **Measure workflow completion time, not feature count.** Success isn't "we rebuilt 80 features." Success is: "the order-approval workflow takes 45 seconds in the new panel vs. 60 in the old one, and zero ops users have reverted." Instrument the key workflows in both panels during coexistence and compare.

6. **Plan the decommission as carefully as the launch.** When do you kill the old panel? Not when the new one is "done" — when every active user has migrated and the old panel's traffic is near zero. Set a traffic threshold for decommission. Communicate a sunset date. Leave a redirect for stragglers.

## How to know you got it right

- **Power-user adoption within 2 weeks of each slice** — if they're still using the old panel after 2 weeks, something is broken in the workflow translation.
- **Zero dual-panel workarounds** — ops isn't copying data from new panel, pasting into old panel (or vice versa) to complete a task.
- **Workflow completion time equal or better** than the old panel for the top 5 workflows, measured during coexistence.
- **Support ticket rate for "where is X?"** drops to near-zero within 1 month of each slice. If ops can't find things, the navigation model failed.
- **Old panel traffic declines monotonically** after each slice ships — no reversions, no "I went back because."

## Closing

Admin panel modernization is a change management problem disguised as an engineering project. The PM who treats it as a rewrite will deliver a prettier tool that ops doesn't use. The PM who treats it as a workflow migration will deliver a tool that ops forgets is new — which is the highest compliment an internal tool can receive.
