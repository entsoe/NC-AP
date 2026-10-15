# ENTSO-E Network Code Profiles Application Profiles Library

# Overview
This library contains all applications profiles for  Network Code Profiles (NCP) organised in a folder structure. The application profiles consist of vocabulary schemas (RDFS) and constraints (SHACL) derived from the information model in UML available on the [ENTSO-E CGMES Library](https://www.entsoe.eu/data/cim/cim-for-grid-models-exchange/#_ENTSO_E_CGMES_Extension_and_Profile).

For each release the repository has PROF, RDFS and SHACL packages which contain machine readable artifacts. PROF contains the description of the profiles; RDFS is the RDF schemas and SHACL is the SHACL based constraints. 

Enhanced versions of other previous release folders can be generated under request.


### How to provide feedback
When importing any data contained in the repository, you might find bugs or issues to report. Please, open a GitHub issue and include your export log when applicable. Additionally, if needed, please write an email to [cim@entsoe.eu](cim@entsoe.eu).

Do not forget to read [CONTRIBUTING](https://github.com/entsoe/NC-AP/blob/main/.github/CONTRIBUTING.adoc) file.

### License
Please, refer to the [LICENSE](https://github.com/entsoe/NC-AP/blob/main/LICENSE.txt) for more information on the open-source license collaboration framework of the repository.

### Accreditations
List of the people and organisations contributing to this repository.

- [@HarisVranaj](https://github.com/HarisVranaj) - ENTSO-E
- [@griddigit-eu](https://github.com/griddigit-eu) - gridDigIt

# Introduction

ENTSO-E agreed to publish all application profiles in the Network Code Profiles (NCP) package, related RDFS (vocabulary) and SHACL (constraints) artifacts. All the material can be found on the CGMES Library under the Application Profiles Library tab.

The application profiles are provided to facilitate the implementation of the profiles and related constraints as defined in different standards, e.g. IEC 61970-600-1:2021, IEC 61970-600-2:2021, IEC 61970-301 and other related 61970-45x series of profiles. 

Note that the application profile serialization based on RDFS and RDF XML syntax is defined in IEC 61970-501:2006 (Ed1) and CIM XML serialization is defined in IEC 61970-552:2016. However, current implementations deviate from these standards due to various reasons. For additional information, please refer to the latest version of the [RDF-syntax User Guide](https://www.entsoe.eu/data/cim/cim-for-grid-models-exchange/#_ENTSO_E_CGMES_Extension_and_Profile) CGMES Library.


# ENTSO-E GitHub Repository Structure

ENTSO-E offers the [NCP-application-profiles-library](https://github.com/entsoe/NC-AP) GitHub Repository storing all the published CIM (NCP) related artifacts.

This current document establishes in [section 6](# Application Profiles – File Naming Convention) the naming convention used for the artifacts in the application profiles library. However, in order to exploit git functionalities for version control, the artifacts in the main branch may not include the version number in the files’ name. Every time a change is done in a file, it must increase the version number but if one does this, git will not capture the changes as it needs the file name to be static. 

Regardless of the file naming convention used exclusively for human readability, all artifacts contain version fields in the dataset header (_dcat:isVersionOf_, _dcterms:conformsTo_, etc.) which shall be used for machine readability.

## Special Mentions

### Solving inconsistency in the use of dcterms ontology

All current releases of NCP, RDFS and SHACL artifacts are updated to replace the term _dct_ with the term _dcterms_. This is done because dct was initially use but later decided not to use. This change enhances consistency.

# RDF schemas (RDFS)
The RDFS 2020 export of the RDFS augmented version is based on IEC 61970-501:2006 (Ed1). The package contains RDFS for CGMES v3.0 and NC Profiles. These are the files which file name contains “Voc-RDFS2020”.

## RDFS conventions and the RDFS ↔ SHACL division of responsibility

The RDFS2020 vocabulary is the **self-standing structural schema** of the exchange. It is serialised as flat RDF/XML — one `rdf:Description` per class or attribute, **no nested descriptions and no blank nodes** — and it defines:

- **Classes and attributes** — which classes exist and which attributes each class carries. This structural backbone lives here, and only here.
- **The class ↔ attribute linkage**, expressed so that reused vocabularies are not altered:
  - attributes **owned in a CIM namespace** (`https://cim4.eu/ns/dcatcim#`, `https://cim4.eu/ns/Metadata-European#`, `https://cim.ucaiug.io/ns#`, …) use **`rdfs:domain`** — CIM owns the term, so a strict domain is correct, even when the domain class is external (e.g. `dcatcim:preferredVersion rdfs:domain dcat:Dataset`);
  - **reused external terms attached to a class of the same vocabulary** (`dcat:keyword` on `dcat:Dataset`, `rdf:subject`/`rdf:predicate`/`rdf:object` on `rdf:Statement`) also use **`rdfs:domain`** — the linkage stays inside the owner's own vocabulary, so it restates rather than alters the owner's intent (RDF Schema itself declares `rdfs:domain rdf:Statement` for the reification properties);
  - **reused external terms attached to a foreign class** (`dcterms:`, `prov:`, `adms:` on `dcat:Dataset` or `dcatcim:Statements`) use **`schema:domainIncludes`** — schema.org's non-inferential counterpart to `rdfs:domain`, which records "expected on this class" **without** the RDFS entailment. Asserting `rdfs:domain` on a term the profile does not own is ontology hijacking (`dcterms:title rdfs:domain dcat:Dataset` would entail that every titled resource, including an ontology, is a `dcat:Dataset`); Dublin Core leaves these terms domain-free for exactly that reason. See [#92](https://github.com/entsoe/application-profiles-library/issues/92).
  - **Exception**: `dcat:startDate` and `dcat:endDate` keep **`schema:domainIncludes`** although property and class share the `dcat:` namespace — DCAT itself declares `rdfs:domain dcterms:PeriodOfTime` for both, so asserting `rdfs:domain dcat:Dataset` would contradict the owner's own declaration. The general test is: `rdfs:domain` may only be asserted where it matches, or at least cannot conflict with, what the owning vocabulary declares.
- **Simple, attribute-local constraints** — datatype (`cims:dataType`) and multiplicity (`cims:multiplicity`).

> **NB — BREAKING for `rdfs:domain`-driven tooling.** For reused external terms attached to a foreign class the linkage is now `schema:domainIncludes`, not `rdfs:domain`. Tooling that derives attribute-to-class membership must read both predicates. The `https://schema.org/` namespace is declared on the root `rdf:RDF` element.

```xml
<!-- reused external term on a same-vocabulary class — rdfs:domain restates the owner -->
<rdf:Description rdf:about="http://www.w3.org/ns/dcat#keyword">
  <cims:dataType rdf:resource="#String"/>
  <rdfs:domain rdf:resource="http://www.w3.org/ns/dcat#Dataset"/>
  <cims:multiplicity rdf:resource="http://iec.ch/TC57/1999/rdf-schema-extensions-19990926#M:0..n"/>
  <cims:stereotype>dcat</cims:stereotype>
  <rdf:type rdf:resource="http://www.w3.org/1999/02/22-rdf-syntax-ns#Property"/>
</rdf:Description>

<!-- reused external term on a foreign class — non-inferential class linkage, no hijacking -->
<rdf:Description rdf:about="http://purl.org/dc/terms/accessRights">
  <cims:dataType rdf:resource="#URI"/>
  <schema:domainIncludes rdf:resource="http://www.w3.org/ns/dcat#Dataset"/>
  <cims:multiplicity rdf:resource="http://iec.ch/TC57/1999/rdf-schema-extensions-19990926#M:0..1"/>
  <cims:stereotype>dcterms</cims:stereotype>
  <rdf:type rdf:resource="http://www.w3.org/1999/02/22-rdf-syntax-ns#Property"/>
</rdf:Description>

<!-- CIM-owned term — rdfs:domain stays -->
<rdf:Description rdf:about="https://cim4.eu/ns/Metadata-European#usedSettings">
  <cims:dataType rdf:resource="#URI"/>
  <rdfs:domain rdf:resource="http://www.w3.org/ns/dcat#Dataset"/>
  <rdf:type rdf:resource="http://www.w3.org/1999/02/22-rdf-syntax-ns#Property"/>
</rdf:Description>
```

The **Simple** SHACL constraints are generated mechanically from this RDFS (cardinalities and datatypes), so the vocabulary is the single source of truth for the structure. It follows that the structure must not migrate into SHACL: removing structural information from the RDFS without an RDFS-expressible replacement breaks every tool that generates the Simple SHACL, or an exchange schema, from the vocabulary.

**SHACL** carries only what RDFS cannot express — the **Complex** and **Validation** constraints: value ranges, conditional and cross-attribute rules, cross-profile checks and SPARQL-based logic. SHACL is the validation layer; it does not hold the structural backbone.

| Layer | Holds | Examples |
|---|---|---|
| **RDFS** (vocabulary) | structure + simple attribute-local constraints | classes, attributes, class↔attribute linkage (`rdfs:domain` / `schema:domainIncludes`), `cims:dataType`, `cims:multiplicity` |
| **Simple SHACL** | mechanical projection of the RDFS | cardinality and datatype shapes generated from the vocabulary |
| **Complex / Validation SHACL** | what RDFS cannot express | value ranges, conditionals, cross-profile, SPARQL |

(The IEC 61970-501:Ed2 beta below explores moving these simple constraints fully into SHACL; the RDFS2020 files documented here keep them in the vocabulary.)

In addition a [beta version](https://github.com/entsoe/application-profiles-library/tree/main/CGMES/CurrentRelease/RDFS/Beta_501_Ed2_CD) of application profile based on RDFS specified in the draft IEC 61970-501:Ed2 (see subfolder CGMES\RDFS\Beta_501_Ed2_CD) is provided. The purpose of inclusion of the beta version in the distribution is to enable the review process. 

Please use these files only for information on the direction where RDFS will evolve in that standard and provide feedback that will be discussed in the standardisation process. 
Namely, the RDFS contains the vocabulary part only, while the constraints (cardinalities, datatypes, etc.) are expressed by SHACL based constraints.

# Profile description (PROF)
The datasets in this package are using PROF vocabulary from W3C to describe the relationship between profiles and their components. The vocabulary is used to define the structure and semantics of the profiles, enabling interoperability and data exchange between different systems and applications.

## From instance data to SHACL validation via PROF

An instance file declares which profile it conforms to via `dcterms:conformsTo` in its `dcat:Dataset` header:

```xml
<dcat:Dataset rdf:about="urn:uuid:...">
  <dcterms:conformsTo rdf:resource="https://ap.cim4.eu/AssessedElement/2.4"/>
</dcat:Dataset>
```

The PROF descriptor for that profile (`NCP/CurrentRelease/PROF/AssessedElement-AP-RDFS2020.rdf`) declares the available resources and their roles:

```
Profile URI (e.g. https://ap.cim4.eu/AssessedElement)
  │
  ├─ prof:hasResource (role: vocabulary)
  │    └─ hasArtifact → AssessedElement-AP-Voc-RDFS2020.rdf
  │
  ├─ prof:hasResource (role: constraints)  ← Simple SHACL
  │    └─ hasArtifact → AssessedElement-AP-Con-Simple-SHACL.ttl
  │
  ├─ prof:hasResource (role: constraints)  ← Complex SHACL
  │    └─ hasArtifact → AssessedElement-AP-Con-Complex-SHACL.ttl
  │
  ├─ prof:hasResource (role: validation)   ← Validation SHACL (planned)
  │    └─ hasArtifact → AssessedElement-AP-Con-Validation-SHACL.ttl
  │
  └─ prof:hasResource (role: specification)
       └─ hasArtifact → AssessedElement_Profile_Specification.pdf
```

Each constraint resource in the PROF has a URI (e.g. `https://ap-con.cim4.eu/AssessedElement-Simple/2.4`). The corresponding SHACL file declares the same URI as its `owl:versionIRI`:

```turtle
ae:Ontology rdf:type owl:Ontology ;
    owl:versionIRI    <https://ap-con.cim4.eu/AssessedElement-Simple/2.4> ;
    owl:versionInfo   "2.4.0"@en .
```

There are two approaches to resolve which SHACL rules apply to an instance file:

1. **Via PROF** — follow the `hasArtifact` URL from the PROF constraint resource to locate the SHACL file
2. **Via SHACL headers** — load all SHACL files, read each `owl:versionIRI`, and match against the constraint resource URIs declared in the PROF for the profile that the instance conforms to

# Constraints
SHACL based constraints are provided for NC Profiles. These are the files which file name contains “Con-SHACL” or “Con-{xys}-SHACL”, where {xys} indicates the main content. For instance, “Con-Simple-SHACL” indicates that the constraints were derived from RDFS and “Con-Complex-NotSolvedMAS-SHACL” indicates that constraints have manually been produced and are applicable for a model that contains Contingency (CO) and Equipment Reliability (ER) profiles of NCP.

Please, note that constraints automatically generated from the schema are denoted as Simple. The constraints needing some manual expert modifications are denoted as Complex. In case these two scenarios are combined, this is indicated not specifying anything (nor Simple, nor Complex). 

There are constraints for cardinalities and datatypes derived from the RDFS, constraints defined in the descriptions of the classes and attributes, constraints defined in IEC 61970-600-1:2021, IEC 61970-600-2:2021, IEC 61970-301 and other related 61970-45x series of profiles and expressed there in plain English text. Note that SHACL based constraints in this folder are serialised in two RDF formats, Turtle and RDF XML plain (no nesting). Originally the constraints were developed in Turtle using Notepad++ as an editor and then converted to RDF XML using [CimPal](https://github.com/griddigit-ci/CimPal) app. 

The recommended serialisation of SHACL constraints is Turtle as that was the primary serialisation and it is well tested. This is because many constraints rely on SHACL SPARQL method, which is not covered in the draft IEC 61970-501:Ed2, the RDF XML may not represent the desired way of serialisation. However, the resulted RDF XML version was not used or validated in terms of content and should be used with a caution.

OCL based constraints are not provided. Versions of OCL constraints provided earlier are no longer maintained. Only SHACL constraints will be maintained.

# Application Profiles – File Naming Convention
Filename shall only be used for human consumption and shall not be processes to decode metadata. Machine interpretation shall use the Manifest file, that includes all the relevant metadata and reference to the dataset artifacts. The datasets are currently included in the sip file but can later be stored in a linked repository. Manifest file will be added as a technical update as soon as it is ready.

Filename convention is linked to the dataset information in the file header.
< dct:publisher>_<dct:title>_<owl:versionInfo>.<mediaType>

Examples:
* [61970-301-DiagramLayout-AP-Con-SHACL_v3-0-0.rdf](https://github.com/entsoe/application-profiles-library/blob/main/CGMES/CurrentRelease/RDFS/61970-600-2_DiagramLayout-AP-Voc-RDFS2020.rdf)
* [61970-600-2-Header-AP-Voc-RDFS2019_v3-0-0.rdf](https://github.com/entsoe/application-profiles-library/blob/main/CGMES/CurrentRelease/RDFS/61970-600-2_Header-AP-Voc-RDFS2019.rdf)
* [61970-600-2-Operation-AP-Con-Simple-SHACL_v3-0-0.ttl](https://github.com/entsoe/application-profiles-library/blob/main/CGMES/CurrentRelease/SHACL/TTL/61970-600-2_Operation-AP-Con-Simple-SHACL.ttl)

ENTSO-E does not put the publisher in the file name as it is not expected that these artifacts will be republished. 
