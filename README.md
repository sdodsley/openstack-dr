# openstack-dr
MVP for Pure / Red Hat / Trilio OpenStack DR/Failover service.

This playbook is only for an instance with a single cinder-based boot volume.

The boot volume on the primary openstack cloud is of a volume type where the volume has been replicated (asynchronously) between two Pure FlashArrays.
