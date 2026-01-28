# AIDentifyAGE Ontology

This repository contains all development for the project's ontology/knowledge model.

## Competency Questions

### Judicial and  Forensic Domain

These questions validate the ontology's ability to track the legal context of an examination.
- CQ1: Which forensic expert performed the legal medical exam for a specific undocumented individual case?
- CQ2: What is the requesting entity (e.g., court or immigration service) associated with a particular forensic report?
- CQ3: On what date was the orthopantomography (OPG) procedure conducted for a given individual?

### Manual Dental Age Assessment

These questions prove the ontology can manage complex clinical staging and statistical references.
- CQ4: What tooth developmental stage was assigned to a specific tooth using the Kullman or Haavikko method?
- CQ5: Which population-specific reference studies were used to calculate the dental age assessment?
- CQ6: What are the statistically derived outputs (mean age, standard deviation, and age interval) for a manual assessment?

### AI-Based Dental Age Assessment

These questions address the explainability requirements for modern medical informatics.

- CQ7: Which specific AI model (e.g., a specific CNN architecture) was used for an inference run on a set of OPG images?
- CQ8: What were the configurations (ModelCharacteristics) for the model(s) used in a specific AI dental age assessment?
- CQ9: Did the AI model perform a regression task (continuous age) or a classification task (relative to a legal age threshold)?

### Interoperability and Data Integrity

These questions demonstrate compliance with FAIR principles and external standards.
- CQ10: How is a specific tooth represented across different international naming schemes (e.g., FDI, UNS, Palmer, or Haderup)?
- CQ11: Which external biomedical ontologies (e.g., OBI, OHD, or SNOMED-CT) provide the semantic definition for a given term in the forensic workflow?

## Competency Question–Driven SPARQL Queries

Consider for the following SPARQL queries the prefixes below.

```
PREFIX aida:   <https://aidentifyage.github.io/ontology/AIdentifyAGE#>
PREFIX dc:     <http://purl.org/dc/terms/>
PREFIX foaf:   <http://xmlns.com/foaf/0.1/>
PREFIX mls:    <http://www.w3.org/ns/mls#>
PREFIX obo:    <http://purl.obolibrary.org/obo/>
PREFIX rdfs:    <http://www.w3.org/2000/01/rdf-schema#>
PREFIX snomed: <http://snomed.info/id/>
PREFIX xsd:    <http://www.w3.org/2001/XMLSchema#>
```

#### CQ1: Which forensic expert performed the legal medical exam for a specific undocumented individual case?

```
SELECT ?exam ?expert ?expertName
WHERE {
    ?exam a aida:LegalDentalMedicalExamData ;
        aida:forensicCaseNumber ?caseNumber ;
        aida:hasForensicExpert ?expert .
    ?expert a aida:ForensicExpertPerson ;
        foaf:name ?expertName .
    FILTER(STR(?caseNumber) = "111111")
}
```
This query identifies the forensic expert who performed a legal dental medical exam for the forensic case number ``111111''.

#### CQ2: What is the requesting entity (e.g., court or immigration service) associated with a particular forensic report?

```
SELECT ?report ?entity ?entityName
WHERE {
    ?report a/rdfs:subClassOf* aida:ReportData ;
        aida:containsData ?exam .
    ?exam a aida:LegalDentalMedicalExamData ;
        aida:forensicCaseNumber ?caseNumber ;
        aida:hasRequestingEntity ?entity .
    ?entity a foaf:Organization ;
        aida:requestingEntityName ?entityName .
    FILTER(STR(?caseNumber) = "111111")
}
```
This query retrieves the institutional requesting entity associated with the forensic report number ``111111''.

#### CQ3: On what date was the orthopantomography (OPG) procedure conducted for a given individual?

```
SELECT ?individual ?opg ?date
WHERE {
    ?exam a aida:LegalDentalMedicalExamData ;
        aida:hasIndividual ?individual ;
        aida:hasRadiographicImaging ?opg .
    ?individual a aida:ForensicIndividualCasePerson ;
        foaf:name ?name .
    ?opg a aida:OPG ;
        aida:radiographicTakenDatetime ?date .
    FILTER(STR(?name) = "Individual 01")
}
```
This query retrieves the acquisition date of an orthopantomography (OPG) performed for a specific individual, namely ``Individual 1''.

#### CQ4: What tooth developmental stage was assigned to a specific tooth using Kullman or Haavikko method?

