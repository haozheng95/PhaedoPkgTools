# PhaedoPkgTools

## Introduced
Provides a way to make the image of initContainer in the [bitfusion-with-kubernetes-integration] project
## Prerequisites
- python3 + pip3
- docker
- Ensure that the host can connect to the Internet

## Usage
Start by installing the dependencies used by the project
```bash
pip3 install -r requirement.txt
```
Go to Bitfusion's website to download the Bitfusion client version that needs to be packaged into the image. Here provides Bitfusion installation tutorial at https://docs.vmware.com/en/VMware-vSphere-Bitfusion/index.html

Here, we select Bitfusion Client 4.0.1-5 Ubuntu 18, Ubuntu 20, Centos 7, and Centos8 OS versions as examples

1.Download the installation package to the local host
```
├── bitfusion-client-centos7-4.0.1-5.x86_64.rpm
├── bitfusion-client-centos8-4.0.1-5.x86_64.rpm
├── bitfusion-client-ubuntu1804_4.0.1-5_amd64.deb
├── bitfusion-client-ubuntu2004_4.0.1-5_amd64.deb

```

2.Modify the build.yaml file
This file is passed in as a parameter needed to run the program. Different keys mean different things
- imagename Indicates the name of the generated image
- OSVersion Indicates the operating system version of the corresponding installation package
- BitfusionVersion Indicates the version of the Bitfusion Client. The value is an integer to comply with the rules in the [bitfusion-with-kubernetes-integration] project. The value can be a major version X 100, for example, 300 is used to generate 3.xx
- buildimage Specifies the system version of the Bitfusion Client that needs to be installed. The value can be found in the downloaded installation file, for example, bitfusion-client-ubuntu18044.0.1-5_amd64.deb ubuntu1804 is the corresponding build
- local On behalf of the location of the installation package. The relative path or absolute path can be used
```yaml

imagename: bitfusion-client:example
data:
  - OSVersion: ubuntu18
    BitfusionVersion: 400
    buildimage: ubuntu:18.04
    local: bitfusion-client-ubuntu1804_4.0.1-5_amd64.deb
  - OSVersion: ubuntu20
    BitfusionVersion: 400
    buildimage: ubuntu:20.04
    local: bitfusion-client-ubuntu2004_4.0.1-5_amd64.deb
  - OSVersion: centos7
    BitfusionVersion: 400
    buildimage: centos:7
    local: bitfusion-client-centos7-4.0.1-5.x86_64.rpm
  - OSVersion: centos8
    BitfusionVersion: 400
    buildimage: centos:8
    local: bitfusion-client-centos8-4.0.1-5.x86_64.rpm
```
3. Run the following command to run the project
```bash
python3 build.py
```
Two files are generated before the image is built
- Dcoekrfile Verifies the Dockerfile to view the steps of the image construction
- bitfusion-client-configmap.yaml Check this file to determine the final location of the Bitfusion Client installation package in the image. Do not modify this file
## Output content
The output includes
- image Indicates the image of bitfusion-client:example. The name is user-defined.
- Dockerfile is used to build images
- bitfusion-client-configmap.yaml needs to replace the corresponding files in the bitfusion-with-kubernetes-integration project. The bitfusion-client-configmap.yaml file records the location information of clients of different versions in image

## Used in the Device Plugin

Assuming there is only one K8S node

You need to replace the name of the currently output image with the name of the original image. For example the current [bitfusion-with-kubernetes-integration] the initContainer name for project use bitfusiondeviceplugin/bitfusion-client:0.2.2, The name of the image we output is bitfusion-client:example. We need to run the following command to replace the original image with our image
```bash
docker rmi bitfusiondeviceplugin/bitfusion-client:0.2.2
docker tag bitfusion-client:example bitfusiondeviceplugin/bitfusion-client:0.2.2
```
Then replace the [bitfusion-client-configmap.yaml] file under [bitfusion-with-kubernetes-integration/bitfusion_device_plugin/webhook/deployment/bitfusion-client-configmap.yaml] in the [bitfusion-with-kubernetes-integration] project with the [bitfusion-client-configmap.yaml] file we output
After the above two steps are complete, run the following command to redeploy the [bitfusion-with-kubernetes-integration] project. Run in [bitfusion-with-kubernetes-integration/bitfusion_device_plugin/]
```bash
make deploy
```
At this point, our single-node replacement is completed. If there are multiple K8S nodes, the operation of image replacement needs to be repeated on each node, which can be deployed and replaced by referring to installation Method 2 in readME of Bitfusion-with-Kubernetes-Integration project
