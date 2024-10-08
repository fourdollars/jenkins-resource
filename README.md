 [![GitHub: fourdollars/jenkins-resource](https://img.shields.io/badge/GitHub-fourdollars%2Fjenkins%E2%80%90resource-darkgreen.svg)](https://github.com/fourdollars/jenkins-resource/) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![Bash](https://img.shields.io/badge/Language-Bash-red.svg)](https://www.gnu.org/software/bash/) ![Docker](https://github.com/fourdollars/jenkins-resource/workflows/Docker/badge.svg) [![Docker Pulls](https://img.shields.io/docker/pulls/fourdollars/jenkins-resource.svg)](https://hub.docker.com/r/fourdollars/jenkins-resource/)
# jenkins-resource
[Concourse CI](https://concourse-ci.org/)'s jenkins-resource to watch and trigger the Jenkins builds by https://www.jenkins.io/doc/book/using/remote-access-api/.

## Config 

### Resource Type

```yaml
resource_types:
- name: resource-jenkins
  type: registry-image
  source:
    repository: fourdollars/jenkins-resource
    tag: latest
```

or

```yaml
resource_types:
- name: resource-jenkins
  type: registry-image
  source:
    repository: ghcr.io/fourdollars/jenkins-resource
    tag: latest
```

### Resource

* host: **Required**
* user: **Required**
* token: **Required**
* job: **Required**, such as 'job/JOB_NAME'.
* port: Optional
* protocol: Optional, such as 'http' or 'https', using 'https' by default.
* timezone: Optional, such as 'Asia/Taipei'. 'UTC' by default.
* timeout: Optional, seconds to timeout. No timeout by default.
* builds: Optional, limit the build number to check. No limit by default.
* skip: Optional, don't check the result and the duration in check step or don't wait for the result in get step.
* crumb: Optional, using 'skip' if you want to skip the crumbIssuer.
* debug: Optional

```yaml
resources:
- name: test
  icon: bow-tie
  type: resource-jenkins
  check_every: 5m
  source:
    host: 127.0.0.1
    user: user
    token: wxdnqsclxzrmhb2k27frgjc7hdp3zqk0b4
    job: job/test
    port: 8080
    protocol: http
    builds: 5
- name: release
  icon: bow-tie
  type: resource-jenkins
  check_every: never
  source:
    host: 127.0.0.1
    user: user
    token: wxdnqsclxzrmhb2k27frgjc7hdp3zqk0b4
    job: job/release
    port: 8080
    protocol: http
    builds: 5
```

#### check step params

 * skip: Don't check the result and the duration to skip the builds without the result or the duration.
 * expectedResult: Optional, only check for the expected results.

#### get step params

 * expectedResult: Optional, waiting for the expected results.
 * timeout: Optional, seconds to timeout. No timeout by default.
 * skip: Don't wait for the result.

#### put step params

 * expectedResult: Optional, only generate the expected results.
 * stringParameters: Optional.
 * fileParameters: Optional.
 * timeout: Optional, seconds to timeout. No timeout by default.

### Job Example

```yaml
jobs:
- name: check-test-job
  plan:
  - get: test
    trigger: true
  - task: check
    config:
      platform: linux
      image_resource:
        type: registry-image
        source:
          repository: alpine
          tag: latest
      inputs:
        - name: test
      outputs:
        - name: outgoing
      run:
        path: sh
        args:
        - -exc
        - |
          apk add jq
          jq -r < test/api.json
          cp test/api.json outgoing/api.json
  - put: release
    inputs: [outgoing]
    params:
      stringParameters:
        id: 123
        verbosity: high 
      fileParameters:
        FILE_LOCATION_AS_SET_IN_JENKINS: outgoing/api.json
    get_params:
      expectedResult:
        - SUCCESS
        - UNSTABLE
        - FAILURE
        - NOT_BUILT
        - ABORTED
```
