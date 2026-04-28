# Notes from EO Alert Suppression Sync — April 2026

Hey team, capturing notes from today's sync on the suppression improvements work. This is rough so bear with me.

## Background / Why we're doing this

So customers have been complaining for a while that EO creates too much noise. Basically when the same underlying issue fires 50 events in a row, each one goes through the routing rules and potentially creates incidents. Ops teams are drowning in duplicate alerts. We've heard this from at least 3 enterprise customers in the last quarter and it came up in the last NPS batch too.

The current workaround is customers manually write suppression rules, but it's a pain to set up and most customers don't know how to tune them properly. Some customers just turn off alerting for entire services to stop the noise which is obviously terrible.

## What we want to build

The rough idea is smarter deduplication / suppression at the EO layer. When EO sees a pattern of similar events within a time window, it should be able to either:
- Collapse them into one "representative" event
- Suppress the duplicates after the first one goes through
- Maybe let customers configure a threshold (like "if same event type fires > 5 times in 10 minutes, suppress")

We should probably also show customers what's being suppressed so they don't think alerts are just disappearing. Some kind of suppression log or summary would be useful.

Being able to set this per-service or per-orchestration would be really important, global suppression settings feel too blunt.

It would be great if customers could get a weekly digest or summary of what got suppressed so compliance teams can still see that alerts happened even if they were suppressed.

The API should expose suppression data too so customers can pull it into their SIEM or dashboards.

## Who this is for

Primarily SRE teams and NOC operators who are managing high-volume services. Also compliance officers at regulated customers who need to know even suppressed alerts are being tracked.

## What we're not trying to do

We're definitely not trying to replace the existing manual suppression rules — this is additive. But I'm not sure about the edge cases here, need to think through more.

## Timeline thoughts

This feels like something we could do for EA with a limited set of customers who have the highest alert volume. GA would be broader rollout with the digest feature.

## Open questions

- How long do we retain suppression event data?
- Does this interact with the existing dedup logic at the ingest layer or is EO-level suppression separate?
- What's the right UI surface for the suppression config? Inside the rule editor or a separate settings page?
- Do we need approval from the data retention team for the suppression logs?
