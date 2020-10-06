# sctk-kubernetes

Go to [Google Cloud Console](https://console.cloud.google.com/) and create a new project. Make sure billing is enabled for the project. In the navigation bar, search for "Kubernetes Engine". On the Clusters page, click "Create cluster".

Most of the settings here can be left to the default. I typically chose a better name (sctk-cluster), and perhaps change the Nodes under Node Pools (default-pool). Each node will support a certain amount of pods (or SCTK instances), and when you reach a certain memory or CPU threshold, a new node will be created. You could have 1 large expensive node that supports many pods if you envision supporting lots of concurrent users. Or rather a smaller node that needs to scale more often if you only have a few users. Each user receives their own pod. In this case I go with "Number of nodes" = 3 and Autoscaling on. Finally click Create at the bottom of the page.

Clone the [SCTK repo](git@github.com:compbiomed/singleCellTK.git) and checkout the devel branch. Make sure you have docker installed. Make note of the SCTK project ID on GCP, as well as the version number of the library. The project ID can be found on the [dashboard page](https://console.cloud.google.com/home), or in the pop-up when you view the list of projects at the top of the page. Build a docker container with that information.

```$ docker build -t gcr.io/sctk-xxxx/sctk:vx.x.x```

Depending on your environment, this might take a few hours.