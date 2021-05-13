# kubevirt-realtime-doc
Manifests and documentation for running a RHEL8 VM under a customized Kubevirt version

1. Setting up your host environment
Tune your RHEL bare metal host for realtime kernel and pin your CPUs to run under realtime

2. Configuring OCP for realtime workloads
Select the target node where the realtime workload will run. Add a new worker lable using this command:

`oc label node worker-0-0 node-role.kubernetes.io/worker-rt=""`

This label is used to select the nodes that will be tuned by the Peformance Addon Operator to run realtime workloads. Deploy the PAO by running the following command:

`oc create -Rf manifests/01-pao`

3. Install CDI (Container Data Importer)
CDI is used to import the docker image of the customized RHEL as a data volume. Follow the instructions defined in the [CDI documentation page](https://kubevirt.io/user-guide/operations/containerized_data_importer/#install-cdi).

4. Install the Kubevirt manifests

`oc create -Rf manifests/02-kubevirt/`

5. Create the DataVolume and deploy the VM

`oc create -f manifests/03-vm/`
