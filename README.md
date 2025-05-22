# OpenStack-DR
## MVP for Pure / Red Hat / Trilio OpenStack DR/Failover service.

This playbook is only for an instance with a single cinder-based volume. If the volume is bootable, then an instance will be started using it.

The boot volume on the primary openstack cloud is of a volume type where the volume has been replicated (**asynchronously or synchronously**) between two Pure FlashArrays.

The playbook requires that `clouds.yaml` has authentication details for both source and target OS clouds.

Asynchronous replication does not require the source Nova instances to be shutdown when the DR is performed.
Synchronous replication requires that the Nova instances be quiesed before performing the DR.

**NOTE:** The `volume_manage` module is not currently upstream. Details of the actual code are available here: https://review.opendev.org/c/openstack/ansible-collections-openstack/+/947517

Under current upstream cinder code replication failover is per backend, so it cannot be used for per tenant or per application failover.

To enable something that would effectively enable these scripts to work on a per tenant or application basis the following methodology should be performed, based on a single physical FlashArray connected to the source OS cloud, replicated to another FlashArray connected to a second OS cloud

In the `cinder.conf` define multiple backends, all pointing to the same physical source-side FlashArray. For example:

```
[DEFAULT]
...
enabled_backends = fa_tenant_1, fa_tenant_2, fa_tenant_3
...

[fa_tenant_1]
volume_backend_name = fa_tenant_1
volume_driver = cinder.volume.drivers.pure.PureISCSIDriver
san_ip = <FA 1 MGMT IP address>
replication_device = backend_id:fa-2,san_ip:<IP address of FA 2>,api_token:<API token of FA 2>,type:async
pure_api_token = <FA 1 API token>
pure_replication_pg_name = tenant_1

[fa_tenant_2]
volume_backend_name = fa_tenant_2
volume_driver = cinder.volume.drivers.pure.PureISCSIDriver
san_ip = <FA 1 MGMT IP address>
replication_device = backend_id:fa-2,san_ip:<IP address of FA 2>,api_token:<API token of FA 2>,type:async
pure_api_token = <FA 1 API token>
pure_replication_pg_name = tenant_3

[fa_tenant_3]
volume_backend_name = fa_tenant_3
volume_driver = cinder.volume.drivers.pure.PureISCSIDriver
san_ip = <FA 1 MGMT IP address>
replication_device = backend_id:fa-2,san_ip:<IP address of FA 2>,api_token:<API token of FA 2>,type:async
pure_api_token = <FA 1 API token>
pure_replication_pg_name = tenant_3
```

Create a replicated volume type for each backend and assign to individual tenants:
```
openstack volume type create Tenant1
openstack volume type set --property volume_backend_name=fa_tenant_1
openstack volume type set --property replication_enabled='<is> True' Tenant1
openstack volume type set --property replication_type='<in> async' Tenant1
openstack volume type set --project <tenant 1 name>

openstack volume type create Tenant2
openstack volume type set --property volume_backend_name=fa_tenant_2
openstack volume type set --property replication_enabled='<is> True' Tenant2
openstack volume type set --property replication_type='<in> async' Tenant2
openstack volume type set --project <tenant 2 name>

openstack volume type create Tenant3
openstack volume type set --property volume_backend_name=fa_tenant_3
openstack volume type set --property replication_enabled='<is> True' Tenant3
openstack volume type set --property replication_type='<in> async' Tenant3
openstack volume type set --project <tenant 3 name>
```
