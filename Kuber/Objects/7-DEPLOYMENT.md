Update your app with zero-downtime update process.

### Updating applications running in pods
A normal service looks like 
![[Pasted image 20240502185509.png]]

When we create new version of application and push it to docker repository, how can we update the pod version?

We can do that with two ways:
- Delete all existing pods first and then start the new ones.
- Start new ones and, once they’re up, delete the old ones. You can do this either by adding all the new pods and then deleting all the old ones at once, or sequentially, by adding new pods and removing old ones gradually.

#### Deleting old pods and replacing them with new ones
If you have a ReplicationController managing a set of v1 pods, you can easily replace them by modifying the pod template so it refers to version v2 of the image and then deleting the old pod instances. The ReplicationController will notice that no pods match its label selector and it will spin up new instances.

![[Pasted image 20240502190402.png]]

>You should accept the short downtime between the time the old pods are deleted and new ones are started.

