# k8s-shared-storage
This is a simple project that uses PV which is also known as **PersistentVolume** in Kubernates. and link it to the application to use one storage volume 
using PVC which is known as **PersistentVolumeClaim**

we will touch every part of the project in this file.

# Prerequisites
- [x] Linux OS
- [x] Minikube and Kubernates installed
- [x] free storage locally, ideally for the development, 5 GB 

# App Parts
## ConfigMap 

The **ConfigMap** file holds the declarations for the globally defined variables for **MariaDB** to be used later on.
We called this ConfigMap `mariadb-config` to be referenced in `app.yaml`
We used only two environment variables that were used in the MariaDB image on [DockerHub](https://hub.docker.com/_/mariadb).
So we used:
- MARIADB_DATABASE: for database name will be created once the pod is up and running. It is set to `test`
- MARIADB_USER: for the normal user to use the database for MariaDB. It is set to `user`

## Secret

The **Secret** file holds the declarations for the encoded variables for **MariaDB** like passwords and other sensitive data to be used later on in the application.
We called it `mariadb-secret` to be passed to the main application.
The data passed to the application were used in the official MariaDB image on [DockerHub](https://hub.docker.com/_/mariadb).
Data goes as follows:
- MARIADB_ROOT_PASSWORD: password for master database admin which is `root`
- MARIADB_PASSWORD: password used for normal uses like `user` we had earlier in `mariadb-config`

> [!NOTE]  
> Please don't push the Secret file to git. we just pushing this for demonstration purposes only.

## Service

The Service used in the application is `ClusterIP` which facilitates the communication between pods in the statefulset in Kubernetes.
it uses `TCP` protocol and Port 3306 which is the port MariaDB uses to operate. 

The name used for the service `mariadb-service`. Please note that we will use the name of the service for connecting to the MariaDB database.


## Storage

We used two volumes. The first one is `PersistentVolume` and called it `mariadb-storage`. This creates storage in the provided `path`
and has a `retain` policy which means the storage still exists when the pod is deleted.

we also used `PersistentVolumeClaim` as a linkage for the application to use the storage shared across all replicas in the system.


## App

the application is really simple. It uses a MariaDB image from Docker, and it has 3 replicas. 
The container has the name of `mariadb` and uses port 3306 the same one used for the service port. 
We also mount the storage in a specific path on pods `/var/lib/mariadb`. the application uses `mariadb-secret` and `mariadb-config` we created earlier.

for the most crucial part, the volume linkage which is done using the following block and is pretty much self-explanatory.

```
volumes:
- name: mariadb-storage
  persistentVolumeClaim:
    claimName: mariadb-pvc
```

# Running the Application

First of all, start **minikube** using `minikube start`. to visualize the Kubernetes resources, run the following command `minikube dashboard`

1- I used a new namespace to have all my resources which is called **test-ns** using the following command `kubectl create ns test-ns`.

2- By now we have the minikube running, to run the application make sure you have all the files in the current directory and run the following command `kubectl apply -f . -n test-ns`

3- This will create all resources on the namespace. you can connect to a pod using the dashboard we ran earlier. If you want to use kubectl, we can use `kubectl exec -it <pod-name> -n test-ns` replace <pod-name> with the name of the pod for example mariadb-0 or whatever the name you changed. you can always check pod names using `kubectl get pods -n test-ns`

4- after connecting to the pod, please run the following command `apt-get update && apt-get install mysql-server -y` This command will install mysql-server to connect to MariaDB database. To connect to the database use the following command `mysql -h mariadb-service -u root -p` and then you are prompted to type in a password. decode the password provided and type it in


5- Now you have the application running, and you can run SQL commands on one pod and can see the shared results when connecting to the database on the other pods.

> [!NOTE]  
> The root password is encoded base 64 and can be found in secret.yaml


# Conclustion 

We can infer that the usage of PV and linking it to the application using PVC is key to sharing a storage volume across the statefulset of the application. 

There are other ways to share the volume with the replicas like: NFS server that could be running on the local machine or on Cloud Provider Storage and pass it to the application.

In developing this, we thought that it might be conventional to run it locally with the help of PV and PVC to make it to the point with no further complications

# Appreciation

Thanks for following up and I hope you deployed your project as you wanted. In case of any inquiry, please contact me, you can find the contacts on my profile page here 😄 [Profile](https://github.com/abdelrahman-saad)
