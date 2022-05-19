A while ago my brother reached out to me asking about containerizing his application. Turns out he develops on his laptop which is an `x86` and had to deploy his application to a Raspberry Pi running `arm`. He was writing his application in Python, so he it worked on both architectures and hated copying files back and forth. I suggested building a pipeline leveraging GitHub Actions to build both architectures so he could just `docker pull` down the container on the different systems. This has a great side effect how too, so his cohorts just had to do the same to get either  and test on their hardware too!

This was a great representation of containers making your life easier, and leveraging the following GitHub Action it should work out of the box. 

Obviously you’ll need to change the registry targets, create the secrets, and fair warning it’s slow due to `qemu` but it does work. Hopefully this’ll help someone in the future :).


```yaml
 name: Docker Build
on:  
  push:    
    branches: [ main ]  
jobs:   

  build-multiarch-dockerfile:
    name: Build multi-architecture image 
    env:
      IMAGE_NAME: NAME_OF_IMAGE
      DOCKER_REGISTRY: quay.io 
      DOCKER_IMAGE: awesome/NAME_OF_IMAGE 
      DOCKER_TARGET_PLATFORM: linux/arm/v7                 
    runs-on: ubuntu-20.04
    strategy:
      fail-fast: false
      matrix:
        arch: [ amd64, arm64 ]

    steps:    
    - name: Checkout the code       
      uses: actions/checkout@v1          

    - name: Log in to Quay.io
      uses: redhat-actions/podman-login@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
        registry: ${{ env.DOCKER_REGISTRY }}

    - name: Install qemu dependency
      run: |
        sudo apt-get update
        sudo apt-get install -y qemu-user-static
    - name: Buildah Action
      uses: redhat-actions/buildah-build@v2
      with:
        image: ${{ env.DOCKER_IMAGE }}
        tags: latest
        arch: ${{ matrix.arch }}
        build-args: ARCH=${{ matrix.arch }}
        containerfiles: |
          ./Dockerfile
    - name: Push To quay.io
      id: push-to-quay
      uses: redhat-actions/push-to-registry@v2
      with:
        image: ${{ env.IMAGE_NAME }}
        tags: latest
        registry: quay.io/awesome
        

    - name: Print image url
      run: echo "Image pushed to ${{ steps.push-to-quay.outputs.registry-paths }}"
```
