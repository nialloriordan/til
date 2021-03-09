A Makefile is often used for convenience when automating deployment.

Often docker containers require complex docker arguments, env files, prouction yaml files which can be confusing for users. 
To reduce human error, a Makefile can be used to provide specific targets depending on the task e.g `make build` to build images or `make run-dev` to run containers in a dev env.

By making a self documented Makefile the user can type `make` and receive a full list of all possible targets and what their purpose is.

```
build      build image locally
run-dev    run the docker services locally
run-prod   run the docker services in prod
```

To do this we need to add a help target to our Makefile and some shell scripting.

```Makefile
.DEFAULT_GOAL := help 

help:
	@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
```

Note:
* The `.DEFAULT_GOAL := help` tells us to set `help` as the default target.
* `@grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'` is our shell script to self document our makefile based on the comments that are added beside our makefile target (more info below).

For demo purposes I have created a simple Makefile for building images, running docker services in a dev and prod environment.

```Makefile
.DEFAULT_GOAL := help 

help:
	@egrep -h '\s#\s' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?# "}; {printf "\033[36m%-20s\033[0m %s\n", $$1, $$2}'

build: # build image locally
	docker build -f Dockerfile .

run-dev: # run the docker services locally
	docker-compose up

run-prod: # run the docker services in prod
	docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

By running make in the same directory as this Makefile, the folowing output will be returned in your terminal:
```
build      build image locally
run-dev    run the docker services locally
run-prod   run the docker services in prod
```
