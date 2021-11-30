gpu passthrough vdsm hook
=================================
oVirt VDSM Hook for passing through GPU

Installation:
* Use engine-config to create a custom property:

```bash
sudo engine-config -s "UserDefinedVMProperties=gpupassthrough=^(true|false)$"
```

* To confirm that the property was added:

```bash
sudo engine-config -g UserDefinedVMProperties
```

* Restart the engine:

```bash
sudo systemctl restart ovirt-engine
```

* Place 99_gpu_passthrough in /usr/libexec/vdsm/hooks/before_vm_start:

```bash
sudo cp 99_gpu_passthrough /usr/libexec/vdsm/hooks/before_vm_start
sudo chmod +x /usr/libexec/vdsm/hooks/before_vm_start/99_gpu_passthrough
```

Usage:
Edit the VM, add the following custom property to hide kvm from the VM and add a vendor id:

**gpupassthrough=true**

Details:
This adds the following entries to the guest XML:

```xml
  <features>
    ...
    <hyperv>
      ...
      <vendor_id state='on' value='whatever'/>
    </hyperv>
    ...
    <kvm>
      <hidden state='on'/>
    </kvm>
  </features>
```

**Troubleshooting**

* failed to execute

If you see the following error in your event log:  
```
VM xxx is down with error. Exit message: Hook Error: (b'taskset: failed to execute /usr/libexec/vdsm/hooks/before_vm_start/99_gpu_passthrough: No such file or directory\n',).
```

Although it's not clear from the message, it's most likely the python interpreter that's not been found, not the 99_gpu_passthrough script.  

To fix the problem, replace the shebang line in `99_gpu_passthrough`

`#!/usr/bin/python2`

with the equivalent from `50_hostedengine`, for example:

`#!/usr/libexec/platform-python`
  
  
  
Adapted from this thread:
https://www.mail-archive.com/users@ovirt.org/msg40377.html

Thanks, Martin!
