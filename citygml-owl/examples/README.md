# Test files
lod3-sample*.files are taken from [Ingolstadt, Germany](https://github.com/savenow/lod3-road-space-models/blob/main/models/building/lod3/combined/citygml/lod3_building_models.gml)

CityGML 2.0 property `bldg:measuredHeight` is obsolete in 3.0 and replaced with `con:height`. 

In `lod3-sample-fail.*` used `bldg:measuredHeight` and prefixes of CityGML 2.0 - validator should return error

In `lod3-sample-ok.*` used `con:height` and prefixes of CityGML 3.0. There should not be errors of validation.

The conversion from GML to RDF/TTL is done with [xSPARQL script](https://github.com/Accord-Project/Tegel-scripts/blob/main/ifc/cityrdf_typedGenAttrs-withIDs-citygml3.xsparql), created in frame of [EU ACCORD project "Automated Compliance Checks for Construction, Renovation or Demolition Works", 2022-2025](https://accordproject.eu) (grant agreement 101056973).