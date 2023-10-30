## Bitbucket Pipelines

### configuration reference

- ref: https://support.atlassian.com/bitbucket-cloud/docs/bitbucket-pipelines-configuration-reference/

- global options: optional

- git clone behavior to control clone behavior, optional

- definitions: defines resources used elsewhere in the pipelines. Resources include:
    - caches
    - services
    - YAML anchors used to define steps and reuse the defined steps. Example: `&build-test` defines a step that includes name, script and artifacts. This step can then be repeated by calling `*build-test` in pipelines.
        ```yaml
        definitions: 
          steps:
            - step: &build-test
                name: Build and test
                script:
                  - mvn package
                artifacts:
                  - target/**

        pipelines:
          branches:
            develop:
              - step: *build-test
            main:
              - step: *build-test
        ```
        
- secured variable to store username and password
    - https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/
    - set workspace variables to be used in all repositories, or 
    - set repository varaibles to be used by this repo. For example the variables name is `MY_VAR` and value is "xxxyyy". The value is hidden and nobody can see it after being created. To use the variable in pipelines, use `$MY_VAR`. 



## QA

### QA: how to set default branch for Bitbucket repos?

From Repository Settinging --> Repository details --> Advanced --> main branch to select one.

