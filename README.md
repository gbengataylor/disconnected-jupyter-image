Instructions to save jupyter images locally then reload them in another registry and openshift environment (disconnected operations)
=================================================================================================

This repository contains steps to 
* build jupyter images using instructions found https://github.com/jupyter-on-openshift/jupyter-notebooks
* slight change made in https://github.com/jupyter-on-openshift/jupyter-notebooks to update base image to RHEL python
* create tar files from these images
* recreate the images from the tar files locally
* manually export the images to a different docker repository and openshift cluster
* load the images to the new docker repository
* deploy instances of the jupyter notebook in the new OpenShift Cluster


Build Jupyter images
-----------------------------
see original_readme.md. This instructions assume the images are on version 3.5. If the jupyter-on-openshift instructions have been updated with a new version, update the version.

In these examples all images are added to the openshift namespace. Ensure that your user has permission to write to this project. If not, substitute "openshift" namespace for a project you have access to


Prepare the environment variables
```
#modify as needed
export LOCAL_DOCKER_REGISTRY=172.30.1.1:5000
export LOCAL_IMAGE_PROJECT=openshift
export JUPYTER_VERSION=3.5
export JUPYTER_PASSWORD=mypassword
```

Create the images
```
#oc create -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/images.json -n $LOCAL_IMAGE_PROJECT
#use RHEL base layer instead
oc create -f https://raw.githubusercontent.com/gbengataylor/jupyter-notebooks/master/images.json  -n $LOCAL_IMAGE_PROJECT
```

once the images have been created (see original instructions on how to confirm, a tagged image ``s2i-minimal-notebook:$JUPYTER_VERSION`` should be created in the openshift project or the project you ran the command in), you can test the deployment

```
oc new-app s2i-minimal-notebook:$JUPYTER_VERSION --name my-notebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=$JUPYTER_PASSWORD
```

Go ahead and delete the deplyoment 

```
oc delete all -lapp=my-notebook
```



Save the images to tar files

```
docker save $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-minimal-notebook:$JUPYTER_VERSION > s2i-minimal-notebook.tar
docker save $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-tensorflow-notebook:$JUPYTER_VERSION > s2i-tensorflow-notebook.tar
docker save $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-scipy-notebook:$JUPYTER_VERSION > s2i-scipy-notebook.tar
```

Example
```
docker save 172.30.1.1:5000/openshift/s2i-minimal-notebook:$JUPYTER_VERSION > s2i-minimal-notebook.tar
docker save 172.30.1.1:5000/openshift/s2i-tensorflow-notebook:$JUPYTER_VERSION > s2i-tensorflow-notebook.tar
docker save 172.30.1.1:5000/openshift/s2i-scipy-notebook:$JUPYTER_VERSION > s2i-scipy-notebook.tar
```

Now let's delete the images
```
#oc delete -f https://raw.githubusercontent.com/jupyter-on-openshift/jupyter-notebooks/master/images.json -n $LOCAL_IMAGE_PROJECT

# use our version
oc delete -f https://raw.githubusercontent.com/gbengataylor/jupyter-notebooks/master/images.json  -n $LOCAL_IMAGE_PROJECT

docker rmi $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-tensorflow-notebook:$JUPYTER_VERSION
docker rmi $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-minimal-notebook:$JUPYTER_VERSION
docker rmi $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-scipy-notebook:$JUPYTER_VERSION
```

Verify they have been deleted. This command should return no results

```
docker images | grep $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-
```


Let's perform a quick test of loading the images from the tar files
```
docker load --input s2i-minimal-notebook.tar
docker load --input s2i-tensorflow-notebook.tar
docker load --input s2i-scipy-notebook.tar
```

Verify that they have been loaded. The following statement should return results
```
docker images | grep $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-
```

Login into registry and push the images to the docker registry. Ensure that you are logged into openshift (Note: the push may be unnecessary)
```
docker login -u openshift -p $(oc whoami -t) $LOCAL_DOCKER_REGISTRY

docker push $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-minimal-notebook:$JUPYTER_VERSION
docker push $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-tensorflow-notebook:$JUPYTER_VERSION
docker push $LOCAL_DOCKER_REGISTRY/$LOCAL_IMAGE_PROJECT/s2i-scipy-notebook:$JUPYTER_VERSION
```

Test the images by deploying
```
oc new-project jupyter
oc new-app s2i-minimal-notebook:$JUPYTER_VERSION --name my-notebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=$JUPYTER_PASSWORD

oc create route edge my-notebook --service my-notebook \
    --insecure-policy Redirect
```
confirm that it is working by accessing the notebook via the route

login with $JUPYTER_PASSWORD  

create a new python workspace

enter the following in the code section

```
for i in range(500):
    print(2**i - 1)
```

and execute

Clean up
```
oc delete all -lapp=my-notebook
```

Zip files
```
gzip s2i-minimal-notebook.tar
gzip s2i-tensorflow-notebook.tar
gzip s2i-scipy-notebook.tar
```


Loading into new registry/openshift cluster
-------------------------------------------
Transfer the zip files and unzip

```
gzip -d s2i-minimal-notebook.tar.gz
gzip -d s2i-tensorflow-notebook.tar.gz
gzip -d s2i-scipy-notebook.tar.gz
```

Prepare environment variables
```
# modify as needed 
export OPENSHIFT_CONSOLE = https://18.188.76.211:8443
export DOCKER_REGISTRY = docker-registry-default.apps.18.217.182.217.nip.io/
export IMAGE_PROJECT = openshift
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

Test the images by deploying in the new cluster
```
oc new-project jupyter
oc new-app s2i-minimal-notebook:$JUPYTER_VERSION --name my-notebook \
    --env JUPYTER_NOTEBOOK_PASSWORD=$JUPYTER_PASSWORD

oc create route edge my-notebook --service my-notebook \
    --insecure-policy Redirect
```
confirm that it is working by accessing the notebook via the created route

login with $JUPYTER_PASSWORD (set above during deploy)

create a new python workspace

enter the following in the code section

```
for i in range(500):
    print(2**i - 1)
```

and execute

Clean up
```
oc delete all -lapp=my-notebook
```