```
SELECT ?tooth ?mth ?stage
WHERE {
    ?report a aida:DentalAgeAssessment ;
        aida:hasReferenceStudyResult ?refStudyResult .
    ?refStudyResult a aida:ReferenceStudyResult ;
        aida:hasScoring ?scoring .
    ?scoring a/rdfs:subClassOf* aida:ScoringMethod ;
        aida:hasToothStage ?toothStage .
    ?toothStage a/rdfs:subClassOf* aida:ToothStage ;
        aida:hasStage ?stage ;
        aida:hasTooth ?tooth .
    ?stage a/rdfs:subClassOf* ?mth .
    VALUES ?mth {aida:HavvikkoStages aida:KullmanStages}
    ?fdi a aida:FDIDentalClassificationSystem ;
       obo:IAO_0000136 ?tooth ;
       aida:toothLabel "13" .
}
```
The corresponding SPARQL query retrieves the specific tooth evaluated, namely tooth 38, the developmental stage assigned, and the staging method applied.


#### CQ5: Which population-specific reference studies were used to calculate the dental age assessment?

```
SELECT ?assessment ?referenceStudy ?authorName ?year
WHERE {
    ?assessment a aida:DentalAgeAssessment ;
        aida:containsData ?exam ;
        aida:hasReferenceStudyResult ?result .
    ?exam a aida:LegalDentalMedicalExamData ;
        aida:forensicCaseNumber ?caseNumber .
    ?result a aida:ReferenceStudyResult ;
        aida:hasReferenceStudy ?referenceStudy .
    ?referenceStudy a aida:ReferenceStudy ;
        aida:author ?authorName ;
        aida:studyYear ?year .
    FILTER(STR(?caseNumber) = "111111")
}
```
This query retrieves the population‑specific reference studies used in a dental age assessment for a given forensic case, namely case number ``111111'', including author and publication year.

#### CQ6: What are the statistically derived outputs (mean age, standard deviation, and age interval) for a manual assessment?

```
SELECT ?assessment ?std ?mean ?min ?max
WHERE {
    ?assessment a aida:DentalAgeAssessment ;
        aida:containsData ?exam ;
        aida:standardDev ?std ;
        aida:meanAge ?mean ;
        aida:minimumProbableAge ?min ;
        aida:maximumProbableAge ?max .
    ?exam a aida:LegalDentalMedicalExamData ;
        aida:forensicCaseNumber ?caseNumber .
    FILTER(STR(?caseNumber) = "111111")
}
```
This query retrieves the mean age, standard deviation, and age interval produced by a manual dental age assessment for the specific forensic case ``111111''

#### CQ7: Which AI model was used for an inference run on a set of OPG images?

```
SELECT ?assessment ?inference ?modelName ?dataCollection
WHERE {
    ?assessment a/rdfs:subClassOf* aida:AIDentalAgeAssessment ;
        aida:hasInferenceRun ?inference .
    ?inference a aida:InferenceRun ;
        aida:hasModel ?model ;
        aida:hasDataCollectionInput ?dataCollection .
    ?model a mls:Model ;
        mls:hasQuality ?modelQuality .
    ?modelQuality a mls:ModelCharacteristic ;
        dc:title "name" ;
        mls:hasValue ?modelName .
}
```
This SPARQL query retrieves the inference run linked to an AI‑based dental age assessment, identifies the specific machine‑learning model employed, and captures its name together with the input data collection. 

#### CQ8: What were the configurations (ModelCharacteristics) for the model(s) used in a specific AI dental age assessment?

```
SELECT ?model ?propertyName ?propertyValue
WHERE {
    ?rep a/rdfs:subClassOf* aida:AIDentalAgeAssessment ;
        aida:containsData ?exam ;
        aida:hasInferenceRun ?inferenceRun .
    ?exam a aida:LegalDentalMedicalExamData ;
        aida:forensicCaseNumber ?caseNumber .
    ?inferenceRun a aida:InferenceRun ;
        aida:hasModel ?model .
    ?model a mls:Model ;
        mls:hasQuality ?quality .
    ?quality a mls:ModelCharacteristic ;
        dc:title ?propertyName ;
        mls:hasValue ?propertyValue .
    FILTER(STR(?caseNumber) = "111111")
}
```
This query retrieves the configuration parameters of AI models used in a specific dental age assessment, namely for the forensic case number ``111111''

