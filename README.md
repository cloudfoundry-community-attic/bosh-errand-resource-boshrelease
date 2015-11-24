BOSH Release for bosh-errand-resource
============================================

Run BOSH errands on Concourse!

Final releases are automatically created based on any changes to the upstream bosh-errand-resource

See the build pipeline https://ci.starkandwayne.com/pipelines/bosh-errand-resource-boshrelease for status.

Final releases are available on https://bosh.io/releases as well as this project's own [GitHub releases](https://github.com/cloudfoundry-community/bosh-errand-resource-boshrelease/releases).

Installation
------------

To use this bosh release, first upload it to the BOSH/bosh-lite that is running Concourse:

```
bosh upload release https://bosh.io/d/github.com/cloudfoundry-community/bosh-errand-resource-boshrelease
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

The final change is to explicitly list all the resource types (they are implicit) and add the `bosh-errand-resource` package to the list:

```yaml
jobs:
- name: worker
  ...
  properties:
    groundcrew:
      resource_types:
      - type: archive
        image: /var/vcap/packages/archive_resource
      - type: cf
        image: /var/vcap/packages/cf_resource
      - type: docker-image
        image: /var/vcap/packages/docker_image_resource
      - type: git
        image: /var/vcap/packages/git_resource
      - type: s3
        image: /var/vcap/packages/s3_resource
      - type: semver
        image: /var/vcap/packages/semver_resource
      - type: time
        image: /var/vcap/packages/time_resource
      - type: tracker
        image: /var/vcap/packages/tracker_resource
      - type: pool
        image: /var/vcap/packages/pool_resource
      - type: vagrant-cloud
        image: /var/vcap/packages/vagrant_cloud_resource
      - type: github-release
        image: /var/vcap/packages/github_release_resource
      - type: bosh-io-release
        image: /var/vcap/packages/bosh_io_release_resource
      - type: bosh-io-stemcell
        image: /var/vcap/packages/bosh_io_stemcell_resource
      - type: bosh-deployment
        image: /var/vcap/packages/bosh_deployment_resource
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
