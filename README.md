This directory contains the GTFS-flex specification and documentation.

### About GTFS-flex

GTFS-flex is a proposed/prototype extension to the [General Transit Feed Specification](https://github.com/google/transit/tree/master/gtfs). GTFS-flex would add the capability to model various demand-responsive transportation (DRT) services to GTFS, which currently only models fixed-route public transportation. GTFS-flex is in a prototyping phase in which its capabilities to model transportation services are being tested and adapted. There is no software that currently consumes GTFS-flex data, though projects have been proposed.

### Change Process

Discussion and change proposals are strongly encouraged. Rapid iteration and prototyping at this stage is necessary to develop GTFS-flex. We have adopted the following change proposal process:

1. Discussion and needs documentation occur in GitHub issue threads. Refer to and describe real DRT service features in order to develop and support needs for GTFS-flex modeling capabilities.
2. Propose changes to the spec that refer to needs identified in GitHub issues spec.
3. Create pull request to vote on changes to the prototype spec. The pull requester becomes advocate for the proposal.
4. Over the course of 7 days, stakeholders vote on the pull requests. Constructive feedback or alternative suggestions are required with downvote.
5. Pull requests with at least one +1 and no -1 are merged.

The above process is intended to begin tentative governance practices, and prevent spec bloat. Given that the GTFS-flex is in early prototype phase, speculative and experimental features are welcome.

### Quick links
* [GTFS-flex Google Group](https://groups.google.com/forum/#!forum/gtfs-flexible-wg)
* [Original Google Doc proposal](https://docs.google.com/document/d/1UTcpMJlANSoJ1ZEk5IrQh_plza1ZnvgwraMEBI_o2mw/edit?usp=sharing)
* [GTFS-flex draft spec in this repo](spec/reference.md)

### Background & Purpose

We’d like to better support modeling of flexible public transportation services in GTFS. For the purposes of this discussion, “flexible” services will be defined with regard to the following characteristics, as defined in [TCRP Program Report #140 - A Guide for Planning and Operating Flexible Public Transportation Services](http://www.trb.org/Main/Blurbs/163788.aspx):

* **Route Deviation:** vehicles operating on a regular schedule along a well-defined path, with or without marked bus stops, that deviate to serve demand-responsive requests within a zone around the path. The width or extent of the zone may be precisely established or flexible.
* **Point Deviation:** vehicles serving demand-responsive requests within a zone and also serving a limited number of stops within the zone without any regular path between the stops.
* **Demand-Responsive Connector:** vehicles operating in demand-responsive mode within a zone, with one or more scheduled transfer points that connect with a fixed-route network. A high percentage of ridership consists of trips to or from the transfer points.
* **Request Stops:** vehicles operating in conventional fixed-route, fixed-schedule mode and also serving a limited number of undefined stops along the route in response to passenger requests.
* **Flexible-Route Segments:** vehicles operating in conventional fixed-route, fixed-schedule mode, but switching to demand-responsive operation for a limited portion of the route.
* **Zone Route:** vehicles operating in demand-responsive mode along a corridor with established departure and arrival times at one or more end points in the zone.

The TCRP report catalogs a number of transit agencies across the US, evaluating how each agency uses flexible public transportation services in their own system. What’s clear from this analysis is that these agencies use a variety of techniques, picking and choosing from various strategies and flavors of flexible service when implementing their systems. Supporting all such services in GTFS will be tricky, but hopefully not impossible.