---
title: Limitations of Neurobagel data model
tags: [Data model]

---

During https://github.com/neurobagel/bulk_annotations/issues/10 
we discovered a couple of limitations of our data model.
The idea of this page is to keep a record of them,
so we don't have to discover the same limitation twice.
Some limiations are expected and intentional,
we don't want to model everything we can
because we want a balance of usefulness and complexity.

## Lack of longitudinal phenotypic info
Phenotypic data is modeled at the subject level, 
not at the session level.
This is mainly due to BIDS, as longitudinal phenotypic
data in BIDS are currently not defined in the spec.

Problems this causes
 
- cannot model a change in diagnosis over time. 
If a participant has a DX of "yes autism" on session 1,
but then a DX of "no autism" on session 2,
we currently have to pick one. 
And then the participant will show up with all of
their sessions for only that sessions profile.


## Missing-value forced to be a catch-all category
For example, I have a column about diagnosis. 
The correct ontology is "SNOMED-CT".
My column has four levels:

- Schizophrenia
- Alzheimer's Disease
- Sibling with Schizophrenia

The last one is not in SNOMED (one could argue that this is more a modeling issue than an ontology issue).
But in my data dictionary, I need to map this value to something.
The only option I currently have is to declare it as 
"missing value" - which is probably not a good option.

The alternative would be to "force" annotators
to pick the "next closest" term from SNOMED-CT,
but this comes with the cost of reducing 
annotation accuracy.

## Lack of model variables to say things about:

- relations. For example, I cannot model that my participant has a sibling with a diagnosis of "Schizophrenia"
- medication status. E.g. cannot say: "my participant received Drug XYZ"
- assessments that aren't tied to a specific tool. e.g. "Body Mass Index" is not a tool in the same way that "blood pressure" is not a tool. But still relevant info
- "assessments" and "diagnosis" are probably too narrow. 
E.g.: I can't really expect cognitive atlas to have a purely clinical test battery like the MS EDSS scores.
Diagnosis is too narrow because we can differentiate "Diagnosis" from "Finding" or "Observation". 
An example is "Smoker" vs "Cancer". The former is an observation, the latter a diagnosis.
**Importantly**: SNOMED has categories for observation **and** disorders.
So we cannot deduce from a term being in the SNOMED namespace that is a diagnosis.
(e.g. "Smoker" exists in snomed - snomed:77176002)