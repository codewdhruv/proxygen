Build & Test Proxygen on Harness CI
=======================================
This is a fork of Proxygen project by Facebook. This project is used for testing Harness CI's capabilities by the CME team at Harness. Please follow [these instructions](https://github.com/harness-community/graphql-js/blob/trunk/.harness/README.md) on how to clone this repo and see the results yourself.


- [Harness Fast CI Blog Announcement](https://harness.io/blog/announcing-speed-enhancements-and-hosted-builds-for-harness-ci)
- [Get Started with Harness CI](https://harness.io/products/continuous-integration)

## Setting up this pipeline on Harness CI Hosted Builds

1. Create a [GitHub Account](https://github.com) or use an existing one

2. Fork [this repository](https://github.com/harness-community/graphql/graphql-js) into your GitHub account. 

3. If you are new to Harness CI, signup for [Harness CI](https://app.harness.io/auth/#/signup)
  * Select the `Continuous Integration` module and choose the `Starter pipeline` wizard to create your first pipeline using the forked repo from #2.
  * Go to the newly created pipeline and hit the `Triggers`tab. If everything went well, you should see two triggers auto-created. A `Pull Request`trigger and a `Push`trigger. For this exercise, we only need `Pull Request`trigger to be enabled. So, please disable or delete the `Push`trigger.

4. If you are an existing Harness CI user, create a new pipeline to use the cloud option for infrastructure and setup the PR trigger.

5. Add the pipeline.yaml stages in the YAML editor:

```
  tags: {}
  properties:
    ci:
      codebase:
        connectorRef: dhrubaaccountconnector
        repoName: proxygen
        build: <+input>
  stages:
    - stage:
        name: Build
        identifier: BuildKernel
        type: CI
        spec:
          cloneCodebase: true
          execution:
            steps:
              - step:
                  type: Run
                  name: Fetch Deps
                  identifier: Fetch_Deps
                  spec:
                    shell: Sh
                    command: python3 build/fbcode_builder/getdeps.py fetch --no-tests <+matrix.deps>
                  failureStrategies: []
                  strategy:
                    matrix:
                      deps:
                        - ninja
                        - cmake
                        - zlib
                        - zstd
                        - boost
                        - double-conversion
                        - fmt
                        - gflags
                        - glog
                        - googletest
                        - libevent
                        - lz4
                        - snappy
                        - autoconf
                        - automake
                        - libtool
                        - gperf
                        - libsodium
                        - xz
                        - folly
                        - fizz
                        - mvfst
                        - wangle
                      maxConcurrency: 5
              - step:
                  type: Run
                  name: Build Deps
                  identifier: Build_Deps
                  spec:
                    shell: Sh
                    command: python3 build/fbcode_builder/getdeps.py build --no-tests <+matrix.deps>
                  failureStrategies: []
                  strategy:
                    matrix:
                      deps:
                        - ninja
                        - cmake
                        - zlib
                        - zstd
                        - boost
                        - double-conversion
                        - fmt
                        - gflags
                        - glog
                        - googletest
                        - libevent
                        - lz4
                        - snappy
                        - autoconf
                        - automake
                        - libtool
                        - gperf
                        - libsodium
                        - xz
                        - folly
                        - fizz
                        - mvfst
                        - wangle
                      maxConcurrency: 1
              - step:
                  type: Run
                  name: Build Proxygen
                  identifier: Build_Proxygen
                  spec:
                    shell: Sh
                    command: python3 build/fbcode_builder/getdeps.py build --src-dir=. proxygen  --project-install-prefix proxygen:/usr/local
              - step:
                  type: Run
                  name: Copy artifacts
                  identifier: Copy_artifacts
                  spec:
                    shell: Sh
                    command: python3 build/fbcode_builder/getdeps.py fixup-dyn-deps --strip --src-dir=. proxygen _artifacts/linux  --project-install-prefix proxygen:/usr/local --final-install-prefix /usr/local
              - step:
                  type: Run
                  name: Test Proxygen
                  identifier: Test_Proxygen
                  spec:
                    shell: Sh
                    command: python3 build/fbcode_builder/getdeps.py test --src-dir=. proxygen  --project-install-prefix proxygen:/usr/local
                    reports:
                      type: JUnit
                      spec:
                        paths:
                          - "**/*.xml"
          platform:
            os: Linux
            arch: Amd64
          runtime:
            type: Cloud
            spec: {}

```
> Make sure you modify the connectors with the connectors you create. 

6. Create a Pull Request in a new branch by updating the project. (e.g. add a comment or new line). This should invoke a build in Harness CI

7. Merge the PR after the pipeline execution is successfull.

8. Enable GitHub Actions : The repository forked in Step 2 already has a GitHub Actions workflow file added. You can choose to enable this workflow from the Actions tab on GitHub.

9. Create any other Pull Request with a few source or test file changes. You can consider cherry-picking any of the commits from the main repository.

10. This PR will trigger the Harness CI pipeline (as well as GitHub Actions workflow if enabled in Step-9). Here we are implementing a single workflow for Github Actions. (Get_Deps_Mac)
