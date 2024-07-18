---
permalink: /event_template_best_practices/
title: "Event Template Best Practices"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---
Event templates are a great way to define and enforce consistency in how events are captured, however there is an art to defining event templates. Below are a few techniques to help operators and scientists can get the most out of event templates.

### Think about the output... 
Most users are going to interact with the event data as a spreadsheet/table. This is an important consideration when designing event templates. The event templates should produce a dataset that efficiently populates a spreadsheet without excessive largely empty columns. 

### Use concise event_option names
It's possilble to define event options with long descriptive names but not recommended. Long descriptive event option names will clutter the event template form and the exported data. Try to keep event option names short and concise.

### Reuse event_option names to minimize the number of columns in the output.
Use event_value as a top-level category and a common event_option and a sub category.

### Use the Free Text field
Some events will require the user to provide a textual description of the observation.  Rather than adding this information to a "description" event option, use the built in free_form text field. The free_form field can be set as required to ensure users provide some type of description prior to submission.

### Examples 
Below is a list of Event templates that utilize the recommended techniques:
```
event_value -> event_option: value, ...
-----------------------------------------------------------------------------
Multibeam -> Status: Start of Survey, System: EM712
Multibeam -> Status: Setting Change, System: EM712 (Free_form field required)
Multibeam -> Status: End of line, System: EM712
Multibeam -> Status: Start of line, System: EM712
Multibeam -> Status: End of Survey, System: EM712

Vehicle -> Milestone: Off Deck
Vehicle -> Milestone: Descending
Vehicle -> Milestone: On Bottom
Vehicle -> Milestone: Off Bottom
Vehicle -> Milestone: On Surface
Vehicle -> Milestone: On Deck

Problem -> System: Vehicle (Free_form field required)
Problem -> System: EM712 (Free_form field required)
Problem -> System: CTD (Free_form field required)
Problem -> System: ADCP (Free_form field required)
Problem -> System: Winch (Free_form field required)

Observation -> Type: Geology (Free_form field required)
Observation -> Type: Biology (Free_form field required)
Observation -> Type: Anthropogenic (Free_form field required)

Sample -> Type: Geology, Storage Location: Box 1A (Free_form field required)
Sample -> Type: Biology, Storage Location: Box 1C (Free_form field required)
Sample -> Type: Anthropogenic, Storage Location: Box 1B (Free_form field required)
```

These event templates are optimized for query. Let's looks at some examples:
- If a user wants all the vehicle-related events they simply filter for `Vehicle`. This will return all the `Vehicle` events as well as the `Problem` events where `System` equals `Vehicle`.
- If a user wants all the Geology-related events they simply filter for `Geology`. This will return all the `Sample` and `Observation` events where `Type` equals `Geology`.

The output from these event templates is also optimized.  The event_option names `Type` and `System` are used across multiple templates. In the exported output the `System` column will be populated for events created from 10 event templates and the `Type` column will be populated for event created from 6 event templates. Using unique event_option names would result in an output with additional, largely empty columns.
