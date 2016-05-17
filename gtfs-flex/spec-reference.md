## Introduction

Discussion: GTFS Flexible Transit Working Group (Google Groups)

We’d like to better support modeling of flexible public transportation services in GTFS.  For the purposes of this discussion, “flexible” services will be defined with regard to the following characteristics, as defined in TCRP Program Report #140 - A Guide for Planning and Operating Flexible Public Transportation Services:

* Route Deviation: vehicles operating on a regular schedule along a well-defined path, with or without marked bus stops, that deviate to serve demand-responsive requests within a zone around the path. The width or extent of the zone may be precisely established or flexible.
Point Deviation: vehicles serving demand-responsive requests within a zone and also serving a limited number of stops within the zone without any regular path between the stops.
* Demand-Responsive Connector: vehicles operating in demand-responsive mode within a zone, with one or more scheduled transfer points that connect with a fixed-route network. A high percentage of ridership consists of trips to or from the transfer points.
Request Stops: vehicles operating in conventional fixed-route, fixed-schedule mode and also serving a limited number of undefined stops along the route in response to passenger requests.
* Flexible-Route Segments: vehicles operating in conventional fixed-route, fixed-schedule mode, but switching to demand-responsive operation for a limited portion of the route.
* Zone Route: vehicles operating in demand-responsive mode along a corridor with established departure and arrival times at one or more end points in the zone.

The TCRP report catalogs a number of transit agencies across the US, evaluating how each agency uses flexible public transportation services in their own system.  What’s clear from this analysis is that these agencies use a variety of techniques, picking and choosing from various strategies and flavors of flexible service when implementing their systems.  Supporting all such services in GTFS will be tricky, but hopefully not impossible.

## Sections

* [Defining Areas]
* [Route Deviation]
* [Request Stops]
* [Examples]