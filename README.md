# ktp gatekeeper issue

The current setup is intended to validate all sub-pacakges in a mono repo with
a common set of policies.

## Issue

When running the top level pipeline containing gatekeeper as validator, no
error is flagged, even though the nested sub package [prod/app/](./prod/app/) 
does violate the non-root contraint.

```
$ kpt fn render
Package "kpt-gatekeeper-issue/prod/app": 
Package "kpt-gatekeeper-issue/stage/app": 
Package "kpt-gatekeeper-issue": 
[RUNNING] "gcr.io/kpt-fn/gatekeeper:v0.2.1"
[PASS] "gcr.io/kpt-fn/gatekeeper:v0.2.1" in 600ms

Successfully executed 1 function(s) in 3 package(s).
```

## Root Cause

It appears that the cause of the problem is that the same package is in both
environments. One time it has an contraint violation and one time it does not.

These two packages are merged together internally to some degree so that 
the error in the prod package is lost.

It works as expected if the stage folder is deleted.

```bash
$ rm -rf stage/
$ kpt fn render
Package "kpt-gatekeeper-issue/prod/app": 
Package "kpt-gatekeeper-issue": 
[RUNNING] "gcr.io/kpt-fn/gatekeeper:v0.2.1"
[FAIL] "gcr.io/kpt-fn/gatekeeper:v0.2.1" in 500ms
  Results:
    [error] apps/v1/Deployment/nginx-deploy: Containers must not run as root violatedConstraint: disallowroot
  Stderr:
    "[error] apps/v1/Deployment/nginx-deploy : Containers must not run as root"
    "violatedConstraint: disallowroot"
  Exit code: 1
```
