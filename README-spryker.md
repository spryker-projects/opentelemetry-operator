# README-spryker

This README points to the process of building Target Allocator image for the Highly Available setup of OTEL Collector in ECS.

Current limitation is High Availability and Persistence cannot be enabled at once. For this to happen, this needs to be implemented as a further feature.

Community decided that they would like to expose filesystem based discovery of collectors in the Target Allocator, instead of ECS extention. This is not yet implemented by them. TA is a small application which doesn't have much opportunity for side effects.
When they introduce this functionality, shift the code from [aws_cloud_map.go](cmd/otel-allocator/collector/aws_cloud_map.go) and [collector.go](cmd/otel-allocator/collector/collector.go) to a separate binary the result list must be written in the filesystem by os.Write so the TA could read it. This binary will work in the same task definition with TA container and the small collector which only discovers instances. The results are passed via mount point containers. This is stateless and can run as many instances we need(more than 2 is an overkill).

## Prerequisites

* aws-cli - v2.x
* make - any modern version or system make will work
* docker-cli - any will work
* golang - go 1.22.0 (based on the go.mod file)

## Build

```
cd <base_repo_dir>

export TARGETALLOCATOR_VERSION=<version>
export AWS_ACCESS_KEY_ID="<key_id>"
export AWS_SECRET_ACCESS_KEY="<access_key>"
export AWS_SESSION_TOKEN="<token>"

make container-target-allocator TARGETALLOCATOR_IMG=target-allocator:${TARGETALLOCATOR_VERSION} && \
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/g5b2g3a8 && \ docker tag target-allocator:${TARGETALLOCATOR_VERSION} public.ecr.aws/g5b2g3a8/target-allocator:${TARGETALLOCATOR_VERSION} && \ docker push public.ecr.aws/g5b2g3a8/target-allocator:${TARGETALLOCATOR_VERSION}
```

## Rollout

Update the new version to the Target Allocator service under the [o11y_module](https://github.com/spryker-projects/tf-module-o11y/blob/b52ec0d3f864d0dd70e949290c3b8d43c603ad50/modules/o11y_target_allocator/locals.tf#L150).
