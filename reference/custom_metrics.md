## custom metrics 

In addition to the standard package management measurements like quantity of findings, time findings have been in the backlog, etc, PatchFox presents a set of metrics at the dataset level that have analytic value germane to managing a corpus of packages across an organization. These are recorded in the `dataset_metrics` table and exposed by way of the data-service's API. 

Here is an explanation of what those metrics mean to PatchFox

* *Downlevel* refers to a package that is not the most current version offered by the vendor 

* *Stale* refers to a package that has not been updated by the vendor in [x] time. Note this is different than *downlevel* in that a *stale* package can be the most recent version but the vendor may have put out that version a year ago. 

* *Redundant Package Score (RPS)* in essence, this measure indicates the percentage of packages in the dataset that are different versions of the same thing. The measure goes from 0 -> 100. For example, if a dataset is comprised wholly of unique versions of "foo" then the RPS score will be 100. Conversely if every package in the dataset is a unique package type (a package sans version) then the RPS score is 0. RPS score has analytic value because there is a strong correlation between a dataset with a high RPS score and the liklihood that new findings will be associated with the dataset in future. A low RPS score is desired because it's an indication that there's reduced liklihood of findings being present in the dataset in future. TO BE EXPLICITLY CLEAR - THIS IS A MEASURE OF DIFFERENT VERSIONS OF THE SAME THING NOT NUMBER OF INSTANCES
OF THE SAME VERSION OF A THING. For example - if the dataset is comprised of ten INSTANCES of jackson-databind v1.1.1 the RPS score is ZERO. Conversely if the dataset is comprised of ten DIFFERENT VERSIONS of jackson-databind the RPS score is 100. 

* *Patch Effort/Impact/Efficacy* One of the things PatchFox endeavors to do is help organizations work smarter, not harder. The idea is to reduce the amount of effort being expended towards package management and maximize the impact (ie the benefit) of said effort. *Patch Efficacy Score (PES)* is a measure of the ratio of *Patch Impact / Patch Effort*. A high PES score is desireable because it indicates the organization is maximizing positive impact for every patch it makes. 
  * *Patch Effort* is considered to be "1" per patch by default. That number is diminished when multiple patches of the same kind are made within a 90 day period. The rationale is that when an organization performs the same task multiple times in a given 90 day period the task becomes easier with each successive time because humans tend to get better at things when they do it a lot. 
  * *Patch Impact* refers to the benefit of making a given Patch. When a patch reduces findings, avoids future findings, lowers *RPS* score, reduces *stale* or *downlevel* packages, the impact measure goes up. 

