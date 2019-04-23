# Test Command

## Overview

`helm test` gives a chart author the ability to define actions on a helm release which validate that the chart is running as intended. This is useful for ensuring that a `helm install` or `helm upgrade` created/modified Kubernetes resources successfully and that the consequential `helm release` is functional.
* A **test** is defined as a command and some arguments executed from a container running in a Kubernetes Pod. If the command completes inside the container with an exit code of 0, it is successful. All other exit codes indicate the test failed.
* Tests must be defined in the `templates/tests/` directory. This used to be recommended as a best practice. See motivations for change below:
  * Makes it much easier for a human to identify which templates are tests vs. which ones are manifests that define the application
  * There is currently no built-in way to create supporting resources to use for running a test. Having a test directory will allow us to more easily have a way to organize and manage resources directly related to testing.
  * This makes initial rendering and filtering resource manifests in the rendering engine much simpler to implement.
However, making this a requirement will make the logic for rendering resources much easier to
* Test annotations
  * Each test pod must have an annotation to denote the expected condition.
    * `helm.sh/test-expect-success: <true/false>`
      * `helm.sh/test-expect-success: true` is used to 1) denote that the pod the annotation is applied to is a test and not a resource to be created during `helm install` and 2) denote the expected outcome of the test.
    * In helm 2.x, tests used the annotation `helm.sh/hook: <test-success/test-fail>` to denote the expected condition. The test feature was implemented using the hooks system in helm 2 to add functionality in a backwards compatible way. We can move away from that now and make tests their own abstraction in helm v3 instead of re-using lifecycle hooks. This will make the test functionality more robust and will set a foundation for us to make the testing functionality even more powerful with possibily its own storage system for saving and fetching test results. It can also lay the foundation to allow us to more cleanly run tests as part of lifecycle hooks (like as a `post-install` hook).
* `helm test` sub-commands
    * `helm test run <release>`
        * Executes tests by creating the Kubernetes Pods declared with the annotation `helm.sh/test-expect-success` annotation in the chart and all of its subcharts in the `charts/` directory and waits for test results
        * Adds appropriate [labels and annotations](#labels-and-annotations) to test Pods
    * `helm test cleanup <release>`
        * Deletes test Pods created in the last test run
    * `helm test result <release>`
        * Outputs results of last test run of a release
        * _Note: This not required as part of the initial helm 3.0.0 release`_

## Labels and Annotations
Helm will add the following labels and annotations to each test Pod when the tests are executed. These labels and annotations will be used by `helm test result <release>` and `helm test cleanup <release>` to display the results from the test run and clean up test pods.
* Helm will add the following labels to each test pod when the test is run:
  * `helm.sh/release: <release-name>`
  * `helm.sh/release-version: <release-version>`
  * `helm.sh/tested: <timestamp> `
* Helm will add the following annotations to each test pod when the test is run:
  * `helm.sh/test-result-success: <true/false>`

## Outstanding questions/comments
* Do we need `helm test results` ? and what should it store exactly?
  * Consider taking `helm test result` out completely
* Where do we want to store the output of tests? Within the release object or a separate object?
* Look at using Jobs instead of Pods or both (Jacob)
* Would be usefult to turn off and on subchart/dependency tests (Jacob)
* More structured test output: table, JSON
* Code coverage
* Resource-policy for after-success auto cleanup
