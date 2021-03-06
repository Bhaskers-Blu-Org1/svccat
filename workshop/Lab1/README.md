# Lab 1: Deploy an application with in-memory storage

In this Lab we'll learn how to deploy an application hosted within the IBM Kubernetes Service that stores
state in memory.

# Deploy your application

In this part of the lab we will deploy an application called `guestbook`
that has already been built and uploaded to DockerHub under the name
`ibmcom/guestbook:v1`.

Start by running `guestbook`:

```
$ kubectl run guestbook --image=ibmcom/guestbook:v1
```

This action will take a bit of time. To check the status of the running application,
you can use `kubectl get pods`.

You should see output similar to the following:
```console
$ kubectl get pods
NAME                          READY     STATUS              RESTARTS   AGE
guestbook-59bd679fdc-bxdg7    0/1       ContainerCreating   0          1m
```
Eventually, the status should show up as `Running`.
```console
$ kubectl get pods
NAME                          READY     STATUS              RESTARTS   AGE
guestbook-59bd679fdc-bxdg7    1/1       Running             0          1m
```

The end result of the run command is not just the pod containing our application containers,
but a Deployment resource that manages the lifecycle of those pods and allows us the scale
the number of instances up and down as needed.
 
Once the status reads `Running`, we need to expose that deployment as a
service so we can access it through the IP of the worker nodes.
The `guestbook` application listens on port 3000.  Run:

```console
$ kubectl expose deployment guestbook --type="NodePort" --port=3000
service "guestbook" exposed
```

To find the port used on that worker node, examine your new service:
```console
$ kubectl get service guestbook
NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
guestbook   NodePort   10.10.10.253   <none>        3000:31208/TCP   1m
```

We can see that our `<nodeport>` is `31208`. We can see in the output the port mapping from 3000 inside 
the pod exposed to the cluster on port 31208. This port in the 31000 range is automatically chosen, 
and could be different for you.

`guestbook` is now running on your cluster, and exposed to the internet. We need to find out where it is accessible.
The worker nodes running in the container service get external IP addresses.
Run `bx cs workers <name-of-cluster>`, and note the public IP listed on the `<public-IP>` line.
```console
$ bx cs workers osscluster
OK
ID                                                 Public IP        Private IP     Machine Type   State    Status   Zone    Version  
kube-hou02-pa1e3ee39f549640aebea69a444f51fe55-w1   173.193.99.136   10.76.194.30   free           normal   Ready    hou02   1.5.6_1500*
```

We can see that our `<public-IP>` is `173.193.99.136`.
   
Now that you have both the address and the port, you can now access the application in the web browser
at `<public-IP>:<nodeport>`. In the example case this is `173.193.99.136:31208`.
   
You've now deployed the basic version of the application. If you've worked through the [kube101 workshop](https://github.com/IBM/kube101/tree/master/workshop)
this should look fairly familiar. We've just deployed Guestbook without an attached backing store, so it has defaulted to storing state in memory. You can verifiy
this by following the `/info` link on the front page, and it should display that the application is storing data in memory.

Test out the application by entering a few entries. Then, delete the Guestbook pod running in your cluster and see the data disappear:
```console
$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
guestbook-7b79469f5f-2tjhm   1/1       Running   0          14m
$ kubectl delete pod guestbook-7b79469f5f-2tjhm
pod "catalog-catalog-apiserver-8d85b8b48-k2nng" deleted
$ kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
guestbook-7b79469f5f-p9p8t   1/1       Running   0          11s
```

After the new pod is up and running, if we go back and refresh Guestbook in our browser, we can see that the entries 
we previously created are gone. This is obviously not ideal. We would like data
entered into our application to persist beyond the lifetime of individual containers.

In [the next lab of this course](../Lab2/README.md), we'll learn to manually provision
and connect some stable storage to persist our application data.
