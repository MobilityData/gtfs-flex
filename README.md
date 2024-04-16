Update: 
- April 16, 2024. **[GTFS-Flex has been merged to GTFS](https://github.com/google/transit/pull/433). This repo is no longer up-to-date and will deprecated. Consult the [google/transit](https://github.com/google/transit) repo for the up-to-date information**
- February 14, 2024. The most recent GTFS-Flex proposal can be found on [google/transit](https://github.com/google/transit/pull/433). **If the proposal passes, this repo will be out-of-date and removed.** Please consult [this proposal](https://github.com/google/transit/pull/433) for more details. 
- November 25, 2020. The spec in this repository is now the most up-to-date version of GTFS-Flex. The code in this repo contains the official proposal for GTFS-Flex v2, a GTFS extension that covers all demand-responsive services for the purposes of discovery in trip planning. All features of this specification proposal are currently produced by [Trillium](https://trilliumtransit.com/) and consumed by [OpenTripPlanner version 2](https://www.opentripplanner.org/). DemandTrans and IBI group expect to produce data in this spec by early 2021.

"Version 1" of the GTFS-Flex specification is utilized by OTP 1.4. You can review the Version 1 specification by reviewing versions of this repository from before October 2020.

### About GTFS-Flex

GTFS-Flex is a proposed extension to the [General Transit Feed Specification](http://gtfs.org/). GTFS-Flex adds the capability to model various demand-responsive transportation (DRT) services to GTFS, which currently only models fixed-route public transportation. GTFS-flex is now produced for over 100 transit services, and provides flexible transit trip plans through [OpenTripPlanner](https://www.opentripplanner.org/).

[See the GTFS-Flex proposal here.](spec/reference.md)

### Spec extension schematic diagram

The below shows updated and added files in GTFS-Flex.

![Diagram of added files in GTFS-Flex](spec/Flex%20Schema%20Diagram.png)

### Example Flex v2 Feeds
[On-demand service](spec/FlexExample--on-demand-service.zip)<br>[Same-day service](spec/FlexExample--same-day-service.zip)<br>[Various](spec/FlexExample--various.zip)
