[![Binder](https://mybinder.org/badge_logo.svg)](https://mybinder.org/v2/gh/SantanderMetGroup/binder-atlas/master)

This repo holds the binder for https://github.com/IPCC-WG1/Atlas

The docker image was built with:

```bash
GIT_REPO="https://github.com/SantanderMetGroup/binder-atlas"
GIT_REF=master
GIT_HASH=$(git ls-remote ${GIT_REPO} ${GIT_REF} | awk '{ print $1 }' | cut -c -7)
DOCKER_IMAGE_REF="santandermetgroup/ipcc-wgi-ar6-atlas-env:${GIT_HASH}"
repo2docker --no-run --subdir binder/conda --user-name jovyan --image ${DOCKER_IMAGE_REF} --ref ${GIT_REF} ${GIT_REPO}
docker push ${DOCKER_IMAGE_REF}
```
