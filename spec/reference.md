# GTFS-Flex v2
Flexible public transit services in GTFS.

## Overview

GTFS-Flex v2 is composed of two extensions that aim to model the variety of demand responsive services that do not always follow the same fixed stops. The following two extensions address this need:

| Extension name | Description |
| ----- | ----- |
| **[GTFS-FlexibleTrips](#gtfs-flexibletrips)** | Flexible services that operate according to some schedule but are responsive to on-demand requests of individual riders. |
| **[GTFS-BookingRules](#gtfs-bookingrules)** | Booking information for rider-requested services using **GTFS-FlexibleTrips**, such as how far in advance booking should occur or a phone number that should be called. |

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

- **Describes locations and groups of locations where riders can request pickup or drop off**: these locations are included in a new file called `locations.geojson` and the existing Fares v2 file `stop_areas.txt`.
- **Indicates the times when services are available at on-demand locations and the expected travel times**: new fields in `stop_times.txt` provide ranges that equate to service hours and expected travel times on demand trips
- **Clarifies elements of the current specification necessary to inform data consumers of how to interpret the above files and fields added**: new fields in `stop_times.txt` provide ranges that equate to service hours or expected traversal times for locations

### Requirements
None. Extends the GTFS.

In order for a trip planner to provide a user with information about how to request many flexible services, data producers must also provide information according to the **GTFS-BookingRules** extension.

### Files extended or added
| File name | State | Defines |
| --------- | ----- | ------- |
| `locations.geojson` | Added | (Optional file) Adds GeoJSON locations, which are `Polygon` and `MultiPolygon` features that indicate groups of lat/lon coordinates defining zones where riders can request either pickup or drop off. |
| `stop_areas.txt` | Modified | (Optional file) Modifies file to allow grouping of GeoJSON locations and/or stops which allow predetermined groups of these features to be specified on individual rows of `stop_times.txt`. |
| `stop_times.txt` | Extended and modified |

### File definitions

#### locations.geojson (file added)
- This file uses a subset of the GeoJSON format, described in [RFC 7946](https://tools.ietf.org/html/rfc7946).
- The `locations.geojson` file must contain a `FeatureCollection`.
- A `FeatureCollection` defines various stop locations where riders may request pickup or drop off.
- Every GeoJSON `Feature` must have an `id`. The `id` belongs to the same namespace as `stop_id` in `stops.txt` and `location_group_id` in `location_groups.txt`, called “stop locations”.
- Every GeoJSON `Feature` should have objects and associated keys according to the table below:

| Field Name | Presence | Type | Description |
| ----- | ----- | ----- | ----- |
| -&nbsp;`type` | **Required** | String | `"FeatureCollection"` of locations. |
| -&nbsp;`features` | **Required** | Array | Collection of `"Feature"` objects describing the locations. |
| &nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`type` | **Required** | String | `"Feature"` |
| &nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`id` | **Required** | String| Location ID belonging to the same namespace as `stops.stop_id`. Therefore, it is forbidden to define an `id` from `locations.geojson` with the same value as a `stops.stop_id`.<br><br>By default, every `id` from `locations.geojson` belongs to a `location_groups.location_group_id` of the same value.|
| &nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`properties` | **Required** | Object | Location property keys. |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`stop_name` | Optional | String | Indicates the name of the location as displayed to riders. |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`stop_desc` | Optional | String | Meaningful description of the location to help orient riders. |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`zone_id` | **Conditionally Required** | String | Identifies the fare zone for a stop.<br><br>Conditionally required:<br>- **Required** if `fare_rules.txt` is defined.<br>- Optional otherwise.|
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`stop_url` | Optional | URL |  URL of a web page about the location.<br><br>If provided, the URL should be different from the `agency.agency_url` and the `routes.route_url` field values. |
| &nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`geometry` | **Required** | Object | Geometry of the location. |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`type` | **Required** | String | Must be of type:<br>-&nbsp;`"Polygon"`<br>-&nbsp;`"MultiPolygon"` |
| &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;\-&nbsp;`coordinates` | **Required** | Array | Geographic coordinates (latitude and longitude) defining the geometry of the location. |

#### areas.txt (no change)
| Field Name | Type | Presence | Description |
| ---------- | ---- | ------------ | ----------- |
| `area_id` | Unique ID	| **Required** | Identifies an area. Must be unique in areas.txt.
| `area_name` | Text | **Optional** | The name of the area as displayed to the rider.

#### stop_areas.txt (file modified)

| Field Name | Type | Presence | Description |
| ---------- | ---- | ------------ | ----------- |
| `area_id` | Foreign ID referencing `areas.area_id` | **Required** | Identifies an area to which one or multiple stop_ids belong. The same `stop_id` may be defined in many `area_id`s.<br><br>May also identify a group of stops and/or GeoJSON locations that together indicate locations where a rider may request pickup or drop off.<br><br>It is forbidden to define an `area_id` with the same value as a `stop_id` or `id` from `locations.geojson`. | 
| `stop_id` | ID referencing `stops.stop_id` or `id` from `locations.geojson` | **Required** | Identifies a stop or GeoJSON location. If a station (i.e. a stop with `stops.location_type=1`) is defined in this field, it is assumed that all of its platforms (i.e. all stops with `stops.location_type=0` that have this station defined as `stops.parent_station`) are part of the same area. This behavior can be overridden by assigning platforms to other areas. |

#### stop_times.txt (file extended)

| Field Name | Type | Presence | Description |
| ---------- | ---- | -------- | ----------- |
| `stop_id` | ID referencing `stops.stop_id`, `location_groups.location_group_id`, or `id` from `locations.geojson`  | **Required** | Identifies the serviced stop. All stops serviced during a trip must have a record in `stop_times.txt`. Referenced locations must be stops, not stations or station entrances. A stop may be serviced multiple times in the same trip, and multiple trips and routes may service the same stop.<br><br>If service is on demand, a GeoJSON location or location group can be referenced:<br>-&nbsp;`id` from `locations.geojson`<br>-&nbsp;`location_groups.location_group_id` |
| `stop_sequence` | Non-negative integer  | **Required** | Order of stops for a particular trip. The values must increase along the trip but do not need to be consecutive. <br><br> *Example: The first location on the trip could have a `stop_sequence`=`1`, the second location on the trip could have a `stop_sequence`=`23`, the third location could have a `stop_sequence`=`40`, and so on.* <br><br> Travel within the same location group or GeoJSON location requires two records in `stop_times.txt` with the same `stop_id` and consecutive values of `stop_sequence`. |
| `arrival_time` | Time | **Conditionally Required** |Arrival time at a specific stop for a specific trip on a route. If there are not separate times for arrival and departure at a stop, enter the same value for `arrival_time` and `departure_time`.<br><br> Scheduled stops where the vehicle strictly adheres to the specified arrival and departure times are timepoints. If this stop is not a timepoint, it is recommended to provide an estimated or interpolated time. If this is not available, `arrival_time` can be left empty. Further, indicate that interpolated times are provided with `timepoint=0`. If interpolated times are indicated with `timepoint=0`, then time points must be indicated with `timepoint=1`. Provide arrival times for all stops that are timepoints.<br><br>**Conditionally Required**:<br>-&nbsp;**Required** for the first and the last stop in a trip.<br>-&nbsp;**Forbidden** when `stop_times.start_pickup_drop_off_window` or `stop_times.end_pickup_drop_off_window` are defined.|
| `departure_time` | Time | **Conditionally Required** | Departure time from a specific stop for a specific trip on a route.  If there are not separate times for arrival and departure at a stop, enter the same value for `arrival_time` and `departure_time`. See the `arrival_time` description for more details about using timepoints correctly.<br><br>The `departure_time` field should specify time values whenever possible, including non-binding estimated or interpolated times between timepoints.<br><br>**Conditionally Required**:<br>-&nbsp;**Required** for the first and the last stop in a trip.<br>-&nbsp;**Forbidden** when `stop_times.start_pickup_drop_off_window` or `stop_times.end_pickup_drop_off_window` are defined.|
| `start_pickup_drop_off_window` | Time | **Conditionally Required** | Time that on-demand service becomes available in a GeoJSON location, location group or stop.<br><br>**Conditionally Required**:<br>-&nbsp;**Required** if `stop_times.stop_id` refers to `location_groups.location_group_id` or `id` from `locations.geojson`. <br>-&nbsp;**Forbidden** if `stop_times.arrival_time` or `stop_times.departure_time` are defined.  |
| `end_pickup_drop_off_window` | Time | **Conditionally Required** | Time that on-demand service ends in a GeoJSON location, location group or stop.<br><br>**Conditionally Required**:<br>-&nbsp;**Required** if `stop_times.stop_id` refers to `location_groups.location_group_id` or `id` from `locations.geojson`. <br>-&nbsp;**Forbidden** if `stop_times.arrival_time` or `stop_times.departure_time` are defined. |
| `pickup_type` | Enum | **Conditionally Forbidden** | Indicates pickup method. Valid options are:<br><br> `0` or empty - Regularly scheduled pickup.<br> `1` - No pickup available.<br> `2` - Must phone agency to arrange pickup.<br> `3` - Must coordinate with driver to arrange pickup.<br><br>  **Conditionally Forbidden**: <br>- `pickup_type=0` **forbidden** for `stop_times.stop_id` referring to `location_groups.location_group_id` or `id` from `locations.geojson`.<br> - `pickup_type=3` **forbidden** for `location_groups.location_group_id` or `locations.geojson` that are not a single `"LineString"`.<br> - Optional otherwise. |
| `drop_off_type` | Enum | **Conditionally Forbidden** | Indicates drop off method. Valid options are:<br><br> `0` or empty - Regularly scheduled drop off.<br> `1` - No drop off available.<br> `2` - Must phone agency to arrange drop off.<br> `3` - Must coordinate with driver to arrange drop off.<br><br> **Conditionally Forbidden**:<br> - `drop_off_type=0` **forbidden** for `stop_times.stop_id` referring to `location_groups.location_group_id` or `id` from `locations.geojson`.<br> - Optional otherwise.
| `mean_duration_factor`<br><br>and<br><br>`mean_duration_offset` | Float | **Conditionally Forbidden** | Together, `mean_duration_factor` and `mean_duration_offset` allow an estimation of the duration a rider’s trip will take, in minutes, using the on-demand service in a GeoJSON location or location group. <br><br> Data consumers are expected to use `mean_duration_factor` and `mean_duration_offset` to make the following calculation:<br><br>`MeanTravelDuration = mean_duration_factor × DrivingDuration + mean_duration_offset`<br><br>Where `DrivingDuration` is the time it would take in a car to travel the distance being calculated for the on-demand service, and `MeanTravelDuration` is the calculated average time one expects to travel the same trip using the on-demand service.<br><br>The `MeanTravelDuration` may be calculated for the time and the day of the trip to take into account traffic; in other words the consumer is expected to know that `DrivingDuration` is dynamic. Producers should thus provide values that reflect increases in `DrivingDuration` due to additional pickups and drop offs beyond that of the passenger. A downtown TNC will likely always have a `mean_duration_factor` of 1, with or without traffic, since it goes with the flow. But a shared service can have a factor of 2 or more if many additional pickups and drop offs are expected. `mean_duration_offset` can be utilized to increase travel times of shorter trips relatively more than times for longer trips.<br><br>While traveling through undefined space between GeoJSON locations or location groups, it is assumed that:<br><br>`MeanTravelDuration = DrivingDuration`<br><br>**Conditionally Forbidden**:<br>- **Forbidden** if `stop_times.stop_id` does not refer to a `location_groups.location_group_id` or an `id` from `locations.geojson`.<br>- Optional otherwise. |
| `safe_duration_factor`<br><br>and<br><br>`safe_duration_offset` | Float | **Conditionally Forbidden** | Together, `safe_duration_factor` and `safe_duration_offset` allow an estimation of the longest amount of time a rider can expect the on-demand service in a GeoJSON location or location group may require, in minutes, for 95% of trips. <br><br> Data consumers are expected to use `safe_duration_factor` and `safe_duration_offset` to make the following calculation:<br><br> `SafeTravelDuration = safe_duration_factor × DrivingDuration + safe_duration_offset`<br><br> Where `DrivingDuration` is the time it would take in a car to travel the distance being calculated for the on-demand service, and `SafeTravelDuration` is the longest amount of time a rider can expect the on-demand service in a GeoJSON location or location group may require.<br><br>**Conditionally Forbidden**:<br>- **Forbidden** if `stop_times.stop_id` does not refer to a `location_groups.location_group_id` or an `id` from `locations.geojson`.<br>- Optional otherwise. |

## GTFS-BookingRules

### Goals
Many flexible services included in the **GTFS-FlexibleTrips** extension must be booked in advance and/or by using a phone or the internet. This extension provides the rider with information about how to request service.

### Requirements
None. Extends the GTFS.

### Files extended or added
| File Name | State | Defines |
| --------- | ----- | ------- |
| `stop_times.txt` | Extended | Adds links to booking rules. |
| `booking_rules.txt` | Added | Defines the booking rules. |

### Table Definitions
#### stop_times.txt (file extended)

| Field Name | Type | Presence | Description |
| ---------- | ---- | -------- | ----------- |
| `pickup_booking_rule_id` | ID referencing `booking_rules.booking_rule_id` | Optional | Identifies the boarding booking rule at this stop time.<br><br>Recommended when `pickup_type=2`. |
| `drop_off_booking_rule_id` | ID referencing `booking_rules.booking_rule_id` | Optional | Identifies the alighting booking rule at this stop time.<br><br>Recommended when `drop_off_type=2`. |

#### booking_rules.txt (file added)

| Field Name | Type | Presence | Description |
| ---------- | ---- | -------- | ----------- |
| `booking_rule_id` | ID | **Required** | Identifies the rule. |
| `booking_type` | Enum | **Required** | Indicates how far in advance booking can be made. Valid options are:<br><br>`0` - Real time booking.<br>`1` - Up to same-day booking with advance notice.<br>`2` - Up to prior day(s) booking. |
| `prior_notice_duration_min` | Integer | **Conditionally Required** | Minimum number of minutes before travel to make the request.<br><br>**Conditionally Required**:<br>- **Required** for `booking_type=1`.<br>- **Forbidden** otherwise. |
| `prior_notice_duration_max` | Integer | **Conditionally Forbidden** | Maximum number of minutes before travel to make the booking request.<br><br>**Conditionally Forbidden**:<br>- **Forbidden** for `booking_type=0` and `booking_type=2`.<br>- Optional for `booking_type=1`.|
| `prior_notice_last_day` | Integer | **Conditionally Required** | Last day before travel to make the booking request. <br><br>Example: “Ride must be booked 1 day in advance before 5PM” will be encoded as `prior_notice_last_day=1`.<br><br>**Conditionally Required**:<br>- **Required** for `booking_type=2`.<br>- **Forbidden** otherwise. |
| `prior_notice_last_time` | Time | **Conditionally Required** | Last time on the last day before travel to make the booking request.<br><br>Example: “Ride must be booked 1 day in advance before 5PM” will be encoded as `prior_notice_last_time=17:00:00`.<br><br>**Conditionally Required**:<br>- **Required** if `prior_notice_last_day` is defined.<br>- **Forbidden** otherwise. |
| `prior_notice_start_day` | Integer | **Conditionally Forbidden** | Earliest day before travel to make the booking request.<br><br>Example: “Ride can be booked at the earliest one week in advance at midnight” will be encoded as `prior_notice_start_day=7`.<br><br>**Conditionally Forbidden**:<br>- **Forbidden** for `booking_type=0`.<br> - **Forbidden** for `booking_type=1` if `prior_notice_duration_max` is defined.<br> - Optional otherwise. |
| `prior_notice_start_time` | Time | **Conditionally Required** | Earliest time on the earliest day before travel to make the booking request.<br><br>Example: “Ride can be booked at the earliest one week in advance at midnight” will be encoded as `prior_notice_start_time=00:00:00`.<br><br>**Conditionally Required**:<br>- **Required** if `prior_notice_start_day` is defined.<br>- **Forbidden** otherwise. |
| `prior_notice_service_id` | ID referencing `calendar.service_id` | **Conditionally Forbidden** | Indicates the service days on which `prior_notice_last_day` or `prior_notice_start_day` are counted. <br><br>Example: If empty, `prior_notice_start_day=2` will be two calendar days in advance. If defined as a `service_id` containing only business days (weekdays without holidays), `prior_notice_start_day=2` will be two business days in advance.<br><br>**Conditionally Forbidden**:<br> - Optional if `booking_type=2`. <br> - **Forbidden** otherwise. |
| `message` | Text | Optional | Message to riders utilizing service at a `stop_time` when booking on-demand pickup and drop off. Meant to provide minimal information to be transmitted within a user interface about the action a rider must take in order to utilize the service. |
| `pickup_message` | Text | Optional | Functions in the same way as `message` but used when riders have on-demand pickup only. |
| `drop_off_message` | Text | Optional | Functions in the same way as `message` but used when riders have on-demand drop off only. |
| `phone_number` | Phone number | Optional | Phone number to call to make the booking request. |
| `info_url` | URL | Optional | URL providing information about the booking rule. |
| `booking_url` | URL | Optional | URL to an online interface or app where the booking request can be made. |
