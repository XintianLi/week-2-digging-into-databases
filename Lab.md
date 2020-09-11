# Lab - Week 2 - More SQL and PostGIS Basics

## Goal of Lab

* Use PostGIS functions to explore properties of geometries in dataset
* Combine PostGIS with other PostgreSQL `WHERE` conditions, `GROUP BY`s, `ORDER BY`s, etc.
* Deal with the quirks of messy data
* Get familiar with some OpenStreetMap data

## Dataset

**Dataset:** OpenStreetMap Buildings for University City Neighborhood
* [Preview data](https://github.com/MUSA-509/week-2-digging-into-databases/blob/master/data/university_city_osm_buildings.geojson)
* Download link (copy and paste this into the Carto upload tool): `https://raw.githubusercontent.com/MUSA-509/week-2-digging-into-databases/master/data/university_city_osm_buildings.geojson`

I fetched this data from GeoFabrik's wonderful OSM download service. I downloaded the [Pennsylvania](https://download.geofabrik.de/north-america/us/pennsylvania.html) file and filtered to only included building footprints that land in University City neighborhood in Philadelphia.

About the dataset, notice the schema:

- `cartodb_id` (number, actually an integer): Carto adds this as a utility for optimizing performance of this table. It wasn't in the original dataset.
- `the_geom` (geometry): Carto uses `the_geom` for the geometry column. In the GeoJSON it was `geometry` as per the [GeoJSON specification](https://geojson.org/).
- `osm_id` (string): A unique identifier for this geometry in the OSM database
- `code` (number): OSM code for type building. This are all the same in this dataset.
- `fclass` (string): Description of OSM feature. All are `building` here.
- `name` (string): Name of building. Note: many are not named
- `building_type` (string): Type of building (if assigned). Notice that many are null valued.

## Recommended Workflow

1. Upload data to your Carto account
2. Open dataset in map or data view and toggle on the SQL panel
3. Clone this repository or download the `Lab.md` Markdown file
4. Edit the file in your favorite text editor. Here are two recommendations:
  * [Atom](https://atom.io/) -- my favorite
  * [VS Code](https://code.visualstudio.com/)

## Find Average Area (in square meters) of all buildings

Use the PostGIS function [`ST_Area`](https://postgis.net/docs/ST_Area.html). Since our geometries are in WGS84, we need to use `geography` type casting (`the_geom::geography`).

```SQL
SELECT avg(ST_Area(the_geom::geography))
FROM university_city_osm_buildings

1730.416431997879
```

## Find Average Area (in square meters) of buildings by type

Write a query to give:

* type of building
* average area of building by type
* count of buildings of this type

```SQL
SELECT building_type, avg(ST_Area(the_geom::geography)), count(*) as cnt
FROM university_city_osm_buildings
GROUP BY building_type
```

## Which university building is largest? Smallest?

Which building is largest?

```SQL
-- Enter your SQL query here
```

Which building is smallest?

```SQL
-- Enter your SQL query here
```

## What is the area (in square meters) of Meyerson Hall?

```SQL
-- Enter your SQL query here
```

## Find the 10 closest buildings to Meyerson Hall

Using the [`ST_Distance`](https://postgis.net/docs/ST_Distance.html) function, find the 10 closest buildings to Meyerson Hall.

Hint: We can use `ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)` for the geometry to represent Meyerson Hall.

```SQL
-- Enter your SQL query here
```

## Find the 10 closest buildings to Meyerson Hall using the distance operator

PostGIS has a [distance operator (`<->`)](https://postgis.net/docs/geometry_distance_knn.html) that allows one to do distance ordering in an `ORDER BY`.

```SQL
-- Enter your query here
```

## Find the 10 closest buildings to Meyerson Hall excluding Meyerson Hall

Use the [`ST_Intersects`](https://postgis.net/docs/ST_Intersects.html) function to exclude Meyerson Hall. We know that Meyerson Hall is at the point `ST_SetSRID(ST_MakePoint(-75.19265679, 39.9522405), 4326)`. `ST_Intersects` returns true/false/null. To only included geometries that do not intersect, precede `ST_Intersects` with `NOT`.

```SQL
-- Enter your query here
```

## Compute and Map the Convex Hull of all of the buildings

Use the [`ST_ConvexHull`](https://postgis.net/docs/ST_ConvexHull.html) function to find the minimum convex shape that includes all of the buildings in University City. See step 5 in the [Mapping Follow Along](https://github.com/MUSA-509/week-2-digging-into-databases#mapping-follow-along) from this week's lecture.

```SQL
-- Enter your query here
```

## Finding Largest / Smallest buildings by class

This query gives us the largest building (by building footprint area) for each building class:

```SQL
SELECT
  ST_Area(b.the_geom::geography) as area,
  b.name,
  b.building_type
FROM andyepenn.university_city_osm_buildings as b
JOIN (
  SELECT max(ST_Area(the_geom::geography)) as maxarea, building_type
  FROM andyepenn.university_city_osm_buildings
  GROUP BY building_type
  ) as m
ON m.building_type = b.building_type AND ST_Area(b.the_geom::geography) = m.maxarea
```

Modify this query to give the **smallest** building for each class instead:

```SQL
-- Enter your query here
```
