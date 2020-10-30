# Flexible Public Transportation Services in GTFS

## Overview

Transit operators provide a variety of demand-responsive service which cannot be currently represented in GTFS because the services pick up or drop off riders at a location and/or time chosen by the rider. The following two extensions aim to address this need:

**[GTFS-FlexibleTrips](#gtfs-flexibletrips)** describes flexible services that operate according to some schedule, but which will, on request, during scheduled service, perform certain actions to suit the particular needs of individual riders, such as deviate to a requested address or go to one of a number of predefined stops.

**[GTFS-BookingRules](#gtfs-bookingrules)** provides booking information for rider-requested services using **GTFS-FlexibleTrips**, such as how far in advance booking should occur or a phone number that should be called.

## GTFS-FlexibleTrips

### Goals
This extension describes services that operate according to a schedule, but also include one or more flexible features, such as:

- **Dial-a-ride service**: the vehicle serves a zone where pickups and drop offs are allowed during certain service hours.
- **Route deviation services**: the vehicle serves a fixed route and ordered set of stops, and may detour to pick up or drop off a passenger between stops.
- **Point-to-zone service**: the rider can board at a fixed stop such as a train station, and then alight anywhere within an area, or vice versa. Departures from some locations are scheduled or timed with other services.
- **Point deviation or checkpoint service**: the rider can board at a fixed stop, and then alight anywhere among an unordered list of stops, or the opposite. The driver only serves stops at which a request is made.
- **Hail-and-ride services**: the vehicle stays along a fixed path, but the rider can request a stop anywhere along the path to board or alight.

GTFS-FlexibleTrips describes the times when and locations where flexible service can be requested.

### Overview
This extension

- **Describes locations and groups of locations where riders can request pickup or drop off**: these locations are included in new files called `location_groups.txt` and `locations.geojson`
- **Indicates the times when services are available at on demand locations and the expected travel times**: new fields in `stop_times.txt` provide ranges that equate to service hours and expected travel times on demand trips
- **Clarifies elements of the current specification necessary to inform data consumers of how to interpret the above files and fields added**: new fields in `stop_times.txt` provide ranges that equate to service hours or expected traversal times for locations

### Requirements
None. Extends the core CSV GTFS.

Warning: In order for a trip planner to provide a user with information about how to request many flexible services, data producers must also provide information according to the **GTFS-BookingRules** extension.

### Files extended or added
| File Name | State | Defines |
| --------- | ----- | ------- |
| `location_groups.txt` | Added | Adds location groups. Location groups are groups of stops and GeoJSON locations, which allow predetermined groups of these features to be specified on individual rows of `stop_times.txt`. |
| `locations.geojson` | Added | Adds GeoJSON locations, which are `LineString`, `MultiLineString`, `Polygon` and `MultiPolygon` features that indicate groups of lat/lon coordinates where riders can request either pickup or drop off. |
| `stop_times.txt` | Extended and modified | 

### File Definitions

location_groups.txt (file added)
| Field Name | Details |
| --------- | ------- |
| `location_group_id` | (ID, Required) Defines an identifier for the location group. A location group is a group of stops and/or GeoJSON locations that together indicate various locations where a rider may request pickup or drop off.<br><br>The `location_group_id` belongs to the same namespace as `stop_id` in `stops.txt` and `id` in `locations.geojson`, called “stop locations”.<br><br>Multiple rows in `location_group.txt` can have the same `location_group_id`. | 
| `location_id` | (ID, Optional) Adds this stop location to the location group.<br><br>Refers either to:<br>- a `stop_id` from `stops.txt`<br>- an `id` of a location in `locations.geojson`. |
| `location_group_name` | (Text, Optional) Defines the name of the location group. For one `location_group_id`, either only one entry should have a `location_group_name` or each must contain the same value. |

locations.geojson (file added)
- This file uses a subset of the GeoJSON format, described in [RFC 7946](https://tools.ietf.org/html/rfc7946).
- The `locations.geojson` file must contain a `FeatureCollection`.
- A `FeatureCollection` defines various stop locations where riders may request pickup or drop off.
- Only features with types `LineString`, `MultiLineString`, `Polygon` and `MultiPolygon` can be defined. Individual stops should be defined in `stops.txt` and not in `locations.geojson`.
- Every GeoJSON `Feature` must have an `id`. The `id` belongs to the same namespace as `stop_id` in `stops.txt` and `location_group_id` in `location_groups.txt`, called “stop locations”.
- Every GeoJSON `Feature` must have a `properties` object, with the following keys:

| Key | Value |
| --- | ----- |
| stop_code | (Text, Optional) Short text or a number that identifies the location for riders. These codes are often used in phone-based reservation systems to make it easier for riders to specify a particular location. The stop_code can be the same as id if it is public facing. This field should be left empty for locations without a code presented to riders. |
| stop_name | (Text, Required) Defines the name of the location. The name should be the same, which is used in customer communication, eg. the name of the village where the service stops. |
| stop_desc | (Text, Optional) Description of the location that provides useful, quality information. Can contain a textual representation of the geometry for the location. |
| zone_id | (ID, Optional) Identifies the fare zone for a stop. This field is required if providing fare information using fare_rules.txt, otherwise it is optional. |
| stop_url | (URL, Optional) URL of a web page about the location. This should be different from the `agency.agency_url` and the `routes.route_url` field values. |

stop_times.txt (file extended)
| Field Name | Details |
| ---------- | ------- |
| `stop_id` | [... existing spec...]<br><br>If service is on request, a GeoJSON location or location group can also be referenced:<br>- Field `id` from `locations.geojson`<br>- Field `location_group_id` from `location_groups.txt` |
| `arrival_time` | [... Current definition ..., altering the last sentence “An arrival time must be specified for the first and the last stop time in a trip, unless that stop time refers to a GeoJSON location or location group.” ]<br><br>**Forbidden** when the `stop_id` refers to a GeoJSON location or location group. |
| `departure_time` | [... Current definition ...]<br><br>**Forbidden** when the `stop_id` refers to a GeoJSON location or location group. |
| `start_pickup_dropoff_window`<br><br>and<br><br>`end_pickup_dropoff_window` | (Time, **Conditionally Required**) Service to a GeoJSON location or location group happens within a time frame. Fields `start_pickup_dropoff_window` and `end_pickup_dropoff_window` define the beginning and end of the time frame.<br><br>For times occurring after midnight on the service day, enter the time as a value greater than 24:00:00 in HH:MM:SS for the day on which the trip schedule begins. A single stop time with a `start_pickup_dropoff_window` of 00:00:00 and a `end_pickup_dropoff_window` of 24:00:00 represent service that is available continuously at all times, if service exists on preceding and subsequent days.<br><br>**Forbidden** when the `stop_id` refers to a single stop in `stops.txt`, but required if `stop_id` refers to a location or location group. |
| `pickup_type` | [...]<br><br>GeoJSON locations and location groups:<br>- Cannot have `pickup_type=0`.<br>-Can have `pickup_type=1` or `pickup_type=2`.<br>- Can have `pickup_type=3` only if they are a single GeoJSON LineString, since it implies that the passenger is able to “coordinate with the driver” for pick up, and therefore the driver has to serve all of the location groups to see the potential passenger waiting on the sidewalk and hailing him/her. It implies that the vehicle will entirely serve the path defined by the LineString, in the order in which it is defined in the GeoJSON. |
| `drop_off_type` | [...]<br><br>GeoJSON locations and location groups:<br>- Cannot have `drop_off_type=0`.<br>- Can have `drop_off_type=1`, `drop_off_type=2` or `drop_off_type=3`. |
| `mean_duration_factor`<br><br>and<br><br>`mean_duration_offset` | (Float, **Conditionally Required**) Fields `mean_duration_factor` and `mean_duration_offset` allow the estimation of how fast a rider’s trip will take during the use of service in a GeoJSON location or location group. Below is a description of the calculation data consumers are expected to make, when estimating the travel time within a GeoJSON location or location group.<br><br>Use `mean_duration_factor` and `mean_duration_offset` to calculate the `MeanTravelDuration` based on the `DrivingDuration`. The `MeanTravelDuration` is given by the following formula:<br><br>`MeanTravelDuration = mean_duration_factor × DrivingDuration + mean_duration_offset`<br><br>The `DrivingDuration` is the time it would take in a car to travel the distance being calculated for flexible service, as defined by the data consumer. The `MeanTravelDuration` is the calculated average time one expects the service to travel the same trip.<br><br>The `MeanTravelDuration` may be calculated for the time and the day of the trip to take into account traffic; in other words the consumer is expected to know that `DrivingDuration` is dynamic. Producers should thus provide values which reflect increases in `DrivingDuration` due to additional pickups and drop offs beyond that of the passenger. A downtown TNC will likely always have a `mean_duration_factor` of 1, with or without traffic, since it goes with the flow. But a shared service can have a factor of 2 or more if many additional pickups and drop offs are expected. `mean_duration_offset` can be utilized to increase travel times of shorter trips relatively more than times for longer trips.<br><br>**Conditionally Required**:<br>- **Required** when `stop_id` refers to a GeoJSON location or location group.<br>- **Forbidden** otherwise. |
| `safe_duration_factor`<br><br>and<br><br>`safe_duration_offset` | (Float, **Conditionally Required**) Fields `safe_duration_factor` and `safe_duration_offset` allow calculation of the `SafeTravelDuration` based on the `DrivingDuration`. The `SafeTravelDuration` is a way for data consumers to calculate the likely “worst case scenario” for a rider when estimating an on demand trip.<br><br>The `SafeTravelDuration` is given by the following formula:<br><br>`SafeTravelDuration = safe_duration_factor × DrivingDuration + safe_duration_offset`<br><br>The `SafeTravelDuration` should reflect the longest time one expects the service to require, in 95% of trips. A rider can account for this much time to use the service, and have a very good chance of having the trip take this much time or less.<br><br>**Conditionally Required**:<br>- **Required** when `stop_id` refers to a GeoJSON location or location group.<br>- **Forbidden** otherwise. |

## GTFS-BookingRules

### Goals
Many flexible services included in the **GTFS-FlexibleTrips** extension must be booked in advance and/or by using a phone or the internet. This extension provides the rider with information about how to request service.

### Requirements
[None. Extends the core CSV GTFS.]

### Files extended or added
| File Name | State | Defines |
| --------- | ----- | ------- |
| `stop_times.txt` | Extended | Adds links to booking rules. |
| `booking_rules.txt` | Added | Defines the booking rules. |

### Table Definitions
stop_times.txt (file extended)
| Field Name | Details |
| ---------- | ------- |
| `pickup_booking_rule_id` | (ID, Optional) Defines the boarding booking rules for this stop time, overwriting the default trip booking rule.<br><br>It is strongly recommended to have a booking rule defined (either at the trip or the stop time level) when `pickup_type=2`. |
| `drop_off_booking_rule_id` | (ID, Optional) Defines the alighting booking rules for this stop time, overwriting the default trip booking rule.<br><br>It is strongly recommended to have a booking rule defined (either at the trip or the stop time level) when `drop_off_type=2`. |

booking_rules.txt (file added)
| Field Name | Details |
| ---------- | ------- |
| `booking_rule_id` | (ID, **Required**) Defines an ID for the booking rule, which will be referenced from `stop_times.txt`. |
| `booking_type` | (Enum, **Required**) Defines how much in advance the booking can be made. Value are:<br>- 0: Real-time booking only<br>- 1: Up to same-day booking, with advance notice<br>- 2: Up to prior day(s) booking |
| `prior_notice_duration_min` | (Integer, **Conditionally Required**) Minimum number of minutes of advance time necessary before travel to make a booking request.<br><br>**Conditionally Required**:<br>(The timing must be provided by either a duration or a last time & day)<br>- **Required** for up-to-same-day booking (`booking_type=1`).<br>- **Forbidden** otherwise. |
| `prior_notice_duration_max` | (Integer, Conditionally Optional) Maximum number of minutes of advance time necessary before travel to make a booking request.<br><br>Conditionally Optional:<br>- Optional for up-to-same-day booking (`booking_type=1`)<br>**Forbidden** otherwise.
| `prior_notice_last_day` | (Integer, **Conditionally Required**) Latest day on which a booking can be made. Defined as an offset, so the number of service days in advance of the booking.<br><br>Example: “Ride must be booked 1 day in advance before 5PM” will be encoded as `prior_notice_last_day=1`.<br><br>**Conditionally Required**:<br>(The timing must be provided by either a duration or a last time & day)<br>- **Required** for up-to-prior-day booking (`booking_type=2`).<br>- **Forbidden** otherwise. |
| `prior_notice_last_time` | (Time, **Conditionally Required**) Latest time of day on the last day on which a booking can be made. The timezone used is the one defined by `agency.agency_timezone`.<br><br>Example: “Ride must be booked 1 day in advance before 5PM” will be encoded as `prior_notice_last_time=17:00:00`.<br><br>**Conditionally Required**:<br>- **Required** if `prior_notice_last_day` is defined.<br>- **Forbidden** otherwise. |
| `prior_notice_start_day` | (Integer, Conditionally Optional) Earliest day on which a booking can be made. Defined as an offset, so the number of service days in advance of the booking.<br><br>Example: “Ride can be booked at the earliest one week in advance at midnight” will be encoded as `prior_notice_start_day=7`.<br><br>Conditionally Optional:<br>- Optional for up-to-prior-day booking (`booking_type=2`).<br>- Optional for up-to-same-day booking (`booking_type=1`) if `prior_notice_duration_max` is empty.<br>- **Forbidden** otherwise.
| `prior_notice_start_time` | (Time, **Conditionally Required**) Earliest time of the earliest day at which a booking can be made. The timezone used is the one defined by `agency.agency_timezone`.<br><br>Example: “Ride can be booked at the earliest one week in advance at midnight” will be encoded as `prior_notice_start_time=00:00:00`.<br><br>**Conditionally Required**:<br>- **Required** if `prior_notice_start_day` is defined.<br>- **Forbidden** otherwise. |
| `prior_notice_service_id` | (ID from `calendar.txt`, Optional) When `prior_notice_start_day` is used, `prior_notice_service_id` defines a subset of days that count towards the `prior_notice_start_day`.<br><br>Example: If empty, `prior_notice_start_day=2` will be two calendar days in advance. If defined as a `service_id` containing only business days (weekdays without holidays), `prior_notice_start_day=2` will be two business days in advance. |
| `message` | (Text, Optional) Message to passengers utilizing service at a `stop_time` with this boarding rule. This message appears when the rider is booking a demand responsive pickup and drop off.<br><br>The message is meant to provide minimal information to be transmitted within a user interface about the action a rider must take in order to utilize the service. This text is expected to be a link within user interfaces, whether to a phone number, a url, or a deep link to an app. |
| `pickup_message` | (Text, Optional) Identical to `message` but used when riders have a demand response pickup only. |
| `drop_off_message` | (Text, Optional) Identical to `message` but used when riders have a demand response drop off only.
| `phone_number` | (Phone number, Optional) Phone number to make the booking. Must follow the  E.123 standard (e.g. “`+1 503 238 7433`” for TriMet). 
| `info_url` | (URL, Optional) The `info_url` field contains a URL providing human readable information about the booking rule. |
| `booking_url` | (URL, Optional) If a rider can book trips according to this booking rule through an online interface or app, the link to that reservation system or app download page is the `booking_url`. |
