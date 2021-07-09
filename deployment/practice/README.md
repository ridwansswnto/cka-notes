# This is all exercise, for practice CKA EXAM

## Workloads & Scheduling (15%)

### Understand deployments and how to perform rolling update and rollbacks

1. Create a deployment named nginx-deploy in the nd-cka namespace using nginx image version 1.19 with three replicas. Check that the deployment rolled out and show running pods

2. Scale the deployment to 5 replicas and check the status again. Then change the image tag of nginx container from 1.19 to 1.20.

3. Check the history of the deployment and rollback to previous revision. Then check that the nginx image was reverted to 1.19.
