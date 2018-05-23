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
oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/images.json -n openshift
```

once the images have been created (see original instructions on how to confirm), you can test the deployment

```
oc new-app s2i-minimal-notebook:3.5 --name my-notebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=mypassword
```

Go ahead and delete the deplyment 

```
oc delete all --lapp=my-notebook

```
