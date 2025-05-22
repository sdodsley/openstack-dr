# openstack-dr
MVP for Pure / Red Hat / Trilio OpenStack DR/Failover service.

This playbook is only for an instance with a single cinder-based boot volume.

The boot volume on the primary openstack cloud is of a volume type where the volume has been replicated (**asynchronously**) between two Pure FlashArrays.

The playbook requires that `clouds.yaml` has authentication details for both source and target OS clouds.

NOTE: The `volume_manage` module is not currently upstream. Details of the actual code are availble here: https://review.opendev.org/c/openstack/ansible-collections-openstack/+/947517
