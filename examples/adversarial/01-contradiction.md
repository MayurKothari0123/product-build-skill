# Spec: Document Publishing

> **Fixture note.** FlowBreaker must report the §3/§6 conflict as `contradictory`
> and ask which is correct. It must **not** silently pick one and build on it.
> The conflict is deliberately undramatic — §6 reads as leftover text from an
> earlier draft, which is how this defect actually appears in real documents.

## 1. Problem

Writers draft documents in the tool but have no way to publish them to the public
help centre. They export to HTML and email it to the web team, who paste it in. This
takes two days and introduces formatting errors.

## 2. Users

Writers (12), who draft and publish. Editors (3), who review for style and accuracy.

## 3. Publishing flow

A writer finishes a draft and clicks Publish. The document goes live on the help
centre immediately. The writer sees a confirmation with the public URL.

## 4. Requirements

- Writers can draft documents.
- Writers can publish a document.
- Published documents appear on the public help centre.
- Writers can unpublish a document, removing it from the help centre.
- Editors can leave comments on a draft.
- Published documents show a last-updated date.

## 5. Versioning

Each publish creates a version. Writers can view version history and restore a
previous version.

## 6. Review

All documents require editor approval before publishing. An editor reviews the draft
and either approves it, which publishes it, or returns it to the writer with
comments.

## 7. Out of scope

Translation. Scheduled publishing.

## 8. Success criteria

- Publishing takes under 5 minutes.
- No formatting errors introduced in publishing.
