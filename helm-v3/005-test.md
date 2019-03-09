# Test Command

## Overview

`helm test` gives a chart author the ability to define actions on a helm release which validate that the chart is running as intended. This is useful for ensuring that a `helm install` or `helm upgrade` created/modified Kubernetes resources successfully and that the consequential `helm release` is functional.
* A **test** is defined as a command and some arguments executed from a container running in a Kubernetes Pod. If the command completes inside the container with an exit code of 0, it is successful. All other exit codes indicate the test failed.
* It is best practice to define tests in Kubernetes Manifests that live inside of the `templates/tests/` director inside of a chart.
  * Each test pod must have an annotation to denote the expected condition.
    * `helm.sh/test-expect-success: <true/false>`
      * `helm.sh/test-expect-success: true` is used to 1. denote that the pod the annotation is applied to is a test and not a resource to be created during `helm install` and 2. denote the expected outcome of the test.
    * In helm 2.x, tests used the annotation `helm.sh/hook: <test-success/test-fail>` to denote the expected condition. This will no longer be the case as it no longer makes sense to couple test definitions with hooks.
* `helm test` sub-commands
    * `helm test run <release>`
        * Executes tests by creating the Kubernetes Pods declared with the annotation `helm.sh/test-expect-success` annotation in the chart and all of its subcharts in the `charts/` directory and waits for test results
        * Adds appropriate [labels and annotations](#labels-and-annotations) to test Pods
    * `helm test cleanup <release>`
        * Deletes test Pods created in the last test run
    * `helm test results <release>`
        * Outputs results of last test run of a release

## Labels and Annotations
Helm will add the following labels and annotations to each test Pod when the tests are executed. These labels and annotations will be used by `helm test results <release>` and `helm test cleanup <release>` to display the results from the test run and clean up test pods.
* Helm will add the following labels to each test pod when the test is run:
  * `helm.sh/release: <release-name>`
  * `helm.sh/release-version: <release-version>`
  * `helm.sh/tested: <timestamp> `
* Helm will add the following labels to each test pod when the test is run:
  * `helm.sh/test-result-success: <true/false>`


//TODO:
* Look at using Jobs instead of Pods or both (Jacob)
* Would be usefult to turn off and on subchart/dependency tests (Jacob)
* More structured test output: table, JSON
* Code coverage
* Resource-policy for after-success auto cleanup
* Where do we want to store the output of tests
* Consider taking `helm test result` out completely
* Can create supporting resources like ConfigMaps