#### CQ9: Did the AI model perform a regression task (continuous age) or a classification task (relative to a legal age threshold)?

```
SELECT ?report ?model ?task
WHERE {
    ?report a/rdfs:subClassOf* aida:AIDentalAgeAssessment ;
        aida:hasInferenceRun ?inferenceRun .
    ?inferenceRun a aida:InferenceRun ;
        aida:hasModel ?model .
    ?model a mls:Model ;
        mls:hasQuality ?quality .
    ?quality a mls:ModelCharacteristic ;
        dc:title "task" ;
        mls:hasValue ?task .
}
```
This query identifies whether an AI model performs a regression or classification task.

#### CQ10: How is a specific tooth represented across different international naming schemes (e.g., FDI, UNS, Haderup or Palmer)?

```
SELECT ?toothName ?fdi ?uns ?haderup ?palmer
WHERE {
    ?fdi a aida:FDIDentalClassificationSystem ;
        obo:IAO_0000136 ?tooth ;
        aida:toothLabel "18" .
    ?uns a obo:OHD_0000100 ;
        obo:IAO_0000136 ?tooth .
    ?haderup a aida:HaderupDentalClassificationSystem ;
        obo:IAO_0000136 ?tooth .
    ?palmer a aida:PalmerDentalClassificationSystem ;
        obo:IAO_0000136 ?tooth .
    ?tooth rdfs:label ?toothName .
}
ORDER BY ?fdi
```
This query demonstrates how a single tooth is represented across multiple international dental numbering systems, highlighting AIdentifyAGE’s support for cross‑standard semantic interoperability.


#### CQ11: Which external ontologies provide the semantic definition for a given term?

```
SELECT ?term ?ontology
WHERE {
  ?term rdfs:isDefinedBy ?ontology .
}
```
This query identifies the external ontologies from which semantic definitions are reused, demonstrating that AIdentifyAGE maintains explicit provenance and interoperable links to ontological sources.

## Traceability between Competency Questions and Ontology Axioms

Each competency question is directly supported by explicit ontology axioms.
Object properties such as _has Forensic Expert_, _has Reference Study_, _hasQuality_, and _has Inference Run_ ensure that legal
responsibility, scientific provenance, statistical reasoning, and AI
explainability are formally represented. Data properties such as _Mean Age_, _Standard Deviation_, and _Forensic Case Number_ allow
quantitative clinical outputs to be queried and validated. This traceability
guarantees that the ontology structure is driven real forensic and medical
decision requirements rather than abstract conceptual modeling.

Table that reflects the traceability between competency questions and ontology axioms.

