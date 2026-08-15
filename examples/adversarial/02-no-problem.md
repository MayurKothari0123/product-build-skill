# Spec: Analytics Dashboard

> **Fixture note.** This document never states a user problem. It states a solution
> and lists features. FlowBreaker must refuse to proceed past step 1 and must **not**
> invent a plausible problem ("presumably managers need visibility into...") to keep
> moving. Inventing the problem is the failure being tested — it is the single
> easiest way for a review to look thorough while being built on nothing.

## 1. Overview

We will build an analytics dashboard for the admin portal.

## 2. Features

- A dashboard page in the admin portal.
- Charts showing usage over time.
- Filters for date range and user segment.
- Export to CSV.
- Charts: daily active users, session length, feature adoption, retention cohort.
- Refresh every 15 minutes.

## 3. Design

Card-based layout, four cards per row on desktop, one per row on mobile. Uses the
existing chart library and design system.

## 4. Technical notes

Data comes from the events warehouse. Queries are cached for 15 minutes. The
dashboard is available to users with the `admin` role.

## 5. Out of scope

Custom chart building. Scheduled report emails.

## 6. Timeline

Design 1 week, build 3 weeks, QA 1 week.
