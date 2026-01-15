# GET STARTED
How to get started running PatchFox on your workstation 🦊

## INSTALL WITH GENAI
We've had success using claude code to install PatchFox on a workstation simply by pointing it to this document and asking it to install PatchFox. If you have access to Claude Code or a similar tool we recommend this approach as by far the easiest way to install PatchFox.

## INSTALL MANUALLY

### PREREQUISITES 

* a linux or MacOS environment
  * **NOTE** IF YOU ARE ON WINDOWS YOU NEED TO USE A FULL 'NIX IMAGE -- WSL2 WON'T WORK PROPERLY 
* java 21
* maven 3
* docker
* python >= 3.14
* some kind of SQL tool. [DBeaver](https://dbeaver.io/) is free and it works.
* [syft](https://github.com/anchore/syft) if you want to push your own data through PatchFox 

### NOT EXACTLY A PREREQUISITE BUT A GOOD IDEA 
While not strictly required PatchFox is a unique tool and a lot is going to make a lot more sense if you take a moment to read these short docs that explain what PatchFox is and how it works: 

* [tl'dr What is PatchFox?](reference/tldr_what_is_pf.md)
* [What Problem is PatchFox Addressing?](reference/pf_what_problem_addressing.md)
* [How PatchFox Represents Information](reference/pf_data_nomenclature.md)

### CLONE AND BUILD THE SERVICES 

#### CLONE THE SERVICES 
PatchFox on the backend is a flotilla of microservices and custom libraries that together constitute a data analytics pipeline. We're going to be doing a lot of cloning and building here. Buckle up. 


##### CLONE THESE FIRST - THEY DON'T NEED TO BE COMPILED 
* [GET_STARTED](https://github.com/patchfox-io/GET_STARTED/tree/main)
  * this repo is the docs repo you'll use to seed claude or kiro of whatever agent you're using with all it needs to make the PatchFox awesome happen. 
* [patch-ai](https://github.com/patchfox-io/patch-ai-service)
  * enables genAI/slack user interface to PatchFox
* [docker-compose](https://github.com/patchfox-io/docker-compose)
  * configures and orchestrates pipeline start and stop
* [etl-root](https://github.com/patchfox-io/etl-root)
  * Helper scripts used to push data into PatchFox

##### CLONE THESE NEXT AND COMPILE THEM 

Command to compile:
```
mvn clean install 
```
* [db-entities](https://github.com/patchfox-io/db-entities)
  * PatchFox library that contains all db entities used by same
* [package-utils](https://github.com/patchfox-io/package-utils)
  * PatchFox library that provides logic for processing packages (ie - dependencies)

##### CLONE THESE NEXT AND COMPILE THEM AND BUILD THEIR DOCKER IMAGES 
**NOTE** IF YOU ARE BEHIND A CORPORATE FIREWALL AND/OR YOUR ORGANIZATION USES SOMETHING LIKE ZSCALER OR NETSKOPE THAT INTERCEPTS HTTPS TRAFFIC AND DOES FUN THINGS WITH CERTIFICATES THIS MIGHT FAIL.

Command to compile and build docker image. 

```
mvn clean install jib:dockerBuild
```

* [input-service](https://github.com/patchfox-io/input-service)
  * is the port of entry for all datasource_events into the pipeline.
* [orchestrate-service](https://github.com/patchfox-io/orchestrate-service)
  * job manager 
* [grype-service](https://github.com/patchfox-io/grype-service)
  * enrichment service that performs OSS scanning on the dependency graph contained in a datasource_event.
* [package-index-service](https://github.com/patchfox-io/package-index-service)
  * enrichment service that queries the germane package index (maven central, pypi, etc) to add context to the dependency graph contained in a datasource_event.
* [analyze-service](https://github.com/patchfox-io/analyze-service)
  * aggregates top level metrics at the dataset and datasource level.
* [data-service](https://github.com/patchfox-io/data-service)
  * provides api access to the PatchFox pipeline and datastore 


#### SET UP YOUR PYTHON VIRTUAL ENVIRONMENT AND THE OTHER PYTHON THINGS
There are two python things in PatchFox. One is [etl-root](https://github.com/patchfox-io/etl-root). The other is [patch-ai](https://github.com/patchfox-io/patch-ai-service). 

For `etl-root`, you'll need to install the requirements.txt file into a local python venv. See [this](https://docs.python-guide.org/dev/virtualenvs/) for help on how to do that. It is strongly recommended you use virtual env for this as you'll need it for the next step. 

For `patch-ai`, you'll need to use a different command but use the venv from the previous step. 

```
pip install -e .
```

 
## GET THINGS GOING WITH GENAI 
**NOTE** IF YOU ARE BEHIND A CORPORATE FIREWALL AND/OR YOUR ORGANIZATION USES SOMETHING LIKE ZSCALER OR NETSKOPE THAT INTERCEPTS HTTPS TRAFFIC AND DOES FUN THINGS WITH CERTIFICATES THIS MIGHT FAIL.

Tell Claude Code (or kiro or whatever terminal coding agent you're using) to parse all the documents in this repository (which should now be cloned on your workstation). It should be able to boot up PatchFox. 


## GET THINGS GOING MANUALLY

### START THE PIPELINE 
**NOTE** IF YOU ARE BEHIND A CORPORATE FIREWALL AND/OR YOUR ORGANIZATION USES SOMETHING LIKE ZSCALER OR NETSKOPE THAT INTERCEPTS HTTPS TRAFFIC AND DOES FUN THINGS WITH CERTIFICATES THIS MIGHT FAIL.

From the `docker-compose` directory, execute command 
```
docker compose up 
```

This will bootstrap the pipeline - sans the GenAI interface. 

*NOTE* each PatchFox service proves a swagger doc describing it's interface. The data-service's can be found at `http://localhost:1702/swagger-ui.html`. The data-service proves access to all data stored in the pipeline. It is highly recommended the user access this - in particular - the db query endpoints described in the aforementioned doc 

### GET SAMPLE DATA INTO PATCHFOX
PatchFox provides database dumps of already-processed sample data sourced from public github organizations (eg github.com/reddit, github.com/ibm, etc). 

1. download the one of the sample [db dumps](https://drive.google.com/drive/folders/14rYDt4g-fMoSE2dEBx0cPwYlppTES076?usp=drive_link)
2. use the following command to copy the dump file into the PatchFox Postgres container. If you chose a different dump file, replace the filename with the name of the dump you chose. 
```
docker cp ./reddit_github_recommended_25NOV25.sql.gz docker-compose-postgres-1:/
```
3. log into the postgres container
```
docker exec -it docker-compose-postgres-1 /bin/bash 
```
4. from within the container, unzip the dump 
```
gunzip reddit_github_recommended_25NOV25.sql.gz
```
5. import the dump into postgres **VERY IMPORTANT DO THIS STEP TWICE!!!**
```
psql -U mr_data mrs_db < reddit_github_recommended_25NOV25.sql
```
6. exit out of the container and return to host terminal (did you perform step 5 twice?)
```
exit
```

At this point you should have data loaded into PatchFox. To verify you can:
* use the dataservice api to query the database (http://localhost:1702/api/v1/datasources is a good quick check)
* use your database manager (like DBeaver) to inspect the tables. The connection details are: 
  * db name: mrs_db
  * username: mr_data 
  * password: omnomdata
  * host: localhost
  * port: 54321


### GET -->YOUR<-- DATA INTO PATCHFOX 
In the etl-root project there are several scripts designed to help you load data on your workstation into PatchFox. 

0. ensure you've installed syft as per the prerequisite section of this doc. 
1. goto the `etl-raw` directory 
2. ensure you've got your python venv activated 
3. open file `env.sh` and update the env vars that are marked as needing to be updated. 
4. save `env.sh` and execute command `source env.sh` to load the vars 
5. at this point you're ready to upload data to your locally running PatchFox installation. You're going to point a helper script to a directory on your file system that contains the git repos you want PatchFox to process. 

```
run_local.sh {DIRECTORY THAT CONTAINS GIT REPOS}
```

### GET -->YOUR<-- DATA INTO PATCHFOX BY WAY OF CI/CD 
We have a [ci-cd component](https://hub.docker.com/r/patchfoxio/patchfox-etl) for that! Reach out to us to get the necessary env vars. 


### HOW MUCH DATA CAN I PUT INTO PATCHFOX? 
Quite a lot. The limitation on pipeline processing beyond available compute is time. The `analyze-service` needs to process records in ASC order by commitDateTime and thus can't be parallelized. The current implementation of PF runs efficiently per record. That being said, When the dataset contains hundreds of thousands of packages and tens of thousands of findings, it can take ~2.5m for analyze to process each record. 

That being said - PatchFox need only process a small subset of the data that gets uploaded in order to provide value. It's entirely possible tens of thousands of datasource_event's are uploaded to PatchFox but only a few hundred need to be processed. In fact, often only the HEAD events (the most recent commit to a datasource) is all you need. Interally we use [this sql script](https://github.com/patchfox-io/etl-root/blob/main/sql/fetch_n_months_of_dse_history.sql) to mark events we want to process prior to allowing the orchestrate service to start a job. In this way we're able to process a sample of the data and reduce total processing time by a gazillion. 


### A WORD ABOUT THE SLACK/CUSTOM GEN-AI INTERFACE
Beyond accessing data by way of the database directly or the data-service, PatchFox also has a genAI interface. That service is what we use to plumb genAI models into slack. It also can be used in "console" mode in case you don't want to use Claude Code/Kiro/etc. To use that interface we'll need to do a few things:

1. in the patch-ai directory, open up file `slack_shit.txt` in your favorite editor 

2. For whatever model you intend to use you'll need to ensure the appropriate API key is specified in this file. 

3. execute `source slack_shit.txt` to ensure all the necessary env vars are loaded 

4. after ensuring first that you've activate the python venv you created earlier, execute the `slackbot.py` file. 

5. in a new terminal, and again ensuring first to activate the python venv, execute the `console_chat.py` file. This terminal will be your genAI 
interface into PatchFox. 


### OTHER HELPFUL DOCS 

* [data-service api guide](reference/data_service_api.md)
* [db entities](reference/entities/entities.md)


## I'M STUCK - HOW DO I GET HELP? 
If the docs aren't helpful please feel free to open a github issue. We'll respond quickly and hopefully get you back on track 🍻
