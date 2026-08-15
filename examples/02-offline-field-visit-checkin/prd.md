# PRD: Offline Field Visit Check-In

**Author:** Field Ops · **Status:** Draft v1.1 · **Target:** Q4

## 1. Background

Field technicians visit customer sites to service equipment. Today they record
visits on paper and hand the forms in at the end of the week. Forms get lost,
handwriting is unreadable, and Ops has no idea where anyone is during the day.

## 2. Proposal

A mobile app where a technician checks in on arrival at a site, records what they
did, and checks out when leaving. The app must work offline because many sites have
no signal — basements, rural areas, industrial buildings. Data syncs when the device
regains connectivity.

## 3. Requirements

- Technician sees their assigned visits for the day.
- Technician checks in at a site. Check-in records the time and GPS location.
- Technician records work performed, using a checklist plus free-text notes.
- Technician can attach photos.
- Technician checks out. Check-out records the time.
- Ops sees visit status in near real time.
- Visit records sync automatically when connectivity returns.
- Ops can export completed visits for billing.

## 4. Offline behaviour

The app stores visits locally and uploads them when back online. The technician
should not have to think about it.

## 5. Location

Check-in captures GPS coordinates to confirm the technician was on site.

## 6. Photos

Technicians can attach up to 10 photos per visit.

## 7. Out of scope

Route planning and scheduling. Visits are assigned by the existing dispatch system.

## 8. Success criteria

- No more paper forms.
- Ops knows where technicians are.
- Billing goes out faster.
