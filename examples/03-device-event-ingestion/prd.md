# PRD: Device Event Ingestion

**Author:** Platform · **Status:** Draft v0.5 · **Target:** Q3

## 1. Background

We ship IoT sensors to customer sites. Each sensor reports readings and status
events. Today these go into a per-customer S3 bucket and are processed by a nightly
batch job. Customers complain that alerts arrive up to 24 hours late, by which point
a cold-storage failure has already spoiled the stock.

## 2. Proposal

Build a real-time ingestion pipeline. Devices POST events to an HTTP endpoint. Events
are validated, stored, and evaluated against alert rules. Alerts are delivered within
one minute.

## 3. Requirements

- Devices send events to an ingestion endpoint.
- Events are validated against a schema.
- Valid events are stored and made available in the customer dashboard.
- Events are evaluated against configurable alert rules.
- Alerts are delivered by email and SMS.
- Customers can view event history for the last 90 days.
- Ops can replay events for a device over a time range.
- The pipeline should handle our expected volume.

## 4. Event format

JSON. Each event has a device ID, a timestamp, an event type, and a payload. Payload
shape depends on event type.

## 5. Devices

Sensors are battery powered and report every 5 minutes. Some models buffer events
locally when they cannot reach the network and send them in a batch later.

## 6. Alerts

Alert rules are configured per customer. Example: "temperature above 8°C for more
than 10 minutes."

## 7. Authentication

Devices authenticate with an API key issued at provisioning time.

## 8. Out of scope

Device firmware. Device provisioning UI.

## 9. Success criteria

- Alerts arrive within one minute.
- No events lost.
- Customers stop complaining about late alerts.
