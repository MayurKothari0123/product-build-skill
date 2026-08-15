# Spec: Patient Notes

> **Fixture note.** This document never states who may read a patient's notes. It
> names three roles and describes writing, never reading. FlowBreaker must raise a
> `critical` permission question. It must **not** treat the unwritten access rule as
> presumably fine because the domain "obviously" implies it — in a clinical system,
> "obviously" is how access-control defects ship.
>
> Note the trap: §5's audit requirement makes the spec *feel* security-conscious.
> Logging who read something is not the same as deciding who may.

## 1. Problem

Clinicians record consultation notes on paper and dictate them for transcription.
Notes take up to three days to reach the patient record, so the next clinician to see
that patient often works without them.

## 2. Users

Doctors, nurses, and administrative staff at the practice.

## 3. Requirements

- A clinician can create a note against a patient record during or after a
  consultation.
- Notes support free text and structured vitals.
- A clinician can amend their own note within 24 hours of creation.
- After 24 hours, amendments create an addendum rather than editing the original.
- Notes are timestamped and attributed to their author.
- Administrative staff can attach scanned documents to a patient record.
- A patient record shows all notes in reverse chronological order.

## 4. Data

Notes are stored against the patient record. Vitals are stored as structured fields
for trend charting.

## 5. Audit

All access to patient records is logged with the user, timestamp, and record
accessed.

## 6. Out of scope

Prescribing. Referrals. Patient-facing access to their own notes.

## 7. Success criteria

- Notes available in the record immediately.
- Transcription costs eliminated.
