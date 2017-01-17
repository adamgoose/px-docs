# Volume Scale

Orchestration software such as kubernetes, mesos, etc.  allow scaling the
number of intances of the pod/application.  When these containers are
associated with volumes, there is no way to link the containers to a volume.

Volume scale allows re-use of the same volume name for all container instances
by performing on-demand creation of volumes as the user containers get
scheduled on a node.

In runtime, a node may fail and applications get respawned. With volume 
scale, applications are re-associated with volumes regardless of where they get
spun up again.  Because the association between a container and volume is done
*after* the scheduler picks a node to spin it up, the most optimal volume is
chosen. This can be a volume that has data local to the node enable hyper
convergent architectures.

## Implementation

When a scaled volume is requested to be attached, an attempt to attach a volume
that has data local to the node is made. If not such volume is found, then one is
created if the number of instances is below the scale count.  An error is
returned, If a free volume is not found and no more volumes can be created.

## Usage

A scaled volume can be created using the pxctl CLI, docker CLI, or inline
volume spec. 

## pxctl CLI

```
[root@porx]# pxctl volume create elk_vol --scale 10
Volume successfully created: 232783593254518125

[root@porx]# pxctl volume list
ID                      NAME            SIZE         HA      SHARED  ENCRYPTED       IO_PRIORITY     SCALE   STATUS
232783593254518125      elk_vol         1 GiB        1       no      no              LOW             10      up - detached

```

## Docker CLI
```
[root@porx]# docker volume create --driver pxd --name elk_vol --opt scale=10
[root@porx]# pxctl volume list
ID                      NAME            SIZE         HA      SHARED  ENCRYPTED       IO_PRIORITY     SCALE   STATUS
232783593254518125      elk_vol         1 GiB        1       no      no              LOW             10      up - detached
```

## FAQ

### Update scale factor after volume creation

Volume scale factor can be modified after the volume is created

```
[root@porx]# pxctl volume update scale_vol --scale 12
[root@porx]# pxctl volume list
ID                      NAME            SIZE         HA      SHARED  ENCRYPTED       IO_PRIORITY     SCALE   STATUS
232783593254518125      elk_vol         1 GiB        1       no      no              LOW             12      up - detached
```

Decreasing the scaled volume only affects future volumes - volumes created as result of scale will not be deleted.


### Can I attach more than one instance of a scaled volume on the same node?

If multiple containers request the same scaled volume on a node, only one 
instance is created. The volume will be shared between the client container.

### How do I delete a scaled volume

At present all the instances of the scaled volume need to be deleted one by one.

### How do I request for a specific instance of a scaled volume

You can always specify a spcecific instance of scaled volume to be used.
