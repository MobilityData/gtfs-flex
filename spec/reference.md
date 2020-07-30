## Flexible Public Transportation Services in GTFS

### Table of Contents

* [Point Deviation](#point-deviation)
* [Defining Areas](#defining-areas)
* [Route Deviation](#route-deviation)
* [Request Stops](#request-stops)
* [Defining Service Parameters](#defining-service-parameters)
* [Examples](#examples)

### Point Deviation

The existing GTFS assumes that stops are served in an order, defined by stop\_sequence. In order to describe a point-deviation service, specification needs the capability to override this assumption.

Proposed: A new field in **stop_times.txt**, **unordered**.

| Field Name | Required?  | Details |
|------------|------------|---------|
| unordered  | Optional   | Consecutive values of 1 indicate a block of stops (deviation points) are not served in a predetermined order. This block is interrupted by a 0 value. Empty or other values are presumed to be 0. |

Note that if a **shape_id** is specified for a trip that includes stops where **unordered = 1**, then **shape\_dist\_traveled** must be specified in **stop_times.txt** and **shapes.txt**. The correspondence of **shape\_dist\_traveled** values must make it clear that there is no alignment specified for the block of deviation points.

### Defining Areas

A key concept in flexible services is the idea of a service area or zone. These zones are used define the region where demand-response operation is in effect. We propose a model for defining these regions in GTFS. Because “zone” also tends to get conflated with discussions of fare systems, we propose calling general polygons “areas”. As such, we propose introducing a new file, **areas.txt**, with the following fields:

| Field Name | Required?  | Details |
|------------|------------|---------|
| area_id    | Required   | The **area_id** field contains an ID that uniquely identifies an area. |
| wkt        | Required   | The **wkt** field specifies a polygon, multipolygon, or other area in the well-known text (WKT) format. |

Basically, **areas.txt** provides a mechanism for defining polygon regions, identified by id. The coordinates for areas must be specified in counterclockwise order. Areas follow the "right-hand rule," which states that if you place the fingers of your right hand in the direction in which the coordinates are specified, your thumb points in the general direction of the geometric normal for the polygon.

### Route Deviation

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

Regarding **arrival\_time** and **departure\_time**, these fields can be used to specify rough timing information for a route deviation or flexible route segment relative to other parts of the route, or can be left blank if it’s not the first or last stop-time in the trip. If a time is specified, it indicates when service begins in the area.

### Request Stops

Many transit systems allow riders to board or alight from a transit vehicle at any location along a route. Alternatively known as “flag stop”, riders can flag down a vehicle at locations other than fixed stops.

At the GTFS For the Rest of Us Workshop in Washington, DC (Nov 2013), we came up with a proposal for specifying request stops called “continuous stops”. In this proposal, a route can be annotated as supported continuous stops, such that a rider can board or alight from a transit vehicle at any point along the route, as determined by the route’s shape. See the proposal doc for more details.

One scenario that we did not tackle during the workshop was support for “request stops” for sub-sections of a route.  We’d now like to propose such a mechanism with the following new fields in **stop\_times.txt**:

| Field Name | Required?  | Details |
|------------|------------|---------|
| continuous\_pickup | Optional | The **continuous\_pickup** field can be used to indicate a section of a trip where it is possible to board the transit vehicle at any point along the vehicle’s path of travel. |
| continuous\_drop\_off | Optional | The **continuous\_drop\_off** field can be used to indicate a section of a trip where it is possible to alight from the transit vehicle at any point along the vehicle’s path of travel. |

The **continuous\_pickup** and **continuous\_drop\_off** fields can have the following non-negative integer values:

* 0 - Continuous stopping behavior from this stop-time to the next stop-time in the trip’s sequence.
* 1 or blank - No continuous stopping behavior from this stop-time to the next stop-time in the trip’s sequence.
* 2 - Must phone agency to arrange continuous stopping behavior.
* 3 - Must coordinate with driver to arrange continuous stopping behavior.

If specified as 0, a valid shape must be defined for the trip, in order to indicate the complete path of travel.

### Defining Service Parameters

Demand-responsive transportation services have parameters for request requirements and expected or maximum travel times. Below are additions to the **trips.txt** file that define parameters for the demand-responsive service portions of that trip.

| Field Name | Required? | Details |
|-------------------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| drt\_max\_travel\_time | Optional | Defines maximum travel time for a demand-responsive passenger travel leg on the trip. This time as an arithmetic function in the form xt+c where t=[direct travel time], x and c are constants. For example, the formula 2.5t+5 indicates that the maximum travel time for the passenger's demand-responsive travel leg is 30 minutes if the direct travel time is 10 minutes. |
| drt\_avg\_travel\_time | Optional | Defines average or expected travel time demand-responsive passenger travel leg on the trip. Values or functions are expressed in the same way as drt\_max\_travel\_time. |
| drt\_advance\_book\_min | Optional | Minutes of advance time necessary before travel to make booking request. |
| drt\_pickup\_message | Optional | Message to be communicated to passengers if being picked up by a deviated service during this trip. The message is meant to provide the minimum information to be transmitted within a user interface. If extensive information is required, it should be provided through a phone number, url, or future communication with the end user. |
| drt\_drop\_off\_message | Optional | Message to be communicated to passengers if being dropped off by a deviated service during this trip. The message is meant to provide the minimum information to be transmitted within a user interface. If extensive information is required, it should be provided through a phone number, url, or future communication with the end user. |
| continuous\_pickup\_message | Optional | Message to be communicated to passengers if flagging a vehicle for pickup between designated stops during this trip. The message is meant to provide the minimum information to be transmitted within a user interface. If extensive information is required, it should be provided through a phone number, url, or future communication with the end user. |
| continuous\_drop\_off\_message | Optional | Message to be communicated to passengers if requesting a drop off between designated stops during this trip. The message is meant to provide the minimum information to be transmitted within a user interface. If extensive information is required, it should be provided through a phone number, url, or future communication with the end user. |

**Alternative consideration:** These service parameters could be included in **areas.txt** or **stop_times.txt** for more granular specificity.

### Eligibility

Many flexible services are available to the general public, but others require the customer to fall into a certain class of people (e.g. ages 65+, Medicaid recipient) and/or go through an application process. The variety of such systems is extensive, and will not be represented in the current specification pending further research and development. In order to provide a basic level of information about eligibility within flexible trip planners in the near term, the following field will be added to **routes.txt**. A 1 value for eligibility_restricted could be further explained in the drt\_pickup\_message or drt\_drop\_off\_message field, providing the customer with a short message indicating who the service is restricted to or contact information for details.

| Field Name | Required? | Details |
|-------------------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| eligibility\_restricted | Optional | 0 or blank indicates the service is open to the general public. 1 indicates that the service is restricted to a certain group of riders. 2 indicates that while fixed-route stops are open to the general public, deviations are restricted to a certain group of riders.|
| eligibility\_message | Optional | Message to be communicated to passengers. The message is meant to provide the minimum information to be transmitted within a user interface. If extensive information is required, it should be provided through a phone number, url, or future communication with the end user. |

**Alternative consideration:** These service parameters could be included in **areas.txt** or **stop_times.txt** for more granular specificity.

### Examples

##### Point Deviation, Scheduled Start and Endpoints

The service includes 4 untimed points (Stops B, C, D, and E), which are not served in a predefined order. Riders are required to call the agency in advance to coordinate pickup and dropoff at these stops. The vehicle is regularly scheduled to begin at StopA and return to StopA 30 minutes later.

|                 |          |       |       |       |       |       |
|-----------------|----------|-------|-------|-------|-------|-------|
| trip\_id        | TripX    | TripX | TripX | TripX | TripX | TripX |
| stop\_id        | StopA    | StopB | StopC | StopD | StopE | StopA |
| stop\_sequence  | 0        | 1     | 2     | 3     | 4     | 5     |
| unordered       | 0        | 1     | 1     | 1     | 1     | 0     |
| arrival\_time   | 09:00:00 |       |       |       |       | 9:30  |
| departure\_time | 09:00:00 |       |       |       |       | 9:30  |
| pickup\_type    |          | 2     | 2     | 2     | 2     |       |
| drop\_off\_type |          | 2     | 2     | 2     | 2     |       |

##### Single Zone, No Defined Stops

Let’s consider an agency operating a route with a single demand-response zone.  Riders are required to call the agency in advance to coordinate pickup and dropoff.

In this case, the agency defines a service area in areas.txt with area\_id AreaX.  The agency also defines a trip in **trips.txt** with area\_id *TripX*.  We also add two entries to **stop\_times.txt**:

|  |  |  |
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

##### Single Zone, Defined Endpoints

Let’s now consider an agency operating a route with a single demand-response that has a regular start and end point.  In this case, *StopX* and *StopZ* are the start and end stops, and *AreaX* is the service area served in-between.

|  |  |  |  |  |
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
 
##### Multiple Zones and Intermediate Stops
 
In this case, the agency builds on the previous example, but now defines a mix of normal stop-time and service-area stop-time entries in **stop\_times.txt**:

|  |  |  |  |  |  |  |  |
| --- | --- | --- | --- | --- | --- | --- | --- |
| trip\_id | TripX | TripX | TripX | TripX | TripX | TripX | TripX |
| stop\_id | StopA | StopB | StopB | StopC | StopD | StopD | StopE |
| stop\_sequence | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
| arrival\_time | 09:00:00 |  |  | 09:30:00 |  |  | 10:00:00 |
| departure\_time | 09:00:00 |  |  | 09:30:00 |  |  | 10:00:00 |
| start\_service\_area\_id |  | AreaX |  |  | AreaY | |
| end\_service\_area\_id |  |  | AreaX |  |  | AreaY |  |
| pickup\_type |  | 3 |  |  | 3 |  |  |
| drop\_off\_type |  | 3 |  |  | 3 |  |  |

In this example, the transit vehicle starts at fixed-stop *StopA*, travels through service area *AreaX*, stops at fixed-stop *StopC*, travels through service area *AreaY*, and finishes service at fixed-stop *StopE*.

##### Intermediate Stops Within Zones

In this case, the agency has a fixed start and end stop, a flexible zone between those stops, and fixed stops within the flexible zone.

|  |  |  |  |  |  |
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

##### Radius-Based Zones

Some agencies have a simple methodology for defining demand-response service areas. The zone is typically defined as a particular distance from the base route (eg. ½ mile). For agencies that don’t want to maintain the actual geometry for that corridor, the **start\_service\_area\_radius** and **end\_service\_area\_radius** fields offer a simple way to define a service area.

In the following example, an agency defines a ½ mile (~800 meter) flexible service area around their base route. For this technique to work, the trip with trip_id *TripX* must also specify a **shape\_id** value indicating the path of travel.

|  |  |  |  |  | 
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

##### Advanced

The Cape Cod RTA operates a [flex route](http://www.capecodrta.org/flex-route.htm#map) between Harwich and Provincetown that combines a number of aspects of flexible service: route deviation, request stops, flexible-route segments, etc.

We can combine the new fields we have defined, including **start\_service\_area\_radius**, **end\_service\_area\_radius**, **continuous\_pickup**, and **continuous\_drop\_off**, to concisely represent this route and all its flexible characteristics.

