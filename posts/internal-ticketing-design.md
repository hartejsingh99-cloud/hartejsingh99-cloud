# Designing Internal Ticketing That Ops Teams Actually Use

## Thesis

Most internal ticketing systems fail not because the tech is wrong but because the ticket schema asks too much, the routing logic is absent, and the reporting is invisible to the people who filed the ticket. The PM's job is to design the minimum viable schema that earns its keep, a routing layer that prevents orphaned tickets, and a feedback loop that keeps ops teams filing.

## The frame

Every operations team above a certain size eventually needs a ticketing system. The usual path: someone evaluates Jira/Zendesk/Freshdesk, it gets adopted for external support, and then internal teams try to use it too. It sort of works — until it doesn't. Internal tickets have different SLAs, different routing logic, different escalation paths, and different reporting needs than customer-facing tickets. Eventually someone says "we need our own system."

That's when the PM gets pulled in. And most PMs over-design it.

## Why it's harder than it looks

**1. Schema bloat kills adoption.**
The instinct is to add fields: priority, category, sub-category, affected team, environment, reproduction steps, expected behavior, actual behavior... By the time the requester fills out the form, they've spent more time describing the problem than it would take to walk over and ask. The discipline: every field must earn its spot. If a field isn't used in routing, reporting, or resolution, it doesn't belong on the creation form. You can always ask for more information after triage.

**2. Routing is the real product.**
A ticket without an owner is a ticket that dies. Most internal ticketing systems fail at routing — tickets land in a general queue, nobody claims them, the requester gives up and uses Slack instead. The PM must design the routing layer: which fields determine the owner? Is it category-based (finance tickets → finance team), skill-based (database tickets → the DBA), or load-balanced? The answer is usually a hybrid, and it needs an escape hatch for tickets that don't fit any rule.

**3. SLAs need teeth, but not the teeth you think.**
Internal SLAs feel awkward — you're setting deadlines for your own colleagues. But without them, response time is unbounded. The trick: don't enforce SLAs punitively. Instead, make SLA status visible. A ticket that's been open for 3x its target SLA should show up on the team lead's dashboard with a color change, not trigger an angry email. Visibility creates accountability without creating resentment.

**4. Reporters stop filing if they can't see what happened.**
The most common internal ticketing failure mode: someone files a ticket, it enters a black hole, they never hear back. They switch to Slack DMs. The system withers. The fix is simple but often skipped: every state change emails the reporter. Resolution includes a one-line summary visible to the reporter. A "my tickets" view exists and is fast. If the reporter can see their ticket's journey, they'll keep filing.

## A playbook for PMs designing this

1. **Start with 5 fields or fewer on the creation form.** Title, description, category (dropdown), priority (default to medium), and maybe one routing hint (e.g., affected team). Everything else is post-triage metadata that the resolver fills in.

2. **Design the state machine before the UI.** States: Open → Triaged → In Progress → Blocked → Resolved → Closed. Transitions matter more than states — who can move a ticket from Open to Triaged? Only the assigned team? An auto-triage bot? Define this first; the UI follows.

3. **Build category-based routing with an overflow queue.** Map each category to an owning team. Tickets that don't match any category go to a rotating triage owner. Review the overflow queue weekly — if a category keeps appearing, create a new routing rule.

4. **Make SLA visible, not punitive.** Define target response time and target resolution time per priority level. Display SLA status (on track / at risk / breached) on every ticket and on team dashboards. Don't auto-escalate on breach — just make it visible. Escalation should be a human decision.

5. **Close the loop with the reporter.** Auto-notify on every state change. Require a resolution note on close. Build a "my tickets" page that loads in under 2 seconds. If reporters feel heard, they'll use the system.

6. **Instrument for the ops lead, not just the resolver.** The ops lead needs: tickets by category over time (to spot systemic issues), median resolution time by team (to staff correctly), and reopen rate (to catch premature closures). These three metrics are the minimum viable reporting surface.

## How to know you got it right

- **Filing rate holds steady or grows** week over week — teams aren't abandoning the system for Slack.
- **Median time-to-triage < 4 hours** for high-priority tickets — routing is working.
- **Orphan rate < 5%** — tickets sitting unassigned for more than one business day.
- **Reopen rate < 10%** — tickets aren't being prematurely closed.
- **Reporter satisfaction** (even a simple thumbs-up/down on resolution) stays above 70% positive.
- **Ops leads actually use the dashboard** — check weekly active usage, not just "it exists."

## Closing

Internal ticketing is one of those systems that looks trivial until you build it. The schema, routing, SLA, and feedback loop are all PM decisions, not engineering decisions. Get those four right and the system earns trust. Get any of them wrong and your ops team is back on Slack within a month.
