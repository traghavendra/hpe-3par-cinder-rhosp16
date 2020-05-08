## HPE 3PAR and Primera Cinder volume custom container for RHOSP16
Manual building of a container for HPE 3PAR and Primera cinder volume for RHOSP16

1.	Create Dockerfile

[Dockerfile](https://github.com/hpe-storage/hpe-3par-cinder-rhosp16/blob/master/Dockerfile)
      
2.	Build the podman image
```
sudo buildah bud --format docker --build-arg http_proxy=http://16.85.88.10:8080 --build-arg https_proxy=http://16.85.88.10:8080 . 
```
Notice the dot at the end.

3.	Run podman images command to verify if the container got created successfully or not
```
podman images
REPOSITORY                                           TAG                 IMAGE ID                       CREATED             SIZE
<none>                                               <none>              b497daac7539        21 seconds ago      1.01 GB
```

4.	Add tag to the image created
```
sudo podman tag <image id> cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-volume-hpe:1.0
```

5.	Run podman images command to verify the repository and tag is correctly updated to the docker image
```
podman images
REPOSITORY                                                                                                            TAG                                IMAGE ID               CREATED                    SIZE
cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-volume-hpe                      1.0                             b497daac7539        2 minutes ago       1.01 GB
```

6.	Push the container to a local registry
```
sudo openstack tripleo container image push --local cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-volume-hpe:1.0
```

7.	Create new env file “custom_container_[iscsi|fc].yaml” under /home/stack/custom_container/ with only the custom container parameter and other backend details. Sample files are available in [custom_container](https://github.com/hpe-storage/hpe-3par-cinder-rhosp16/blob/master/custom_container) folder for reference
```
parameter_defaults:
    DockerCinderVolumeImage: cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-volume-hpe:1.0
```

8.	Deploy overcloud
```
openstack overcloud deploy --templates /usr/share/openstack-tripleo-heat-templates -e /home/stack/templates/node-info.yaml -e /home/stack/containers-prepare-parameter.yaml -e /home/stack/custom_container/custom_container_[iscsi|fc].yaml --ntp-server 16.110.135.123 --debug
```

9.	SSH to controller node from undercloud and check the docker process for cinder-volume
```
(overcloud) [heat-admin@overcloud-controller-0 ~]$ sudo podman ps | grep cinder
56baa616ae2c  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-volume:16.0-90       /bin/bash /usr/lo...  2 weeks ago  Up 2 hours ago         openstack-cinder-volume-podman-0
11777e9848c1  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-scheduler:16.0-92    kolla_start           2 weeks ago  Up 2 hours ago         cinder_scheduler
5d6d06cacc19  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-api:16.0-90          kolla_start           2 weeks ago  Up 2 hours ago         cinder_api_cron
4043187451d2  cld13b4.ctlplane.set.rdlabs.hpecorp.net:8787/rhosp-rhel8/openstack-cinder-api:16.0-90          kolla_start           2 weeks ago  Up 2 hours ago         cinder_api
```

10.	Verify the module installed is present in the cinder-volume container
```
(overcloud) [heat-admin@overcloud-controller-0 ~]$ sudo podman exec -it 56baa616ae2c bash
()[root@overcloud-controller-0 /]# pip list | grep 3par
python-3parclient        4.2.11
```
