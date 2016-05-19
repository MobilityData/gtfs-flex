##Flexible Public Transportation Services in GTFS

###Table of Contents

* [Introduction](#introduction)
* [Defining Areas](#defining-areas)
* [Route Deviation](#route-deviation)
* [Request Stops](#request-stops)
* [Examples](#examples)

###Introduction

Discussion: GTFS Flexible Transit Working Group (Google Groups)

We’d like to better support modeling of flexible public transportation services in GTFS. For the purposes of this discussion, “flexible” services will be defined with regard to the following characteristics, as defined in [TCRP Program Report #140 - A Guide for Planning and Operating Flexible Public Transportation Services](http://www.trb.org/Main/Blurbs/163788.aspx):

* Route Deviation: vehicles operating on a regular schedule along a well-defined path, with or without marked bus stops, that deviate to serve demand-responsive requests within a zone around the path. The width or extent of the zone may be precisely established or flexible.
* Point Deviation: vehicles serving demand-responsive requests within a zone and also serving a limited number of stops within the zone without any regular path between the stops.
* Demand-Responsive Connector: vehicles operating in demand-responsive mode within a zone, with one or more scheduled transfer points that connect with a fixed-route network. A high percentage of ridership consists of trips to or from the transfer points.
* Request Stops: vehicles operating in conventional fixed-route, fixed-schedule mode and also serving a limited number of undefined stops along the route in response to passenger requests.
* Flexible-Route Segments: vehicles operating in conventional fixed-route, fixed-schedule mode, but switching to demand-responsive operation for a limited portion of the route.
* Zone Route: vehicles operating in demand-responsive mode along a corridor with established departure and arrival times at one or more end points in the zone.

The TCRP report catalogs a number of transit agencies across the US, evaluating how each agency uses flexible public transportation services in their own system. What’s clear from this analysis is that these agencies use a variety of techniques, picking and choosing from various strategies and flavors of flexible service when implementing their systems. Supporting all such services in GTFS will be tricky, but hopefully not impossible.

###Defining Areas

A key concept in flexible services is the idea of a service area or zone. These zones are used define the region where demand-response operation is in effect. We propose a model for defining these regions in GTFS. Because “zone” also tends to get conflated with discussions of fare systems, we propose calling general polygons “areas”. As such, we propose introducing a new file, **areas.txt**, with the following fields:

| Field Name | Required?  | Details |
|------------|------------|---------|
| id         | Required   | The **id** field contains an ID that uniquely identifies an area. |
| lat        | Required   | The **lat** field specifies the latitude of a single point in the area’s polygon. The field value must be a valid WGS84 latitude. |
| lon        | Required   | The **lon** field specifies the longitude of a single point in the area’s polygon. The field value must be a valid WGS84 longitude. |
| sequence   | Required   | The **sequence** field associates the latitude and longitude of a single point with its sequence order in the area’s polygon. The value of **sequence** must be a non-negative integers and must increase sequentially between each point in the polygon. |

Basically, **areas.txt** provides a mechanism for defining polygon regions, identified by id. The coordinates for areas must be specified in counterclockwise order. Areas follow the "right-hand rule," which states that if you place the fingers of your right hand in the direction in which the coordinates are specified, your thumb points in the general direction of the geometric normal for the polygon.

###Route Deviation

As discussed, flexible routes can use service areas and zones in a variety of ways. Some routes provide demand-response services with a single fixed zone, while others offer regions of flexible services along an otherwise fixed route, while yet others use combinations of the two.

To support this flexibility in GTFS, we propose augmenting the functionality of **stop_times.txt** by allowing a provider to associate service areas with particular stop-time entry.

Specifically, we propose the following additions to **stop_times.txt**:

| Field Name | Required?  | Details |
|------------|------------|---------|
| start\_service\_area\_id | Optional | The **start\_service\_area\_id** specifies the id of a service area defined in the areas.txt file. When specified for a stop-time entry, it indicates the beginning of service in the specified service area and that the transit vehicle may potentially pick-up or drop-off passenger anywhere in the specified service area, as controlled by the **pickup\_type** and **drop\_off\_type** fields. Must be followed by a corresponding stop-time entry in the trip with a matching **end\_service\_area\_id** value. | 
| end\_service\_area\_id | Optional | The **end\_service\_area\_id** specifies the id of a service area defined in the **areas.txt** file. When specified for a stop-time entry, it indicates the end of service in the specified service area. Must be preceded by a corresponding stop-time entry in the trip with a matching **start\_service\_area\_id** value. The **pickup\_type** and **drop\_off\_type** fields will be ignored for this stop-time entry. |
| start\_service\_area\_radius | Optional | The **start\_service\_area\_radius** field provides a convenient way to construct a service area without the need to define an area in **areas.txt**.  When specified, it indicates a distance in meters from the main path of travel for the trip where service is offered. The trip must specify a **shape\_id** if **start\_service\_area\_radius** is specified. The **start\_service\_area\_radius** field has the same semantics as **start\_service\_area\_id** in terms of pickup/dropoff semantics. |
| end\_service\_area\_radius | Optional | The **end\_service\_area\_radius** specifies the end of an area started by a stop-time entry with a **start\_service\_area\_radius** field. Must be preceded by a corresponding stop-time entry in the trip with the same **start\_service\_area\_radius** value. The **pickup\_type** and **drop\_off\_type** fields will be ignored for this stop-time entry. |

Regarding **stop\_id**, two possibilities:

* We allow **stop\_id** to be empty when **start\_service\_area\_id** has been specified. This breaks the idea that **stop\_id** is always required, but is conceptually cleaner.
* We still require **stop\_id** to be specified. It should specify a stop at the center of the service area and might be used to provide additional data for the service area (name? url? fare zone?).

An alternative to the above options is to add a **stop\_times\_areas.txt** file which contains the records with **start\_service\_area\_id** and **end\_service\_area\_id**. The schedule for a **trip\_id** is expressed through the combination of these two files — **stop\_sequence** numbers count up through both files. For an example, see the gtfs-flexible sandbox example 5.A or “Green Mountain”.

Regarding **arrival\_time** and **departure\_time**, these fields can be used to specify rough timing information for a route deviation or flexible route segment relative to other parts of the route, or can be left blank if it’s not the first or last stop-time in the trip. If a time is specified, it indicates when service begins in the area.

###Request Stops

Many transit systems allow riders to board or alight from a transit vehicle at any location along a route. Alternatively known as “flag stop”, riders can flag down a vehicle at locations other than fixed stops.

At the GTFS For the Rest of Us Workshop in Washington, DC (Nov 2013), we came up with a proposal for specifying request stops called “continuous stops”. In this proposal, a route can be annotated as supported continuous stops, such that a rider can board or alight from a transit vehicle at any point along the route, as determined by the route’s shape. See the proposal doc for more details.

One scenario that we did not tackle during the workshop was support for “request stops” for sub-sections of a route.  We’d now like to propose such a mechanism with the following new fields in **stop\_times.txt**:

| Field Name | Required?  | Details |
|------------|------------|---------|
| continuous\_stops | Optional | The **continuous\_stops** field can be used to indicate a section of a trip where it is possible to board or alight from the transit vehicle at any point along the vehicle’s path of travel. |

The **continuous\_stops** field can have the following non-negative integer values:

* 0 or blank - Normal stop behaviour along route (default).
* 1 - Continuous stopping behaviour from this stop-time to the next stop-time in the trip’s sequence.

If specified as 1, a valid shape must be defined for the trip, in order to indicate the complete path of travel.

###Examples

#####Single Zone, No Defined Stops

Let’s consider an agency operating a route with a single demand-response zone.  Riders are required to call the agency in advance to coordinate pickup and dropoff.

In this case, the agency defines a service area in areas.txt with id AreaX.  The agency also defines a trip in **trips.txt** with id *TripX*.  We also add two entries to **stop\_times.txt**:

| --- | --- | --- |
| trip\_id | TripX | TripX |
| stop\_id | StopX | StopX |
| stop\_sequence | 0 | 1 |
| arrival\_time | 09:00:00 | 17:00:00 |
| departure\_time | 09:00:00 | 17:00:00 |
| start\_service\_area\_id | AreaX |  |
| end\_service\_area\_id |  | AreaX |
| pickup\_type | 2 |  |
| drop\_off\_type | 2 |  |

These two stop-time entries start service in *AreaX* at 9 am and stop service in the area at 5 pm.  Riders are required to call the agency in advance to coordinate pickup and dropoff.

#####Single Zone, Defined Endpoints

Let’s now consider an agency operating a route with a single demand-response that has a regular start and end point.  In this case, *StopX* and *StopZ* are the start and end stops, and *AreaX* is the service area served in-between.

| --- | --- | --- | --- | --- |
| trip\_id | TripX | TripX | TripX | TripX |
| stop\_id | StopX | StopY | StopY | StopZ |
| stop\_sequence | 0 | 1 | 2 | 3 |
| arrival\_time | 09:00:00 |  |  | 10:00:00 |
| departure\_time | 09:00:00 |  |  | 10:00:00 |
| start\_service\_area\_id |  | AreaX |  |  |
| end\_service\_area\_id |  |  | AreaX |  |
| pickup\_type |  | 2 |  |  |
| drop\_off\_type |  | 2 |  |  |
 
#####Multiple Zones and Intermediate Stops
 
In this case, the agency builds on the previous example, but now defines a mix of normal stop-time and service-area stop-time entries in **stop\_times.txt**:

| --- | --- | --- | --- | --- | --- | --- | --- |
| trip\_id | TripX | TripX | TripX | TripX | TripX | TripX | TripX |
| stop\_id | StopA | StopB | StopB | StopC | StopD | StopD | StopE |
| stop\_sequence | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| arrival\_time | 09:00:00 |  |  | 09:30:00 |  |  | 10:00:00 |
| departure\_time | 09:00:00 |  |  | 09:30:00 |  |  | 10:00:00 |
| start\_service\_area\_id |  | AreaX |  |  |
| end\_service\_area\_id |  |  | AreaX |  |  | AreaY |  |
| pickup\_type |  | 3 |  |  | 3 |  |  |
| drop\_off\_type |  | 3 |  |  | 3 |  |  |

In this example, the transit vehicle starts at fixed-stop *StopA*, travels through service area *AreaX*, stops at fixed-stop *StopC*, travels through service area *AreaY*, and finishes service at fixed-stop *StopE*.

#####Intermediate Stops Within Zones

In this case, the agency has a fixed start and end stop, a flexible zone between those stops, and fixed stops within the flexible zone.

| --- | --- | --- | --- | --- | --- |
| trip\_id | TripX | TripX | TripX | TripX | TripX |
| stop\_id | StopA | StopB | StopC | StopB | StopD |
| stop\_sequence | 0 | 1 | 2 | 3 | 4 |
| arrival\_time | 09:00:00 |  | 09:30:00 |  | 10:00:00 |
| departure\_time | 09:00:00 |  | 09:30:00 |  | 10:00:00 |
| start\_service\_area\_id |  | AreaX |  |  |  |
| end\_service\_area\_id |  |  |  | AreaX |  |
| pickup\_type |  | 3 |  |  |  |
| drop\_off\_type |  | 3 |  |  |  |

#####Radius-Based Zones

Some agencies have a simple methodology for defining demand-response service areas. The zone is typically defined as a particular distance from the base route (eg. ½ mile). For agencies that don’t want to maintain the actual geometry for that corridor, the **start\_service\_area\_radius** and **end\_service\_area\_radius** fields offer a simple way to define a service area.

In the following example, an agency defines a ½ mile (~800 meter) flexible service area around their base route. For this technique to work, the trip with id *TripX* must also specify a **shape\_id** value indicating the path of travel.
 
| --- | --- | --- | --- | --- |
| trip\_id | TripX | TripX | TripX | TripX |
| stop\_id | StopX | StopY | StopY | StopZ |
| stop\_sequence | 0 | 1 | 2 | 3 |
| arrival\_time | 09:00:00 |  |  | 10:00:00 |
| departure\_time | 09:00:00 |  |  | 10:00:00 |
| start\_service\_area\_radius |  | 800 |  |  |
| end\_service\_area\_radius |  |  | 800 |  |
| pickup\_type |  | 2 |  |  |
| drop\_off\_type |  | 2 |  |  |

#####Advanced

The Cape Cod RTA operates a [flex route](http://www.capecodrta.org/flex-route.htm#map) between Harwich and Provincetown that combines a number of aspects of flexible service: route deviation, request stops, flexible-route segments, etc.

We can combine the new fields we have defined, including **start\_service\_area\_radius**, **end\_service\_area\_radius**, and **continuous\_stops**, to concisely represent this route and all its flexible characteristics.

