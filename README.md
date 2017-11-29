# Zabbix: solve memory monitoring issue inside LXC containers

Zabbix have some problems with memory collecting from cgroups limited
containers.<br> If you using Promxox, you know what I mean: The available memory
collected worng without calculating buffers and cache memory.<br> Zabbix have
[bug
report]([https://support.zabbix.com/browse/ZBX-12164](https://support.zabbix.com/browse/ZBX-12164)),
but it seems that no one don’t want to fix it soon.<br> So let’s fix it together
byself.

First we need to create new user parameter for zabbix, I used `ct.memory.size`
name similarly built-in `vm.memory.size` parameter, but with my own single line
script.

Please be sure that you have newest version of `free` tool, `3.3.10` and higher,
because old one have different output format.

    $ free -V
    free from procps-ng 3.3.10

Edit `/etc/zabbix/zabbix_agentd.conf:`

Add user parameter for retrieve memory information:

    UserParameter=ct.memory.size[*],free -b | awk '{total=$ 2; used=($ 3+$ 5); pused=(($ 3+$ 5)*100/$ 2); free=$ 4; pfree=($ 4*100/$ 2); shared=$ 5; buffers=$ 6; cache=$ 6; available=($ 6+$ 7); pavailable=(($ 6+$ 7)*100/$ 2); } $ 1 == "Mem:" { if("$1" == "") {printf("%.0f", total )} else {printf("%.0f", $1 "" )} }'

Add another one for retrieve swap information:

    UserParameter=ct.swap.size[*],free -b | awk '{total=$ 2; used=$ 3; free=$ 4; pfree=($ 4*100/$ 2); pused=($ 3*100/$ 2)}{ if("$1" == "") } $ 1 == "Swap:" { if("$1" == "") {printf("%.0f", free )} else {printf("%.0f", $1 "" )} }'

It will provide you support for next parameters:

    ct.memory.size[total]
    ct.memory.size[free]
    ct.memory.size[buffers]
    ct.memory.size[cached]
    ct.memory.size[shared]
    ct.memory.size[used]
    ct.memory.size[pused]
    ct.memory.size[available]
    ct.memory.size[pavailable]
    ct.swap.size[total]
    ct.swap.size[free]
    ct.swap.size[shared]
    ct.swap.size[used]
    ct.swap.size[pused]
    ct.swap.size[available]
    ct.swap.size[pavailable]

Don’t forget to restart `zabbix-agent.service` after

Ok, now we can check is our parameter working from zabbix server:

    $ zabbix_get -s <container_ip> -k ct.memory.size[available]
    1709940736

Ok it’s working.

Now let’s configure zabbix for use them. In Zabbix Interface:

Go Configuration → Templtes

* Make full copy “Template OS Linux” to “Template Linux Container”
* Open “Template Linux Container” → Items
* Replace all `vm.memory.size` items to `ct.memory.size`.
* Replace all `system.swap.size` items to `ct.swap.size`:<br> You also need to
remove commas in key filed here. Example:<br> replace `system.swap.size[,free]`
to `ct.swap.size[free]`

Or just download and import [my zabbix
template](https://github.com/kvaps/zabbix-linux-container-template/blob/master/zbx_linux_container_template.xml).

Next, go to the Configuration → Hosts

* Unlink and clear “Template OS Linux” from your hosts
* Attach “Template Linux Container”

Wait some time and check the graphics:

Job is done!

* [Containers](https://medium.com/tag/containers?source=post)
* [Zabbix](https://medium.com/tag/zabbix?source=post)
* [Proxmox](https://medium.com/tag/proxmox?source=post)
* [Lxc](https://medium.com/tag/lxc?source=post)

By clapping more or less, you can signal to us which stories really stand out.

### [kvaps](https://medium.com/@kvapss)