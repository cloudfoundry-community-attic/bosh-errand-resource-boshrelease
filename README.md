BOSH Release for bosh-errand-resource
============================================

Run BOSH errands on Concourse!

Final releases are automatically created based on any changes to the upstream bosh-errand-resource

See the build pipeline https://ci.starkandwayne.com/pipelines/bosh-errand-resource-boshrelease for status.

Final releases are available on https://bosh.io/releases as well as this project's own [GitHub releases](https://github.com/starkandwayne/bosh-errand-resource-boshrelease/releases).

Installation
------------

To use this bosh release, first upload it to the BOSH/bosh-lite that is running Concourse:

```
bosh upload release https://bosh.io/d/github.com/starkandwayne/bosh-errand-resource-boshrelease
```

Next, update your Concourse deployment manifest to add the resource.

Add the `bosh-errand-resource` release to the list:

```yaml
releases:
  - name: concourse
    version: latest
  - name: garden-linux
    version: latest
  - name: bosh-errand-resource
    version: latest
```

Into the `worker` job, add the `{release: bosh-errand-resource, name: just_install_packages}` job template that will install the package:

```yaml
jobs:
- name: worker
  templates:
    ...
    - {release: bosh-errand-resource, name: just_install_packages}
```

The final change is to explicitly add the `bosh-errand-resource` package to the list:

```yaml
jobs:
- name: worker
  ...
  properties:
      additional_resource_types:
      - type: bosh-errand
        image: /var/vcap/packages/bosh-errand-resource
```

Note that it is the latter two lines that are specific to this BOSH release:

```yaml
- type: bosh-errand
  image: /var/vcap/packages/bosh-errand-resource
```

The former lines should be obtained from the Concourse BOSH release, not the documentation above which might be out of date. Use https://github.com/concourse/concourse/blob/master/jobs/groundcrew/spec#L69-L96

And `bosh deploy` your Concourse manifest.

Usage
-----

An example mini-pipeline that would run an errand:

```yaml
---
jobs:
- name: alert
  public: true
  plan:
  - put: errand-prod
    params:
      manifest: prod.yml
      errand: smoke_tests

resources:
- name: errand-prod
  type: bosh-errand
  source:
    target: {{bosh-target}}
    username: admin
    password: admin
    deployment: cf-prod
```

Setup pipeline in Concourse
---------------------------

```
fly -t snw set-pipeline -c pipeline.yml -l credentials.yml -p bosh-errand-resource-boshrelease
```
