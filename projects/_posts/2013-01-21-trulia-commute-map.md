---
layout: page
title: Trulia Commute Map
thumbnail: /images/trulia-new-york-commute-small.png
thumbnail_description: Trulia New York commute map
description: An interactive visualisation of commute times to help home buyers decide where they want to live.
---


The Trulia Commute Map is an interactive map on [Trulia Local](http://www.trulia.com/local) that helps home-buyers to find areas to live which are nearby to their workplace.

![Commute times for New York City](/images/trulia-new-york-commute.png "Commute times for New York City")

The screenshot above is a commute map of New York City, and can be viewed online [here](http://www.trulia.com/local#commute/new-york-ny). It visualises approximate times where someone can drive to starting from Lower Manhattan in under 60 minutes. The origin marker can be dragged around the map, and the time slider can be changed to view areas within a time range.

I worked alongside [Sha Hwang](http://postarchitectural.com) and [Talin Salway](https://twitter.com/YenTheFirst) on this project. I worked on the API and the server configuration, while Sha worked on the front-end visualisation and Talin focussed on the datasets.

### Under the hood
The map uses [OpenStreetMap](http://osm.org) and [GTFS](https://developers.google.com/transit/gtfs/reference) data to calculate the driving and public transport commute times respectively. We used [PostgreSQL](http://postgresql.org) for the database with [PostGIS](http://postgis.refractions.org). [pgRouting](http://pgrouting.net) was used to calculate the commute times as a shortest-path tree from an origin. The JSON API was written in Python, using [GeoDjango](http://geodjango.org) and [TileStache](http://tilestache.org) libraries.

The browser loaded the commute data in 256 x 256 pixel data tiles in JSON. The request is made up of the commute type ("driving" or "transit"), the origin latitude/longitude, and [TMS style](http://en.wikipedia.org/wiki/Tile_Map_Service) x/y/z coodinates for the tile area.

An example request: [http://tiles.trulia.com/commute/time_map/data/driving/40.7143528/-74.0059731/12/1205/1538.json](http://tiles.trulia.com/commute/time_map/data/driving/40.7143528/-74.0059731/12/1205/1538.json) is for a tile area within New York City, which returns a response in the following format:

    {
        costs: [
            [173, 0, 0.999],  // [intersection x, intersection y, (cost * 60) minutes]
            ...
            [180, 84, 0.662],
            ....
            [154, 265, 0.468]
        ],
        pixel_radius: 8
    }

Each array in the `costs` array represents an intersection and it's commute time from the origin. The first two parameters are integers that represent the intersection's (x/y) pixel location relative to the top left of the tile, and the last parameter is a decimal that represents the commute time as a fraction of 60 minutes. The `pixel_radius` value tells the browser how large to render each intersection's circle so the map looks the same independent of the zoom level.

After the user sets the marker on the map, the browser loads the commute data tiles for the marker's origin. On the server side, a shortest-path tree of driving times is calculated using pgRouting's [driving_distance](http://pgrouting.org/docs/1.x/dd.html) function. A query is passed to `driving_distance` to get the edges (road line segments), either from OpenStreetMap data (driving), or GTFS data (public transport), and each edges' cost (fraction of an hour). `driving_distance` will then return a set of node latitude/longitude points and costs for points within a 1.0 degree radius of the origin point. Each tile request for the origin will contain part of this result set for the tile's bounding box, where latitude/longitude points are projected to pixel coordinates using TileStache).

After the browser receives a tile response, it will draw each point from the JSON data set on a canvas element (if the point's time is within the slider's time range). The colour of each intersection ranges from green to red (through the HSL colour model), where light green = 0 minutes, and red = 60 minutes. The colours are grouped in 5 minute increments. 

Here is an image of the commute times from Lower Manhattan.

![Raw commute times for New York](/images/driving-time-new-york.png "Raw Commute times for New York")

### Challenges
After importing all datasets for the US, we found that pgRouting's `driving_distance` function had slow performance. This was mostly due to the large size of the OSM/GTFS datasets for all of the US. Since any given query was only analysing edges within a 1.0 degree radius, we partitioned the tables into areas of 1.0 degree blocks, where each table overlapped it's adjacent tables by 0.5 degrees. This allowed us to query a table that only contained the edges we needed, with a much smaller dataset.

HTML canvas performed slowly when rendering a lot of circles. This was more apparent when the time slider was shifted because a lot of circles would have to be rendered and removed on the fly. The bottleneck was in the `context.arc` function, which was being invoked for every point we needed to draw. We decided to invoke `context.arc` once, and reuse the resulting rendered circle for each intersection. This made the rendering a lot faster, which was important due to the map's interactive functionality.

In an earlier prototype, we used greyscale images to represent the costs for each pixel, where each pixel was coloured by the node of the nearest neighbour. We found this technique to be particularly slow, mainly due to the fact that we were processing 65536 (256 x 256) pixels for each tile. After using the JSON tile approach with only the points that we needed, the performance improved significantly.