| CQ | Required Classes | Required Object / Data Properties | Ontological Role |
|----|------------------|-----------------------------------|------------------|
| CQ1 | Legal Dental Medical Exam Data, Forensic Expert Person | has Forensic Expert, Forensic Case Number, foaf: name | Links a forensic case to the responsible professional |
| CQ2 | Report Data, Legal Dental Medical Exam Data, foaf:Organization | contains Data, has Requesting Entity, rdfs:subClassOf, Forensic Case Number, Requesting Entity Name | Supports legal provenance and institutional responsibility |
| CQ3 | Legal Dental Medical Exam Data, Forensic Individual Case Person, OPG | has Individual, has Radiographic Imaging, Radiographic Taken DateTime, foaf:name | Captures medical imaging temporal provenance |
| CQ4 |  Dental Age Assessment, Reference Study Result, Scoring Method, Tooth Stage, obo:Tooth, Haavikko Stages, Kullman Stages | has Reference Study Result, has Scoring, has Tooth Stage, has Stage, has Tooth, rdfs:subClassOf, Tooth Label | These axioms allow representing that a manual dental age assessment uses a specific staging method (Haavikko or Kullman) and assigns a developmental stage to a specific tooth. The object property \textit{has Reference Study Result} links a reference study result to an assessment, while \textit{has Tooth Stage} make explicit which tooth is evaluated and which developmental stage is attributed, being aggregated inside a scoring method.|
| CQ5 | Dental Age Assessment, Legal Dental Medical Exam Data, Reference Study Result, Reference Study | has Reference Study Result, has Reference Study, author, year | Ensures population-specific scientific grounding |
| CQ6 | Dental Age Assessment, Legal Dental Medical Exam Data | Mean Age, Standard Deviation, Minimum Probable Age, Maximum Probable Age | Encodes statistical reasoning outputs |
| CQ7 | AI Dental Age Assessment (AI Dental Age Threshold Assessment or AI Reg Dental Age Assessment), Inference Run, mls:Model, mls:ModelCharacteristic | hasInference Run, has Model, has Data Collection Input, mls:hasQuality, dct:title, mls:hasValue, rdfs:subClassOf | These axioms support the explicit traceability between an Al-based dental age assessment and the concrete Al model used for inference. The property \textit{has Model} formally links the inference run to a specific CNN or ML model instance, ensuring model-level accountability and reproducibility.|
| CQ8 | AI Dental Age Assessment (AI Dental Age Threshold Assessment or AI Reg Dental Age Assessment), Legal Dental Medical Exam Data, Inference Run, mls:Model, mls:ModelCharacteristic | contains Data, has Inference Run, has Model, mls:hasQuality, rdfs:subClassOf, Forensic Case Number, dc:title, mls:hasValue | Makes Al configuration transparent |
| CQ9 | AI Dental Age Assessment (AI Dental Age Threshold Assessment or AI Reg Dental Age Assessment), InferenceRun, mls:Model, mls:ModelCharacteristic | has Inference Run, hasModel, mls:hasQuality, rdfs:subClassOf, dc:title, mls:hasValue | Makes Al configuration transparent |
| CQ10 | FDI Dental Classification System, obo:'universal tooth numbering', Haderup Dental Classification System, Palmer Dental Classification System | obo:IAO_0000136, Tooth Label, rdfs:label | These axioms enable the representation of a tooth under multiple international dental numbering systems. Each tooth can be associated with several Dental Classification System instances, each linked to a specific naming scheme and its corresponding notation value, ensuring cross-standard semantic interoperability.| 
| CQ11 | | rdfs:isDefinedBy | Enables ontology provenance and FAIR reuse |


## Interoperability Mapping with External Ontologies

This appendix presents the explicit mapping between AIdentifyAGE concepts and
corresponding entities from established external ontologies and standards,
including OBI, OHD, MLS, FOAF, DC Terms, W3C Time, and SNOMED-CT. This table is
focused on upper‑level entities that ensure semantic alignment across the
forensic dental age assessment workflow. The mapping demonstrates how
AIdentifyAGE ensures semantic consistency, data exchangeability, and
interoperability across clinical, forensic, and AI-based decision support
systems.

Domai-specific subclasses--such as individual reference studies (e.g., Mincer
Reference Study) and specific developmental staging systems (e.g., Nolla
Stages).

Interoperability mapping with external ontologies. Most of the concepts
presented on this table are mapped as subclasses of classes of external
ontology. The concepts marked with $*$ are reused directly on AIdentifyAGE, the
ones marked with $\dagger$ are defined as equivalent entities to existent
entities in external ontologies.

| AIdentifyAGE Concept | External Ontology | External Term | Interoperability Purpose |
|----------------------|-------------------|---------------|--------------------------|
| Forensic Expert Person | FOAF | foaf:Person | Standard representation of human agents |
| Forensic Individual Case Person | FOAF | foaf:Person | Standard representation of human agents |
| Forensic Expect Role  | BFO | BFO:0000023 | Standard representation of human roles |
| Requesting Entity | FOAF | foaf:Organization | Representation of courts, immigration services, institutions |
| Legal Medical Exam Data | OGMS  (via OBI)| OGMS:0000123 (clinical data item) | Represents Clinical Data |
| OPG$\dagger$ | SNOMED-CT | Orthopantomogram | Dental radiographic modality standardization |
| Tooth | OHD | OHD: 0000002 (tooth) | Anatomical dental entity |
| Tooth Stage | OBI | OBI:0000938 (categorical measurement datum) | Represents Development Stage associated with a Tooth|
| Data Collection | IAO | IAO:0000100 (dataset) | Representation of a set of Clinical Data |
| Demirjian Maturity Scoring | IAO | IAO:0000100 (dataset) | Representation of Demirjian tooth-``maturity scoring'' mapping |
| Reference Study | IAO | IAO:0000100 (dataset) | Represents Dental Age Assessment study |
| Reference Study Result | OBI | IAO:0000109 (measurement datum) | Represents the result of applying a reference study to a teeth set |
| Scoring Method | OBI | IAO:0000104 (plan specification) | Represents aggregation of a set of tooth-development stage |
| Stage | OBI | IAO:0000104 (plan specification) | Scientific method modeling |
| Treatment Option | OBI | IAO:0000104 (plan specification) | Representation of dental treatments |
| Teeth Set | UBERON (via OBI) | UBERON:0000165 (mouth) | Represents a set of teeth |
| ModelCharacteristic* |  MLS | mls:ModelCharacteristic | Model details representation |
| ModelOutput |  MLS | mls:InformationEntity | Model output produced during an inference run over an OPG |
| InferenceRun |  MLS | mls:InformationEntity | Model execution over a given OPG |
| Report Data | OBI | IAO:0000027 (data item) | Medical/forensic report documentation |
| Demirjian Coefficient Maturity Data | OBI | IAO:0000027 (data item) | Represents a data entry for a Demirjian tooth maturity score reference study table |
| Data Reference Study | OBI | IAO:0000027 (data item) | Data entry from a Reference Study|
| Model* | MLS | mls:Model | AI model interoperability |
| Inference Run | MLS | mls:Run | Run Execution of AI inference |

