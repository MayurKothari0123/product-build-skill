# PRD: Leave Request & Approval

**Author:** People Ops · **Status:** Draft v0.3 · **Target:** Q3

## 1. Background

Employees currently request leave over WhatsApp or email. Managers approve
informally. HR reconciles everything manually into a spreadsheet at the end of each
month, which takes roughly two days and is frequently wrong.

## 2. Proposal

Build a leave request module in the internal portal. An employee submits a leave
request. The employee's line manager approves or rejects it. Approved leave is
deducted from the employee's balance and appears on the team calendar.

## 3. Requirements

- Employees can submit a leave request specifying start date, end date and leave type.
- The system shows the employee their remaining balance.
- Managers see pending requests for their team.
- Managers can approve or reject a request. Rejection requires a reason.
- The employee is notified of the decision.
- Approved leave is deducted from the balance.
- Approved leave appears on the team calendar.
- HR can export a monthly report of all leave taken.
- Leave balances reset annually.
- The approval process should be fast.

## 4. Leave types

Annual, sick, unpaid. Annual leave is limited by the employee's balance. Sick leave
does not require a balance. Unpaid leave does not affect the balance.

## 5. Approval

HR reviews and approves all leave requests to ensure policy compliance before the
leave is confirmed.

## 6. Notifications

Managers are notified by email when a request is submitted. Employees are notified
when a decision is made.

## 7. Out of scope

Integration with payroll.

## 8. Success criteria

- HR no longer reconciles leave manually.
- Employees stop chasing managers for approval.
- Intuitive and easy to use.
