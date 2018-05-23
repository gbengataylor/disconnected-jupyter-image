Instructions to save jupyter images to zip file then reload them in registry
=============================================================================

This repository contains steps to 
* build jupyter images using instructions found https://github.com/jupyter-on-openshift/jupyter-notebooks
* create tar files from these images
* recreate the images from the tar files locally
* manually export the images to a different docker repository and openshift cluster
* load the images to the new docker repository
* deploy instances of the jupyter notebook in the new OpenShift Cluster


Build Jupyter images
-----------------------------
see original_readme.md. This instructions assume the images are on version 3.5. If the jupyter-on-openshift instructions have been updated with a new version, update the version.

In these examples all images are added to the openshift namespace. Ensure that your user has permission to write to this project. If not, substitute "openshift" namespace for a project you have access to


```
set $LOCAL_IMAGE_PROJECT=openshit # change appropiatedly
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/images.json -n $LOCAL_IMAGE_PROJECT
```

once the images have been created (see original instructions on how to confirm, a tagged image ``s2i-minimal-notebook:3.5`` should be created in the openshift project or the project you ran the command in), you can test the deployment

```
oc new-app s2i-minimal-notebook:3.5 --name my-notebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=mypassword
```

Go ahead and delete the deplyment 

```
oc delete all --lapp=my-notebook

```

Save the images to tar files

```
docker save $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-minimal-notebook:3.5 > s2i-minimal-notebook.tar
docker save $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-tensorflow-notebook:3.5 > s2i-tensorflow-notebook.tar
docker save $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-scipy-notebook:3.5 > s2i-scipy-notebook.tar
```

Example if 
```
export LOCAL_DOCKER_REGISTRY=172.30.1.1:5000
export LOCAL_IMAGE_PROJECT=openshift
```

```
docker save 172.30.1.1:5000/openshift/s2i-minimal-notebook:3.5 > s2i-minimal-notebook.tar
docker save 172.30.1.1:5000/openshift/s2i-tensorflow-notebook:3.5 > s2i-tensorflow-notebook.tar
docker save 172.30.1.1:5000/openshift/s2i-scipy-notebook:3.5 > s2i-scipy-notebook.tar
```

Now let's delete the images
```
oc delete -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/images.json -n $LOCAL_IMAGE_PROJECT
docker rmi $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-tensorflow-notebook:3.5
docker rmi $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-minimal-notebook:3.5
docker rmi $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-scipy-notebook:3.5
```


