# Lanzar una instacia en OpenStack desde Ansible

De acuerdo a la documentacion oficial de :q!
TASK [launch an instance] ******************************************************************************************************************************
fatal: [localhost]: FAILED! => {"changed": false, "msg": "To utilize this module, the installed version ofthe openstacksdk library MUST be >=0.12.0"}
        to retry, use: --limit @/home/stack/post/instance.retry

PLAY RECAP *********************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=1

(overcloud) [stack@undercloud post]$ ^C
(overcloud) [stack@undercloud post]$ ^C
(overcloud) [stack@undercloud post]$ ^C
(overcloud) [stack@undercloud post]$ sudo yum install python2-openstacksdk

(overcloud) [stack@undercloud post]$ sudo yum install  python3-pip

(overcloud) [stack@undercloud post]$ sudo pip3 install openstacksdk==0.12.0

