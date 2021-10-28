Hi,
       You can follow the steps below to do that.

1) Stop all the virtual machines.

2) Move all the storage domains other than hosted_storage to maintenance
which will unmount them from all the nodes.

3)  Move HE to global maintenance ´hosted-engine --set-maintenance --mode=global´

4) stop HE vm by running the command ´hosted-engine --vm-shutdown´

5) confirm that engine is down using the command ´hosted-engine --vm-status´

6) stop ha agent and broker services on all the nodes by running the
command: `systemctl stop ovirt-ha-broker ; systemctl stop ovirt-ha-agent`

7) umount hosted-engine from all the hypervisors ´hosted-engine --disconnect-storage`

8) stop all the volumes.

9) power off all the hypervisors.


To bring it up back again below steps will help.


1) Power on all the hypervisors.

2) start all the volumes

3) start ha agent and broker services on all the nodes by running the
command 'systemctl start ovirt-ha-broker' ; 'systemctl start ovirt-ha-agent'

4) Move hosted-engine out of global maintenance by running the command
hosted-engine
--set-maintenance --mode =none

5) give some time for the HE to come up. check for hosted-engine
--vm-status to see if HE vm is up.

6) Activate all storage domains from UI.

7) start all virtual machines.

Hope this helps !!!

Thanks

kasturi.