|![refstudy_hier.png](https://github.com/AIdentifyAGE/ontology/blob/main/imgs/hierarchy/refstudy_hier.png?raw=true)|
|----|
| Figure: A hierarchical ontological structure that classifies individual reference studies under the Reference Study class. Each subclass of the Reference Study class, corresponds to a specific published study. The arrows define rdfs:subClassOf property relations. |

|![stage_hier.png](https://github.com/AIdentifyAGE/ontology/blob/main/imgs/hierarchy/stage_hier.png?raw=true)|
|----|
|Figure: A hierarchical ontological structure that classifies individual stages under the Stage class. Each subclass of the  Stage, corresponds to a specific  developmental stage used in forensic dental age assessment. The arrows define rdfs:subClassOf property relations.|

|![toothscoring_hier.png](https://github.com/AIdentifyAGE/ontology/blob/main/imgs/hierarchy/toothscoring_hier.png?raw=true)|
|----|
| Figure: A hierarchical ontological structure that classifies individual stages under the Scoring Stage class. Each subclass of the  Scoring Stage, corresponds to a specific set of steps of dental development. The arrows define rdfs:subClassOf property relations.|

|![toothstage_hier.png](https://github.com/AIdentifyAGE/ontology/blob/main/imgs/hierarchy/toothstage_hier.png?raw=true)|
|----|
|Figure: A hierarchical ontological structure that classifies individual stages under the Tooth Stage class. Each subclass of the  Stage, corresponds to a specific scientifically established dental age assessment staging system. The arrows define rdfs:subClassOf property relations.|


## Accessing SPARQL in Protégé

1. Launch Protégé: Open the Protégé application on your machine. If you haven't
   installed it yet, you can find the latest version at
   https://protege.stanford.edu/. This ontology was developed using the
   Protégé version 5.6.5.

2. Open Your Ontology File: Once the application is running:
   - Navigate to the top menu and select `File` $\rightarrow$ `Open...`
   - Browse your local directory for the ontology file ANONYMOUS.owl,
     containing individuals to support the Competency Questions. Common
     extensions include `.owl`, `.rdf`, or `.ttl`.
   - Alternatively, if your file is hosted online, select `File` $\rightarrow$
     `Open from URL...` and paste the link.

3. Locate the SPARQL Query Tab: By default, the SPARQL tab may not be visible in the main dashboard.
   - Look at the horizontal tabs at the top (e.g., Entities, Classes, Object Properties).
   - If you do *not* see a tab labeled *SPARQL Query*, navigate to:
   - `Window` $\rightarrow$ `Tabs` $\rightarrow$ `SPARQL Query`, as shown below.

4. Initialize the Query Environment: Clicking the *SPARQL Query* tab
   will open the query editor. You are now ready to paste and execute the
   provided `SPARQL` queries against loaded data, as shown below.

|![sparql_menu_tab.png](https://github.com/AIdentifyAGE/ontology/blob/main/imgs/how_to/sparql_menu_tab.png?raw=true)|
|----|
|Figure: SPARQL Query tab location in Protégé `Window` menu.|

| ![sparql_tab.png](https://github.com/AIdentifyAGE/ontology/blob/main/imgs/how_to/sparql_tab.png?raw=true) |
|---|
|Figure: SPARQL Query tab with example query. This query corresponds to the Competency Question 1.|

