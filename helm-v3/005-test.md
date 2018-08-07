# Test Command

## Overview

`helm test` gives a chart author the ability to define actions on a helm release which validate that the chart is running as intended. This is useful for ensuring that a `helm install` or `helm upgrade` created/modified Kubernetes resources successfully and that the consequential `helm release` is functional.
* A **test** is defined as a command and some arguments executed from a container running in a Kubernetes Pod. If the command completes inside the container with an exit code of 0, it is successful. All other exit codes indicate the test failed.
* Tests are defined in Manifests that live inside of the `templates/tests/success/` and `templates/tests/failure/` directories inside of a chart.
  * Tests that live in `templates/tests/success/` are expected to succeed.
  * Tests that live in `templates/tests/failure` are expected to fail.
  * In helm 2.x, tests used annotations to denote the expected condition. This will no longer be the case and will be replaced with the aforementioned directory placement.
* `helm test` commands
    * `helm test run <release>`
        * Executes tests by creating the Kubernetes Pods declared in the `templates/tests/success` and `templates/tests/failure` directories of the chart and all subcharts in the `charts/` directory and waits for test results
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
  * `helm.sh/test: true`
* Helm will add the following labels to each test pod when the test is run:
  * `helm.sh/test-result-success: <true/false>`
