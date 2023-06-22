[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/SantanderMetGroup/binder-atlas/climate4r-v2.5.4)

This repo holds the binder execution environment for https://github.com/IPCC-WG1/Atlas

The docker image was built with:
```
GIT_REPO="https://github.com/SantanderMetGroup/binder-atlas"
GIT_REF=climate4r-v2.5.4
GIT_HASH=$(git ls-remote ${GIT_REPO} ${GIT_REF} | awk '{ print $1 }' | cut -c -7)
DOCKER_IMAGE_REPO="santandermetgroup/binder-atlas"
DOCKER_IMAGE_REF="${DOCKER_IMAGE_REPO}:${GIT_HASH}"
echo "BUILDING DOCKER IMAGE: " ${DOCKER_IMAGE_REF}
repo2docker --no-run --subdir binder/conda --user-name jovyan --user-id 1000 --image-name ${DOCKER_IMAGE_REF} --ref ${GIT_REF} ${GIT_REPO}
docker tag ${DOCKER_IMAGE_REF} "${DOCKER_IMAGE_REPO}:${GIT_REF}"
#docker login
docker push ${DOCKER_IMAGE_REF}
docker push "${DOCKER_IMAGE_REPO}:${GIT_REF}"
```
