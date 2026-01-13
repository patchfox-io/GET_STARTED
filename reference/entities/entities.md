# Entity Description

## package
Contains every package that has ever been detected by PatchFox. User may also refer to this as a "dependency". For record spec see [package Entity Reference](./package.md).


## finding
Pairs [findingReporter](./findingReporter.md) entity(ies) with a [findingData](./findingData.md) entity. Additionally are package records indicating which packages are tied to this finding. For record spec see [finding Entity Reference](./finding.md).


## findingData
Represents finding data including identifier, description, cpes, and summary text. For record spec see [findingData Entity Reference](./findingData.md).


## findingReporter
Represents sources if [findingData](./findingData.md). Usually an OSS scanner like Grype, Snyk, etc. For record spec see [findingReporter Entity Reference](./findingReporter.md).


## dataset
Represents a named collection of datasources. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [dataset Entity Reference](./dataset.md)


## datasetMetrics
Represents macro level metrics for a [dataset](./dataset.md) at the point the point in time an update represented by a [datasourceEvent](./datasourceEvent.md) is applied. For record spec see [datasetMetrics Entity Reference](./datasetMetrics.md)


## datasource
Represents a single datasource - ie - a single source controlled build file PatchFox is tracking. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [datasource Entity Reference](./datasource.md)


## datasourceEvent
Represents a single commit to a datasource (ie - a source controlled build file). See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [datasourceEvent Entity Reference](./datasourceEvent.md)


## edit
Represents a package change, CREATE, UPDATE, DELETE, resultant from a datasourceEvent (ie - a source controlled build file). Edit objects are additionally used by PatchFox to indicate recommended changes to a Dataset. See [PatchFox Data Nomenclature](./reference/pf_core_concepts/pf_data_nomenclature.md) for more information as to what a Dataset, Datasource, or DatasourceEvent is. For record spec see [edit Entity Reference](./edit.md).

## datasourceMetrics
For every commit to every [datasource](#datasource) a record is made to this table indicating the deltas of all the metrics tracked for all datasources in the [dataset](#dataset). In other words, dataset level metrics are tracked in the [datasetMetrics](#datasetMetrics) table and in this table are the deltas for every commit as to how those datasetMetrics numbers were impacted by any given commit. For record spec see [datasourceMetrics Entity Reference](./datasourceMetrics.md).

## datasourceMetricsCurrent
Represents the current state of every datasource from a metrics perspective in the dataset. This table contains a subset of the metrics contained in [datasetMetrics](#datasetMetrics) and [datasourceMetrics](#datasourceMetrics). Use this table to understand how the datasources connected to PatchFox comprise current datasetMetrics numbers. For record spec see [datasourceMetricsCurrent Entity Reference](./datasourceMetricsCurrent.md).
