# How we reference information and sources therein

For every git commit to every build file PatchFox is instructed to monitor, PatchFox ingests that datum as part of a time series and associates it with one or more "Datasets". 

A "Dataset" is a collection of "Datasources", which is what PatchFox calls a build file it's monitoring. A "Datasource" is the "source" of "events", which PatchFox calls "Datasource Events". 

It enriches the data with information from third party sources like OSS scanners and package indexes. It then tabulates top line metrics describing the state of the "Dataset" 


PatchFox uses the purl spec to refer to software packages - ie "dependencies" 

We also use the same to refer to the sources of said packages and the datums that come out of those sources. 

The purl format looks like this:

```scheme:type/namespace/name@version?qualifiers#subpath```

"scheme" and "type" will always be "pkg" and "generic" respectively 


In PatchFox, in reference to our use of pURL to represent the sources of data, there are three data concepts that are important to understand: 

* DATASOURCE - is exactly what it sounds like - the name of the thing pumping data into the pipeline. In the pURL it's denoted in the "name" and the "version" fields. It's usually a git-tracked build file. 

  We use "__" to denote the directories in which the build file was discovered. The git branch from which the data emanates after the location of the build file using a :: delimiter. Finally we note the type (according to pURL spec) of the file. 
  * example purl: `pkg:generic/github/codeql__java__ql__integration-tests__java__maven-sample-xml-mode-byname%3A%3Amain@maven`

* DATASET - is a collection of "datasources" This is not denoted as a purl, only by name. In this case - and in keeping with the example above from respository "codeql", the name of the dataset is "github" which occupies the "namespace" pURL field. 

* DATASOURCE_EVENT - is a datum of information that eminates from a "datasource" Usually it's an SBOM, the git-annotated build file, and some metadata representing the state of the dependencies represented by the build file at a given git commit. Note how for a "datasource_event", we use the same pURL identifier as we do for the "datasource" from which the datum eminated with additional qualifiers indicating the commit datetime and the commit hash. In the pURL it occupies the "qualifiers" field.
  * example purl: `pkg:generic/github/codeql__java__ql__integration-tests__java__maven-sample-xml-mode-byname%3A%3Amain@maven?commitdatetime=2024-08-30T08%3A28%3A25%2B00%3A00&commithash=321820e758b86827f6e19ab3d7226d70f9ad982c`


## Metrics 
For every processed DatasourceEvent PatchFox will generate a record describing, in numerical terms, the overall state of the Dataset after the DatasourceEvent was applied. Many of these metrics are ones you would expect. Thigs like "how many packages are in the Dataset" and "how many findings have been in the Dataset for more than a month". These formulate a rich time-series that we use to monitor, forecast, and make recommendations as to how to manage your dependencies. 

There are a set of metrics that are unique to PatchFox. Part of our special sauce if you will. See [PatchFox Custom Metrics](./custom_metrics.md) for details. 