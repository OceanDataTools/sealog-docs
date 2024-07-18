---
permalink: /core_concepts/
title: "Core Concepts"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

To better understand how to best leverage Sealog during a cruise or vehicle lowering it is important to understand the underlying concepts and terminology.

## Events
An ‘event’ is any scientific or operational observation not already directly captured by other data logging systems. 

Sealog events are comprised of a timestamp, author, a high-level value, optional free-form text and an optional list of related observational information.

Examples of scientific events include in-situ biological and geological observations.  Examples of operational events are things such as cruise/dive milestones (i.e. ‘on-station’, ‘vehicle-in-water’, ‘vehicle-on-bottom’, ‘start-of-survey’, etc).

An example of an event that would have a list of related observational information would be the event to document a sample collection.  This event should not only capture when a sample was collected but should also document important data points such as it’s unique sample id, the type of sample and where it was stored on the vehicle.

### Event Comments
Event comments are a special related observational information included in all event records. It's a space for users to add additional information after the event is submitted.  Event comments can be used for:
- Recording that an observation is incorrect
- Additional observations not included in the original event submission
- Preliminary notes on the importance of an event

## Ancillary Data
In addition to the direct observational data it’s also important to capture ancillary data such as vessel/vehicle position. The Sealog data model can associated multiple ancillary data points with an event. Examples of ancillary data added to events include: realtime ship/vehicle navigation, frame capture filenames, realtime sensor data, etc.

Ancillary data points can be added at the time the event is created or as part of a post-processing workflow.  This decoupled approach adds some complexity to the data model but also adds the flexibility to gracefully deal with sensor failures and datasets not available at the time the event is recorded.

## Cruises and Lowerings
Sealog event data is not internally organized by cruise or by lowering.  Cruise and lowering information are stored as separate records.  These records include an ID, start time, stop time, and other metadata such as location and description.  These records are used when reviewing or exporting data.  When events for a cruise or lowering accessed, Sealog uses the information stored in the cruise and lowering records to extract events based on the event timestamps.

Likewise lowerings records do not include cruise information. The list of lowering for a given cruise is determined by querying the list of lowerings with start/stop times that occur within a cruise record’s start/stop times.

## Event Templates
Event templates are records that define the related observational data points that should be captured when submitting a particular event type.  In the case of the sample event, the event template includes a list of additional data to capture such as the sample ID, sample type and storage location.