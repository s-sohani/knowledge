You want your deployments to stay up and running automatically and remain healthy without any manual intervention. To do this, you almost never create pods directly. Instead, you create other types of resources, such as ReplicationControllers or Deployments.
Kubernetes checks if a container is still alive and restarts it if it isn’t.

When Kubernetes restart the container?
- If application has a bug that causes it to crash every once in a while.
- Stop working without their process crashing like memory leak. 
- Check an application’s health from the outside.

