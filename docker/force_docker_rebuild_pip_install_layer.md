# Force Docker rebuild a pip install layer

## Overview
<details>
    <summary> Show/Hide Details</summary>

Docker's cache is is very useful when instructions haven't changed and do not need to be run again.
However, there are occasions when docker cannot tell if a change has occurred or not. I recently had this experience when my 
Dockerfile was pip installing a package form a private repo on gitlab. i.e `RUN pip install git+https://${USER}:{TOKEN}@gitlab.example.com/....`.

</details>

## Retrieving the latest SHA commit
<details>
    <summary> Show/Hide Details</summary>

To force Docker to rebuild this layer everytime there is a new commit we can retireve the latest commit to a specified branch and echo this in our
Dockerfile.

To identify our latest sha commit we can run the following command:
`git ls-remote https://${USER}:{TOKEN}@gitlab.example.com/.... refs/heads/$(BRANCH)`

</details>

## All steps to force docker to rebuild a pip install layer
<details>
    <summary> Show/Hide Details</summary>

To run the full procedure we need to do the following:
1. Create env variables for `USER`, `TOKEN` and `BRANCH`
2. Use a script such as a Makefile to make the process easier and include the env variables
3. Create a variable for your repo:
    `REPO=https://${USER}:{TOKEN}@gitlab.example.com/....`
4. Create a variable for your latest SHA commit:
   `LATEST_SHA_COMMIT=$(shell git ls-remote $(REPO) refs/heads/$(BRANCH))`
5. Pass `REPO`,`BRANCH`, and `LATEST_SHA_COMMIT` to your Dockerfile as a build argument:
   
    ```
    docker build -t $(IMAGE_NAME) \
        --build-arg LATEST_SHA_COMMIT="${LATEST_SHA_COMMIT}" \
        --build-arg BRANCH="${BRANCH}" \
        --build-arg REPO="$(REPO)" .
    ```

6. Echo `LATEST_SHA_COMMIT` prior to installing the git repo in your Dockerfile:
   
    ```
    ...
    ...
    ARG REPO
    ARG BRANCH
    ARG LATEST_SHA_COMMIT
    RUN echo "${LATEST_SHA_COMMIT}"
    RUN pip install git+${REPO}@${BRANCH}
    ...
    ...
    ```

This will force Docker to pip install the git repo if it comes across an SHA commit that it hasnt seen before, as all steps after 
`RUN echo "${LATEST_SHA_COMMIT}"` will be run again.

</details>

