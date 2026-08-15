> FlowBreaker · review · prd.md · 2026-08-15
> Status: complete

# Problem Brief

## PROB-001 — HR's manual reconciliation
**Problem:** Leave is requested and approved through informal channels, so there is
no system of record. HR reconstructs it manually into a spreadsheet each month —
about two days of work, frequently wrong.
**Who has it:** HR operations (2 people, monthly). Downstream: anyone relying on
leave balances being correct.
**Today they:** Read WhatsApp and email threads and transcribe them by hand.
**Business objective:** Eliminate ~2 days/month of HR effort and make balances
trustworthy.
**Confidence:** evidence
**Source:** §1 — "HR reconciles everything manually into a spreadsheet at the end of
each month, which takes roughly two days and is frequently wrong."

## PROB-002 — Employees cannot see request status
**Problem:** After sending a request, an employee has no way to know whether it was
seen, is being considered, or was decided. So they chase.
**Who has it:** All employees requesting leave.
**Today they:** Follow up over WhatsApp until someone replies.
**Business objective:** Stated only as a success criterion (§8), never as a problem.
**Confidence:** inference
**Reasoning:** §8 lists "employees stop chasing managers" as success, which implies
chasing is a current problem — but §1 describes the problem entirely from HR's side.
The document is written around HR's pain and treats the employee's as incidental.

## Jobs to be done
- When I need time off, I want to know my request was received and will be decided,
  so I can plan without chasing anyone. *(inference)*
- When my team member requests leave, I want to see the impact on team coverage
  before deciding, so I don't approve two overlapping absences. **Not addressed
  anywhere in the source** → Q-007. *(inference)*
- When the month closes, I want leave data to be already correct, so I don't
  reconstruct it. *(evidence, §1)*

## Non-goals
- Payroll integration *(evidence, §7)*
- **Everything else is unstated.** The document names one non-goal. No statement on
  half-days, carry-over, public holidays, multi-approver chains, or delegation —
  each of which a reader could reasonably assume is in scope. → Q-008

## Business objective
Reduce HR reconciliation cost and produce a trustworthy record of leave.
**Not stated:** whether the goal is also to *change* approval behaviour (faster
decisions, consistent policy) or merely to *record* it. These imply different
products. → Q-003

---

## My understanding — please confirm or correct

This is primarily an **HR data-integrity project**, not an employee-experience
project. The core problem is that leave has no system of record, so HR rebuilds it
by hand each month. Faster approvals and less chasing are hoped-for side effects,
not the target.

**If that is wrong — if the real goal is to speed up approvals — the scope changes
materially**, because §5's HR-review step (which contradicts §2, see Q-005) becomes
the bottleneck you are trying to remove rather than a control you are trying to
preserve.

**Q:** Is the primary objective HR's record accuracy, or the speed and consistency
of approval decisions?
