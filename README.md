# kubevirt-realtime-doc
Manifests and documentation for running a RHEL8 VM under a customized Kubevirt version

1. Setting up your host environment

Tune your RHEL bare metal host for realtime kernel and pin your CPUs to run under realtime. You can follow the steps described in the [RHEL documentation](https://access.redhat.com/rhel-real-time-getting-started)

2. Configuring OCP for realtime workloads

Select the target node where the realtime workload will run. Add a new worker lable using this command:

```bash
$>oc label node worker-0-0 node-role.kubernetes.io/worker-rt=""
```

This label is used to select the nodes that will be tuned by the Peformance Addon Operator to run realtime workloads. Next, deploy the PAO by running the following command:

```bash
$>oc create -Rf manifests/01-performance_addon_operator/
```

Wait until the changes have been applied to the target node by checking that the machine config pool for the new profile has `True` in the `UPDATED` state, like shown below:

```bash
$>oc get mcp worker-rt
NAME        CONFIG   UPDATED   UPDATING   DEGRADED   MACHINECOUNT   READYMACHINECOUNT   UPDATEDMACHINECOUNT   DEGRADEDMACHINECOUNT   AGE
worker-rt            True      False      False      1              0                   0                     0                      3m15s
```

3. Install the customized Kubevirt manifests

Deploy the customized Kubevirt by executing the following command:

```bash
$>oc create -Rf manifests/02-kubevirt/
```

Wait until all the pods in the `kubevirt` namespace are in `Running` state:

```bash
$>oc get pod -n kubevirt
NAME                               READY   STATUS    RESTARTS   AGE
virt-api-6779cd486c-jh5lb          1/1     Running   0          119s
virt-api-6779cd486c-v2pkh          1/1     Running   0          119s
virt-controller-56dc859785-9gwwb   1/1     Running   0          89s
virt-controller-56dc859785-zzckc   1/1     Running   0          89s
virt-handler-lwzdn                 1/1     Running   0          89s
virt-handler-mbp9w                 1/1     Running   0          89s
virt-operator-788b865d68-4zrg8     1/1     Running   0          2m48s
virt-operator-788b865d68-sjztd     1/1     Running   0          2m48s
```

4. Create the `poc` namespace and deploy the VM manifest 

```bash
$>oc create -Rf manifests/03-vm
```

This will create the VM, VMI and the pod in the `poc` namespace.

```bash
$>oc get pod -n poc -o wide
NAME                              READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
virt-launcher-vm-realtime-wscgv   1/1     Running   0          41s     10.131.1.41   worker-0-0   <none>           <none>
```

To access the console of the VM, use the `virtctl` [client tool](https://kubevirt.io/user-guide/operations/virtctl_client_tool/):
```bash
$>virtctl console vm-realtime
Successfully connected to vm-realtime console. The escape sequence is ^]

Red Hat Enterprise Linux 8.3 (Ootpa)
Kernel 4.18.0-193.51.1.rt13.101.el8_2.x86_64 on an x86_64

Activate the web console with: systemctl enable --now cockpit.socket

vm-realtime login:
```
