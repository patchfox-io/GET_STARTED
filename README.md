# GET STARTED
How to get started running PatchFox on your workstation 🦊

## PREREQUISITES 
* a linux environment
  * **NOTE** IF YOU ARE ON WINDOWS YOU NEED TO USE A FULL 'NIX IMAGE -- WSL2 WON'T WORK PROPERLY 
* java 21
* maven 3
* docker

## NOT EXACTLY A PREREQUISITE BUT A GOOD IDEA 
While not strictly required PatchFox is a unique tool and a lot is going to make a lot more sense if you take a moment to read these short docs that explain what PatchFox is and how it works: 

** TODO ADD DOCS FROM DOCS FOLDER HERE FOR EASY REFERENCE ** 

## CLONE AND BUILD THE SERVICES 

### CLONE THE SERVICES 
PatchFox on the backend is a flotilla of microservices that together consitute a data analytics pipeline. **TODO** IN THIS REPO YOU ARE GOING TO USE A HELPER SCRIPT TO CLONE YOU NEED **TODO** The services you'll be cloneing are:
* input-service
  * is the port of entry for all datasource_events into the pipeline.
* orchestrate-service
  * job manager 
* grype-service
  * enrichment service that performs OSS scanning on the dependency graph contained in a datasource_event.
* package-index-service
  * enrichment service that queries the germane package index (maven central, pypi, etc) to add context to the dependency graph contained in a datasource_event.
* analyze-service
  * aggregates top level metrics at the dataset and datasource level.
* data-service
  * provides api access to the PatchFox pipeline and datastore 
* patch-ai
  * enables genAI user interface to PatchFox
* docker-compose
  * configures and orchestrates pipeline start and stop
* db-entities
  * PatchFox library that contains all db entities used by same
* package-utils
  * PatchFox library that provides logic for processing packages (ie - dependencies)
* etl-root
  * Helper scripts used to push data into PatchFox
  
 ### BUILD THE SERVICES 
 **TODO** use script $$$$$$ to build all the services and produce docker images for same. 
 
## GET THINGS GOING 

### START THE PIPELINE 
From the `docker-compose` directory, execute command 
```
docker compose up 
```
This will bootstrap the pipeline - sans the GenAI interface. To access that, execute command 
```
**TODO** PUT COMMAND HERE 
```

*NOTE* each PatchFox service proves a swagger doc describing it's interface. The data-service's can be found at `http://localhost:1702/swagger-ui.html`. The data-service proves access to all data stored in the pipeline. It is highly recommended the user access this - in particular - the db query endpoints described in the aforementioned doc **TODO DOC LINK** 

### GET SAMPLE DATA INTO PATCHFOX
PatchFox provides database dumps of already-processed sample data sourced from public github organizations (eg github.com/reddit, github.com/ibm, etc). To load that dump into your locally running PatchFox environment execute script **TODO PUT SCRIPT HERE** 

### GET -->YOUR<-- DATA INTO PATCHFOX 
In the etl-root project there are several scripts designed to help you load data on your workstation into PatchFox. The steps are **TODO** 

### GET -->YOUR<-- DATA INTO PATCHFOX BY WAY OF CI/CD 
We have a [ci-cd component](https://hub.docker.com/r/patchfoxio/patchfox-etl) for that! Reach out to us to get the necessary env vars. 

## WHAT HAPPENS NOW? 

### WHAT IS PATCHFOX DOING AND HOW LONG WILL IT TAKE? 
PatchFox 

### HOW MUCH DATA CAN I PUT INTO PATCHFOX? 
ASDF

### HOW DO I ENGAGE WITH THE DATA? 
data-service 
genAI 

### WHAT CAN PATCHFOX DO THAT OTHER TOOLS CAN'T? 
ASDF

## I'M STUCK - HOW DO I GET HELP? 
If the docs aren't helpful please feel free to open a github issue. We'll respond quickly and hopefully get you back on track 🍻







