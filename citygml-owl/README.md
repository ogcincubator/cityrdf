# CityGML-OWL derivation

This directory is used to define a CityGML OWL derived from the UML source.
The intention is to set this up as a proof of concept that can be migrated to the (or an) official CityGML specification repository.

We propose **CityRDF** as a name for the target family of ontologies. 

# Acknowledgements

CityRDF was conceived, developed and used first in frame of [EU *ACCORD* project "Automated Compliance Checks for Construction, Renovation or Demolition Works"](https://accordproject.eu/) (grant agreement 101056973, @nataschake, @ar-chad) and in frame of EU [*CHEKdbp* project "Change Toolkit for Digital Building Permits"](https://chekdbp.eu)  (grant agreement 101058559, @rob-metalinkage, @avillar).

## Background

A proposal to implement a CityGML OWL representation to support RDF encodings was presented to the CityGML Working Group in June 2023.

The [CHEK project](https://info.cype.com/en/research/chek-dbp/) has identified a need to reconcile related sources of information from different systems and designed an approach around "semantic uplift" to a minimal profile of each standard used to a common model that can support regulation evaluation. Thus this complex task does not need to be repeated for each version of each encoding approach - such as CityGML, CityJSON, BIM/IFC and various planning GIS layers.

Resources described in CityGML WG proposal (thanks to [Diego Vinasco-Alvarez](https://github.com/DiegoVinasco))

* CityGML 3.0 ShapeChange configs: https://github.com/VCityTeam/UD-Graph/blob/master/Transformations/ShapeChange/CityGML3.0_to_OWL_config.xml for CityOWL CWA and https://github.com/VCityTeam/UD-Reproducibility/blob/master/Computations/RDF/CityOWL/shapechange-configs/CityGML3.0_to_OWL_lite_config.xml for CityOWL OWA. 

* The most recent generated CityGML 3.0 OWL Ontologies: https://dataset-dl.liris.cnrs.fr/rdf-owl-urban-data-ontologies/Ontologies/CityGML/3.0/

* A Python script that fixes a few logical inconsistencies in the ShapeChange output: https://github.com/VCityTeam/UD-Graph/blob/master/Transformations/ShapeChange/ontologyPatcher.py

* A Repository serving as a source of evidence for generated OWL: https://github.com/VCityTeam/UD-Reproducibility/tree/master/Computations/RDF/CityOWL 

## Dependencies
1) ShapeChange (we tested the solution with ShapeChange 3.1.1 built from sources[https://github.com/ShapeChange/ShapeChange])
2) Python3

## The structure of the repository

```bash
CityGML-OWL
├── additional-triples   # additional files used in the workflow
├── CityRDF              # final result: relaxed domain and range restrictions with `schema:domain/rangeIncludes`
    └── codelists        # one of the possible transformation results for CityGML3.0 code lists
├── CityRDF-optimal      # final result: relaxed domain restriction but strict range as `owl:unionOf`
├── CityRDF-strict       # final result: strict domain and range restrictions with `owl:unionOf`
├── examples             # some CityGML files and their Turtle representations
├── scripts              # all necessary scripts used in the transformation
└── statistics           # results of qualifying SPARQL queries affecting the transformation actions
└── presentations        # presentation slides
```

# Current solution

We use [CityGML_3.0-workspaces-documents_shapechange-export.xml](https://github.com/VCityTeam/UD-Graph/blob/master/Transformations/test-data/UML/CityGML_3.0-workspaces-documents_shapechange-export.xml) as a source of CityGML3.0 model.

We added `<<Union>>` stereotype conversion (`rule-owl-cls-union`) and made use of `rule-owl-prop-globalScopeAttributes` to detect duplicate (see below) properties defined more than once per package and across packages.

The sequence of steps is as follows:
1) Using shell script `run.sh` perform ShapeChange transformation of CityGML 3.0 UML into initial set of onotologies.
2) Using shell script `patch-ontologies.sh` make the initial set of ontologies readable in Protege.
3) Using shell script `update-triples.sh` perform SPARQL based transformation of the ontologies.
4) (optional) Using shell scripts `refactor-optimal.sh` and `refactor-strict.sh` apply additional transformation of `schema:domainIncludes`/`schemaRangeIncludes` to wrap up multiple classes into `owl:unionOf` classes accesible with `rdfs:domain`/`rdfs:range` (see below) 

The whole workflow can be executed with `run-workflow.sh` script.

## Step 1

To run the ShapeChange conversion write in bash

```./run.sh```

The results will be in the folder `stage-1`.

## Step 2
To apply patches to make the results better viewable in Protege, see the explanation in [VCityTeam solution](https://github.com/VCityTeam/UD-Graph/tree/master/Transformations/ShapeChange) that is the solution to get the [initial version of CityGML3.0 in OWL](https://dataset-dl.liris.cnrs.fr/rdf-owl-urban-data-ontologies/Ontologies/CityGML/3.0/) and run the second script:

```./patch-ontologies.sh```

The results will be in the folder `stage-2`.

## Step 3

VCityTeam detected several obsolete/hanging definitions that were to be patched, and proposed `update-triples.sh` to patch them. 
The script did the following:

- merges in one file an ontology for the UML package with codelists' values defined in it.
- adds cityModelMember modifications
- adds GeoSPARQL and OWL-Time alignments
- removes outdated core triples and correspondent hanging restrictions
- patches RoomHeight.status range

We changed the script to reflect the logics behind the work with codelists: instead of merging two files - ontology file and codelists file (if codelists exist in a correspondent UML package), we put all files representing package codelists into a `/CityRDF/codelists` folder. This allowed us to avoid punning in OWL representation. 

We extended the script to implement our vision on how we can benefit from custom UML-to-OWL conversion, taking into account abilities of OWL. 

### ADE* classes/ade* properties removal

As far as the implementation of Application Domain Extensions (ADEs) is optional, we decided to remove mentioning of ADE* classes/ade* properties from CityGML ontologies.

Here is the typical example of a restriction involving ADE* class and ade* property:

```turtle
luse:LandUse a owl:Class ;
    rdfs:label "LandUse"@en ;
    rdfs:subClassOf [ a owl:Restriction ;
            owl:allValuesFrom luse:ADEOfLandUse ;
            owl:onProperty luse:adeOfLandUse ],
    ...
luse:adeOfLandUse a owl:ObjectProperty ;
    rdfs:label "adeOfLandUse"@en ;
    rdfs:range luse:ADEOfLandUse ;
    skos:definition "Augments the LandUse with properties defined in an ADE."@en .
luse:ADEOfLandUse a owl:Class ;
    rdfs:label "ADEOfLandUse"@en ;
    iso19150-2:isAbstract true ;
    skos:definition "ADEOfLandUse acts as a hook to define properties within an ADE that are to be added to a LandUse."@en .
```
We apply one SPARQL query ([#1 in the file](./scripts/update-triples.sh)) across all ontologies in `stage-2` to do this.

### Global Properties explication

There are some attributes named equally in different packages, such as `usage`, `class`, `function`, `address`, `value` and others, so there are `brid:Someclass.usage` and `bldg:Anotherclass.usage` that differ only on ranges of property and definitions.

In OWL object/datatype properties are first-class citizens whereas in UML and UML converted into OWL with ShapeChange they are converted into package-level and class-level attributes.

We explicate such attributes, and make a custom postprocessing step.

To explicate, we run ShapeChange twice: first - with the rule ["rule-owl-prop-globalScopeAttributes"](https://shapechange.net/targets/ontology/uml-rdfowl-based-isois-19150-2/#rule-owl-prop-globalScopeAttributes) enabled, to detect what properties are potentially duplicated.

We use the data in the folder `stage-2`, uploaded them into a knowledge graph using [Ontotext GraphDB(C)](https://graphdb.ontotext.com/), and perform the following explication query, resulting in lists of [object properties](./statistics/citygml-how%20many%20ObjectProperties%20reuse%20the%20same%20label.csv), [datatype properties](./statistics/citygml-how%20many%20DatatypeProperites%20reuse%20the%20same%20label.csv):

```sparql
select ?o (count(?s) as ?count) where { 
	?s rdfs:label ?o .
    ?s a owl:ObjectProperty .
#    ?s a owl:DatatypeProperty .
} group by ?o having (count(?s)>1)
order by desc(?count)
```

Here is an example:

```turtle
con:usage  rdf:type owl:ObjectProperty;
        rdfs:label       "usage"@en;
        rdfs:range       con:DoorUsageValue;
        skos:definition  "Specifies the actual uses of the Door."@en .
```
will be splitted into the one defined in the particular package

```turtle
common:usage  rdfs:range       con:DoorUsageValue;
        skos:definition  "Specifies the actual uses of the Door."@en .
```
and another defined once in the `common` package that we propose to be created to store all reused parts of such attributes' definitions:

```turtle
common:usage  rdf:type      owl:ObjectProperty;
        rdfs:label       "usage"@en;
```
We add the following:
- we introduce an ontology `/additional-triples/common.ttl` and a namespace `PREFIX common: <https://www.opengis.net/ont/citygml/common/>` keeping all owl:ObjectProperties and owl:DatatypeProperties sharing the same domain and varying in the `rdfs:range` (and `rdfs:label/skos:definition`). 

Then, we applied ShapeChange second time with the rule [`rule-owl-prop-globalScopeAttributes`](https://shapechange.net/targets/ontology/uml-rdfowl-based-isois-19150-2/#rule-owl-prop-globalScopeAttributes) disabled, and [`rule-owl-prop-localScopeAll`](https://shapechange.net/targets/ontology/uml-rdfowl-based-isois-19150-2/#rule-owl-prop-localScopeAll) enabled, to get complete information on class names and property names to disambiguate class names in `rdfs:subClassOf` axioms. The full configuration of ShapeChange conversion can be found [here](./scripts/CityGML3.0_to_OWL_ontotext_config.xml).

Here is the example of "ambiguity" in classes, produced by ShapeChange with `rule-owl-prop-globalScopeAttributes`. This is the expected behaviour, but we have to preserve proper class names in subclass axioms, that is why we run ShapeChange twice.

```xml
    <Warning message="Property mapping with potentially inconsistent ranges. Type of property 1 is 'BuildingRoomClassValue', while that of property 2 (to which 1 is mapped) is 'BuildingInstallationClassValue'.">
      <Detail message="--- Context - Property 1: Model::CityGML::Building::BuildingRoom::class"/>
      <Detail message="--- Context - Property 2: Model::CityGML::Building::BuildingInstallation::class"/>
    </Warning>
    <Warning message="Property mapping with potentially inconsistent definitions. Definition of property 1 is 'Indicates the specific type of the BuildingRoom.', while that of property 2 (to which 1 is mapped) is 'Indicates the specific type of the BuildingInstallation.'.">
      <Detail message="--- Context - Property 1: Model::CityGML::Building::BuildingRoom::class"/>
      <Detail message="--- Context - Property 2: Model::CityGML::Building::BuildingInstallation::class"/>
    </Warning>
    <Warning message="Property mapping with potentially inconsistent ranges. Type of property 1 is 'BuildingRoomFunctionValue', while that of property 2 (to which 1 is mapped) is 'BuildingInstallationFunctionValue'.">
      <Detail message="--- Context - Property 1: Model::CityGML::Building::BuildingRoom::function"/>
      <Detail message="--- Context - Property 2: Model::CityGML::Building::BuildingInstallation::function"/>
```

We apply four SPARQL queries ([##2-5 in the file](./scripts/update-triples.sh)) across all ontologies in `stage-2` to do this.

There are also object/datatype properties schematically presented as `packagename:Class.property`. For example, object property `boundary` has domain in some abstract class 4 times, and 9 times is has a non-abstract domain (in total, 13 occurences). Following the same argument, we would like to avoid repetitions of definitions in package-level ontologies and apply some more actions on duplicated definitions of object/datatype properties as described below.

### Removal of duplicate definitions for properties of the kind `packagename:Class.property` 

We propose to do the following (we present several alternatives):

#### 1) Relaxed semantics

Each property of the kind is defined with relaxed domain and range restriction using [schema:domainIncludes](https://schema.org/domainIncludes)/[schema:rangeIncludes](https://schema.org/rangeIncludes).

Qualifying query making use of the naming conventions in OWL generated by ShapeChange:

```sparql
#duplications of ObjectProperties 
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select 
(count(?classIndependentPropName) as ?countClasses)
  ?classIndependentPropName 
where { 
	?s a owl:ObjectProperty .
#	?s a owl:DatatypeProperty .
    optional{bind(strafter(str(?s),"#") as ?classIndependentPropName).}
    optional{bind(strbefore(strafter(str(?s),"#"),".") as ?classThePropDepends).}
	?s rdfs:label ?classIndependentPropName .
} group by ?classIndependentPropName having (count(?classIndependentPropName) > 1)
# group by ?classIndependentPropName having (count(?classIndependentPropName) = 1)
```
shows that there are two options:

- a property is unique within the whole family (i.e. there exists exactly one usage of it), for example, `tunnelFurniture`: 

```turtle
tun:AbstractTunnel.tunnelFurniture a owl:ObjectProperty ;
    rdfs:label "tunnelFurniture"@en ;
    rdfs:domain tun:AbstractTunnel ;
    rdfs:range tun:TunnelFurniture ;
    skos:definition "Relates the furniture objects to the Tunnel or TunnelPart."@en .
```

- a property is not unique within the whole family, but is used within one (package) ontology, for example, `lod0MultiCurve`:

```turtle
core:AbstractSpace.lod0MultiCurve a owl:ObjectProperty ;
    rdfs:label "lod0MultiCurve"@en ;
    rdfs:domain core:AbstractSpace ;
    rdfs:range gmlowl:MultiCurve ;
    skos:definition "Relates to a 3D MultiCurve geometry that represents the space in Level of Detail 0."@en .
...
core:AbstractThematicSurface.lod0MultiCurve a owl:ObjectProperty ;
    rdfs:label "lod0MultiCurve"@en ;
    rdfs:domain core:AbstractThematicSurface ;
    rdfs:range gmlowl:MultiCurve ;
    skos:definition "Relates to a 3D MultiCurve geometry that represents the thematic surface in Level of Detail 0."@en .
```
- a property is not unique within the whole family, and used across package ontologies, for example, `boundary`:

Property `boundary` in *construction* package is defined as:

```turtle
con:AbstractConstruction.boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
    rdfs:domain con:AbstractConstruction ;
    rdfs:range core:AbstractThematicSurface ;
    skos:definition "Relates to the surfaces that bound the construction. This relation is inherited from the Core module."@en .
```
in *core* package - as:

```turtle
core:AbstractSpace.boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
    rdfs:domain core:AbstractSpace ;
    rdfs:range core:AbstractSpaceBoundary ;
    skos:definition "Relates to surfaces that bound the space."@en .
```
in *building* package  it is used twicce and is defined as:

```turtle
bldg:BuildingRoom.boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
    rdfs:domain bldg:BuildingRoom ;
    rdfs:range core:AbstractThematicSurface ;
    skos:definition "Relates to the surfaces that bound the BuildingRoom. This relation is inherited from the Core module."@en .
...
bldg:Storey.boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
    rdfs:domain bldg:Storey ;
    rdfs:range core:AbstractThematicSurface ;
    skos:definition "Relates to the surfaces that bound the Storey. This relation is inherited from the Core module."@en .
```
The qualifying query (for `boundary`) is:

```sparql
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
select 
#?s 
?classIndependentPropName 
?classThePropDepends
?domainToInclude
?rangeToInclude
where { 
	?s a owl:ObjectProperty .
    optional{bind(strafter(str(?s),"#") as ?classIndependentPropName).}
    optional{bind(strbefore(strafter(str(?s),"#"),".") as ?classThePropDepends).}
	?s rdfs:label ?classIndependentPropName .
    ?s rdfs:domain ?domainToInclude .
    ?s rdfs:range ?rangeToInclude .
    filter (str(?classIndependentPropName)="boundary")
}
```
with the results:

| classIndependentPropName | classThePropDepends   | domainToInclude                                                        | rangeToInclude                                                           |
|--------------------------|-----------------------|------------------------------------------------------------------------|--------------------------------------------------------------------------|
| boundary                 | WaterBody             | https://www.opengis.net/ont/citygml/waterbody/WaterBody                  | https://www.opengis.net/ont/citygml/waterbody/AbstractWaterBoundarySurface |
| boundary                 | HollowSpace           | https://www.opengis.net/ont/citygml/tunnel/HollowSpace                   | https://www.opengis.net/ont/citygml/core/AbstractThematicSurface           |
| boundary                 | AuxiliaryTrafficSpace | https://www.opengis.net/ont/citygml/transportation/AuxiliaryTrafficSpace | https://www.opengis.net/ont/citygml/transportation/AuxiliaryTrafficArea    |
| boundary                 | TrafficSpace          | https://www.opengis.net/ont/citygml/transportation/TrafficSpace          | https://www.opengis.net/ont/citygml/transportation/TrafficArea             |
| boundary                 | Door                  | https://www.opengis.net/ont/citygml/construction/Door                    | https://www.opengis.net/ont/citygml/construction/DoorSurface               |
| boundary                 | Window                | https://www.opengis.net/ont/citygml/construction/Window                  | https://www.opengis.net/ont/citygml/construction/WindowSurface             |
| boundary                 | BuildingRoom          | https://www.opengis.net/ont/citygml/building/BuildingRoom                | https://www.opengis.net/ont/citygml/core/AbstractThematicSurface           |
| boundary                 | Storey                | https://www.opengis.net/ont/citygml/building/Storey                      | https://www.opengis.net/ont/citygml/core/AbstractThematicSurface           |
| boundary                 | BridgeRoom            | https://www.opengis.net/ont/citygml/bridge/BridgeRoom                    | https://www.opengis.net/ont/citygml/core/AbstractThematicSurface           |

We aim at creating a definition of the kind:

```turtle
common:boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
# all classes in domain
    schema:domainIncludes bldg:BuildingRoom, bldg:Storey, wtr:WaterBody, tun:HollowSpace, transportation:AuxiliaryTrafficSpace, transportation:TrafficSpace, con:Door, con:Window, brid:BridgeRoom ;
# all classes in range
    schema:rangeIncludes con:WindowSurface, con:DoorSurface, transportation:TrafficArea, transportation:AuxiliaryTrafficArea .
```
and in each of ontologies mentioning `boundary` (here `bldg`, `wtr`, `tun`, `transportation`, `con`, `brid`):

```turtle
common:boundary skos:definition "Here goes the definition that originate in the package, 
        for each package it will be different"
```

### 2) Strict semantics 

Each property of the kind having more than one class in its domain is defined with [owl:unionOf](https://www.w3.org/TR/owl-ref/#unionOf-def) for its range and if necessary for its domain:

```turtle
common:boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
# all classes in domain
    rdfs:domain [a owl:Class;
                   owl:unionOf (bldg:BuildingRoom, 
                                bldg:Storey, 
                                wtr:WaterBody, 
                                tun:HollowSpace, 
                                transportation:AuxiliaryTrafficSpace, 
                                transportation:TrafficSpace, 
                                con:Door, 
                                con:Window, 
                                brid:BridgeRoom)];
# all classes in range
    rdfs:range [a owl:Class;
                   owl:unionOf (con:WindowSurface, 
                                con:DoorSurface, 
                                transportation:TrafficArea, 
                                transportation:AuxiliaryTrafficArea)].
```
Enforcing strict semantics is implemented as optional step 4 of the workflow. Its results are stored in the folder `/CityRDF-strict`.
Such strict semantics makes OWL reasoning applicable, although restricts the extension of possible domains and ranges of properties in applications. 

### 3) Optimal semantics

We can also leave domain restrictions more relaxed, and define them with [schema:domainIncludes](https://schema.org/domainIncludes), but provide strict semantics for ranges:

```turtle
common:boundary a owl:ObjectProperty ;
    rdfs:label "boundary"@en ;
# all classes in domain
    schema:domainIncludes bldg:BuildingRoom, bldg:Storey, wtr:WaterBody, tun:HollowSpace, transportation:AuxiliaryTrafficSpace, transportation:TrafficSpace, con:Door, con:Window, brid:BridgeRoom ;    
# all classes in range
    rdfs:range [a owl:Class;
                   owl:unionOf (con:WindowSurface, 
                                con:DoorSurface, 
                                transportation:TrafficArea, 
                                transportation:AuxiliaryTrafficArea)].
```
The results are stored in the folder `/CityRDF-optimal`.

### Post-processing

There was detected a case when in a majority of occurences some reusable property is defined in `common` package as, e.g., owl:ObjectProperty, but in one package an owl:DatatypeProperty property exists with the same name, or vica verse. To overcome this, we do post-processing (SPARQL queries #14-#15)[./scripts/update-triples.sh], ensuring that package level attribute with the same name is used when required. E.g., we have 

```turtle
common:list  rdf:type owl:ObjectProperty; rdfs:label "list"@en .
```
and in the `core` package after all the updates done by `update-triples.sh` up to this step it appears that

```turtle
core:list a owl:DatatypeProperty ;
    rdfs:label "list"@en ;
    rdfs:domain core:DoubleList ;
    rdfs:range xsd:double ;
    skos:definition "Specifies the list of double values."@en .
...
core:DoubleList a owl:Class ;
    rdfs:label "DoubleList"@en ;
    rdfs:subClassOf [ a owl:Restriction ;
            owl:allValuesFrom xsd:double ;
            owl:onProperty <https://www.opengis.net/ont/citygml/common/list> ],
        [ a owl:Restriction ;
            owl:onDataRange xsd:double ;
            owl:onProperty core:list ;
            owl:qualifiedCardinality "1"^^xsd:nonNegativeInteger ] ;
    skos:definition "DoubleList is an ordered sequence of double values."@en .
```
Here usage of `common:list` is wrong because it is the owl:ObjectProperty. 
This post-processing step ensures that in axioms the locally defined property is used.

```turtle
core:DoubleList a owl:Class ;
    rdfs:label "DoubleList"@en ;
    rdfs:subClassOf [ a owl:Restriction ;
            owl:allValuesFrom xsd:double ;
            owl:onProperty core:list],
        [ a owl:Restriction ;
            owl:onDataRange xsd:double ;
            owl:onProperty core:list ;
            owl:qualifiedCardinality "1"^^xsd:nonNegativeInteger ] ;
    skos:definition "DoubleList is an ordered sequence of double values."@en .
```

## Samples

We prepare several small examples of BIMs in GML taking freely available data from GitHub repository [OloOcki/awesome-citygml](https://github.com/OloOcki/awesome-citygml?tab=readme-ov-file), but having in mind the real cases from [ACCORD project](https://accordproject.eu) in particular:

- [building-lod3-sample-fail.gml](./examples/building-lod3-sample-fail.gml) taken from [Ingolstadt, Germany](https://github.com/savenow/lod3-road-space-models/blob/main/models/building/lod3/combined/citygml/lod3_building_models.gml)  
- [building-lod3-sample1-ok.gml](./examples/building-lod3-sample1-ok.gml) taken from [Ingolstadt, Germany](https://github.com/savenow/lod3-road-space-models/blob/main/models/building/lod3/combined/citygml/lod3_building_models.gml) and converted into CityGML 3.0
- [building-lod3-sample2-ok.gml](./examples/building-lod3-sample2-ok.gml) taken from [Ingolstadt, Germany](https://github.com/savenow/lod3-road-space-models/blob/main/models/building/lod3/combined/citygml/lod3_building_models.gml) and converted into CityGML 3.0
- [building-lod2-sample.gml](./examples/building-lod2-sample.gml) taken from [sachsen.de, Germany](https://www.geodaten.sachsen.de/digitale-hoehenmodelle-3994.html) 

The conversion of CityGML into RDF/Turtle is conducted by custom xSPARQL script relying on the typical structure of the CityGML
```xml
CityModel
    boundedBy
        Envelope
    cityObjectMember
        Building
            ...
            genericAttribute
                StringAttribute|DoubleAttribute|...
                    name
                    value
                ...
            stringAttribute|doubleAttribute|...
                value
            ...
    lodNUMGeometry|...
        MultiSurface|CompositeSurface|...
            surfaceMember
                Polygon
                ...
```

