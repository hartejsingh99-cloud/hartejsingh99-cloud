# Designing an Ops Platform for Scheduled Human Interactions

## Thesis

Any consumer business that requires human follow-up — scheduled calls, consultations, check-ins — eventually builds an ops platform to manage them. Most of these platforms break on calendar math, exception handling, and outcome tracking. The PM's job is to design the scheduling model, the exception grammar, and the outcome taxonomy before engineering touches a line of code.

## The frame

You sell a product or service that requires a human to call or follow up with a customer at a scheduled time. Maybe it's a consultation. Maybe it's a post-purchase check-in. Maybe it's a renewal call. Whatever the flavor, the ops team needs a system that answers three questions: who gets called, when, and by whom — and what happens when the answer to any of those changes.

This is not a calendar app. It's an operations engine that happens to involve time slots.

## Why it's harder than it looks

**1. Calendar math is deceptively complex.**
The naive model: agent has a calendar, customer picks a slot, call happens. Reality: agents work shifts that change weekly. Holidays — regional and national — blank out different agents on different days. Customers reschedule. Agents go on leave. A "simple" scheduling model has at least four interacting calendars (agent availability, customer preference, holiday calendar, capacity limits) and the intersection of all four is your actual schedulable space. Most teams discover this complexity after launch, when the exception rate is 30%+ and ops is managing it in spreadsheets alongside the tool.

**2. Outcome taxonomies expand silently.**
When you launch, calls have three outcomes: completed, no-answer, rescheduled. Within a month, ops needs: completed-positive, completed-negative, completed-neutral, no-answer-first-attempt, no-answer-final-attempt, rescheduled-by-customer, rescheduled-by-agent, cancelled, customer-unreachable, wrong-number, call-dropped, partial-completion. If your outcome model doesn't support extensibility, you'll be shipping schema migrations every two weeks. Design the taxonomy as a configurable enum from day one — and accept that you won't get it right the first time.

**3. Retry and escalation logic is where ops actually lives.**
The first call is easy. What happens after a no-answer is the real product. How many retries? At what interval? Does the agent change? Does priority escalate? Does the customer get an SMS before the next attempt? Most platforms either hard-code retry logic (inflexible) or make it fully configurable (nobody configures it correctly). The middle path: sensible defaults with override capability per interaction type. Let the ops lead change retry count and interval per category, but don't expose the full state machine.

**4. Agent assignment looks like a routing problem but is actually a relationship problem.**
Round-robin assignment optimizes for load. But in follow-up-heavy businesses, customers build rapport with specific agents. "My agent" expectations mean assignment isn't just capacity math — it's relationship continuity with a fallback for attrition and leave. The PM must decide: is this a call-center model (any-agent-is-fine) or a relationship model (same-agent-preferred)? The answer is usually "relationship model with graceful degradation," which is the hardest to build.

## A playbook for PMs designing this

1. **Map all four calendars before designing the UI.** Agent shifts, customer windows, holiday/exception calendar, capacity caps. Write out the intersection logic in plain English. If you can't explain when a slot is available without an "it depends," you haven't finished scoping.

2. **Design the outcome taxonomy with ops, not for ops.** Sit with the team lead for 30 minutes. Ask: "What are the 5 things that can happen on a call, and what do you do after each one?" Their answer IS your taxonomy. Add a catch-all "other" with a free-text field. Review the "other" bucket monthly and promote recurring outcomes to named categories.

3. **Spec the retry policy per interaction type.** Not one global policy. A first-time consultation and a 90-day renewal follow-up have different urgency, different retry windows, and different escalation paths. Build a retry-policy object: `{ maxAttempts, intervalHours, escalateAfterAttempt, notifyCustomerBefore }`. Let the ops lead configure per type.

4. **Decide agent-assignment mode explicitly.** Document the choice: round-robin, skill-based, relationship-sticky, or hybrid. For relationship-sticky: define the fallback chain (primary agent → team lead → pool) and the trigger for fallback (leave, attrition, capacity overflow). Don't let this be an implicit decision that engineering makes in a PR.

5. **Build the exception surface, not just the happy path.** The ops lead's daily view should show: today's exceptions (rescheduled, no-shows, capacity overflows), not today's scheduled calls (those are working fine). The exception dashboard is what prevents the spreadsheet-alongside-the-tool pattern.

6. **Instrument for debugging, not just reporting.** Beyond "how many calls completed this week," the ops lead needs: why did 40 calls fail yesterday? Was it a holiday that wasn't in the system? An agent who didn't log in? A batch of wrong numbers from a bad import? The debugging surface (drill into failures by cause) is more valuable than the summary dashboard.

## How to know you got it right

- **Exception rate under 15%** of scheduled interactions — meaning 85%+ happen as planned without manual intervention.
- **Spreadsheet elimination:** ops stops maintaining a parallel tracker within 2 weeks of launch. If they're still in Google Sheets, the tool is missing something.
- **Outcome taxonomy stability:** after the first month of tuning, new outcome types are added less than once per quarter.
- **Agent utilization above 70%** of available capacity — the scheduling engine isn't leaving dead slots.
- **Retry-to-completion rate:** at least 30% of first-attempt no-answers convert to completed interactions through the retry logic (vs. being manually rescheduled or abandoned).

## Closing

Scheduled human interactions are one of those problems that every ops team thinks is "just a calendar." It's not. It's a state machine with four interacting inputs, an expanding outcome set, and a relationship layer that resists automation. The PM who scopes this correctly saves the ops team from the spreadsheet. The PM who doesn't adds another tool to ignore.
