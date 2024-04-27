# Query Open Street Map

Overpass Turbo is a website where we can query OpenStreetMap (OSM).

To display all areas tagged as `beach`, `coastline`, and `sand`:

```
[out:json][timeout:25];
(
  area["natural"="beach"]({{bbox}});
  area["natural"="coastline"]({{bbox}});
  area["natural"="sand"]({{bbox}});
);
out geom;
```

link: https://overpass-turbo.eu/s/1KIB
