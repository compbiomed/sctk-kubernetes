# sctk-kubernetes

Go to [Google Cloud Console](https://console.cloud.google.com/) and create a new project. Make sure billing is enabled for the project. In the navigation bar, search for "Kubernetes Engine". On the Clusters page, click "Create cluster".

Most of the settings here can be left to the default. I typically chose a better name (sctk-cluster), and perhaps change the Nodes under Node Pools (default-pool). Each node will support a certain amount of pods (or SCTK instances), and when you reach a certain memory or CPU threshold, a new node will be created. You could have 1 large expensive node that supports many pods if you envision supporting lots of concurrent users. Or rather a smaller node that needs to scale more often if you only have a few users. Each user receives their own pod. In this case I go with "Number of nodes" = 3 and Autoscaling on. Finally click Create at the bottom of the page.

Clone the [SCTK repo](git@github.com:compbiomed/singleCellTK.git) and checkout the devel branch. Make sure you have docker installed. Make note of the SCTK project ID on GCP, as well as the version number of the library. The project ID can be found on the [dashboard page](https://console.cloud.google.com/home), or in the pop-up when you view the list of projects at the top of the page. Build a docker container with that information.

```$ docker build -t gcr.io/sctk-xxxx/singlecelltk:vx.x.x```

Depending on your environment, this might take a few hours.
Now clone the [SCTK Kubernetes](https://github.com/hicsail/sctk-kubernetes) repo. Inside of the `shinyproxy` folder open `application.yml` and adjust the authentication settings. Build the Docker container for ShinyProxy as follows:

```$ docker build -t gcr.io/sctk-xxxx/shinyproxy:vx.x.x```

Make sure the ID matches the Google Cloud project ID, and the version number is that of ShinyProxy (look at the Dockerfile to see which version of ShinyProxy is being downloaded). Do the same thing for `kube-proxy-sidecar`:

```$ docker build -t gcr.io/sctk-xxxx/kube-proxy-sidecar:v0.1.0```

In `kube-config/sp-deployment.yaml`, edit the image names to make sure they contain the correct project IDs.

Install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/install) and [initialize it](https://cloud.google.com/sdk/docs/initializing). Next install the `kubectl` command-line tool, and configure Docker to push images to Google Cloud:

```
$ gcloud components install kubectl
$ gcloud auth configure-docker
```

For ease of use, set your project as the default one, replacing `project-id` with your project ID:

```$ gcloud config set project project-id```

Now configure `kubectl` with the credentials of your cluster, replacing `cluster-name` with the name of your cluster (sctk-cluster in this example) and `project-id` with your project ID:

```$ gcloud container clusters get-credentials cluster-name --zone us-central1-c --project project-id```

Push all Docker containers to Google Cloud:

```
$ docker push gcr.io/sctk-xxxx/shinyproxy:vx.x.x
$ docker push gcr.io/sctk-xxxx/kube-proxy-sidecar:vx.x.x
$ docker push gcr.io/sctk-xxxx/singlecelltk:vx.x.x
```

Finally open the `kube-config` folder and create the Kubernetes configurations that will set up a load balancer, ties the Docker images together, and direct traffic where needed.

```
$ kubectl create -f sp-authorization.yaml
$ kubectl create -f sp-service.yaml
$ kubectl create -f sp-deployment.yaml
```

In order to connect to ShinyProxy, get the IP address as follows. You can also find it on the Google Cloud Services & Ingress tab, under `Endpoints`.

```$ kubectl get service shinyproxy```
