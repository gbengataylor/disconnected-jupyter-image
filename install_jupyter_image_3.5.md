
Instructions for installing images in docker registry and deploying in openshift
================================================================================
These images were  created using instructions in https://github.com/jupyter-on-openshift/jupyter-notebooks
The readme for this git repo can be found in original_readme.md. Please read original_readme.md for a description about the images. 

The instructions below were created for jupyter notebook version 3.5 images. The local registry path it was created from was 172.30.1.1:5000/openshift

Transfer and unzip files
------------------------

```
gzip -d s2i-minimal-notebook.tar.gz
gzip -d s2i-tensorflow-notebook.tar.gz
gzip -d s2i-scipy-notebook.tar.gz
```

Load into docker registry
-------------------------
Prepare the environment variables

```
#for these particular set of images, leave the next three environment variables as-is
export LOCAL_DOCKER_REGISTRY=172.30.1.1:5000
export LOCAL_IMAGE_PROJECT=openshift
export JUPYTER_VERSION=3.5

#modify the following variables as needed

#password to use when deploying notebook
export JUPYTER_PASSWORD=mypassword

# your openshift master
export OPENSHIFT_CONSOLE = https://18.188.76.211:8443 

# docker registry
export DOCKER_REGISTRY = docker-registry-default.apps.18.217.182.217.nip.io/ 

#project where images are to be installed
export IMAGE_PROJECT = openshift 

#openshift user with access to the registry and the target project
export OPENSHIFT_USER = system 
```

Login to openshift console and docker registry
```
oc login $OPENSHIFT_CONSOLE -u $OPENSHIFT_USER
docker login -p $(oc whoami -t) -u unused $DOCKER_REGISTRY
```

Load images into registry
```
docker load --input s2i-minimal-notebook.tar
docker load --input s2i-tensorflow-notebook.tar
docker load --input s2i-scipy-notebook.tar
```

Verify that the images have been added
```
docker images | grep $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-
```

Tag the images
```
docker tag $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-minimal-notebook:$JUPYTER_VERSION $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-minimal-notebook:$JUPYTER_VERSION

docker tag $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-tensorflow-notebook:$JUPYTER_VERSION $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-tensorflow-notebook:$JUPYTER_VERSION

docker tag $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-scipy-notebook:$JUPYTER_VERSION $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-scipy-notebook:$JUPYTER_VERSION
```

Verify that the images have been added
```
docker images | grep $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-
```

Push the images to the registry
```
docker push $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-minimal-notebook:$JUPYTER_VERSION

docker push $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-tensorflow-notebook:$JUPYTER_VERSION

docker push $DOCKER_REGISTRY/$IMAGE_PROJECT/s2i-scipy-notebook:$JUPYTER_VERSION
```


Deploy and test in OpenShift
----------------------------
Test the s2i-minimal-notebook by deploying in the new cluster
```
oc new-project jupyter
oc new-app s2i-minimal-notebook:$JUPYTER_VERSION --name my-notebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=$JUPYTER_PASSWORD

oc create route edge my-notebook --service my-notebook \
    --insecure-policy Redirect
```

Access the created route and test away!!
