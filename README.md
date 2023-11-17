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

# Appreciation

Thanks for following up and I hope you deployed your project as you wanted. In case of any inquiry, please contact me, you can find the contacts on my profile page here 😄 [Profile](https://github.com/abdelrahman-saad)
