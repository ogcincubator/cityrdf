# the structure
```turtle
con:usage  rdf:type      owl:ObjectProperty;
        rdfs:label       "usage"@en;
        rdfs:range       con:DoorUsageValue;
        skos:definition  "Specifies the actual uses of the Door."@en .
```
will be splitted into one defined in the particular package
```turtle
core:usage  rdfs:range       con:DoorUsageValue;
        skos:definition  "Specifies the actual uses of the Door."@en .
```
and the second defined once in core package

```turtle
core:usage  rdf:type      owl:ObjectProperty;
        rdfs:label       "usage"@en;
```

for ns in citygml_ns, 
    for item in 
{address,
usage,
function,
value,
class}

