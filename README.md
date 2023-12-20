# fedora-bootc

Fork of https://github.com/centos/centos-bootc to build Fedora 39 images.

Experimental, use on your own risk.

## Usage

```
podman pull ghcr.io/ondrejbudai/fedora-bootc:39
```

## Deploying

### Hetzner

1) Create a cx11 instance (the distribution doesn't matter).
2) Click *Enable rescue & power cycle*
3) Build a qcow2:
   ```
   mkdir -p output
   tee output/config.json <<EOF
   {
     "blueprint": {
       "customizations": {
         "user": [
           {
             "name": "foobar",
	         "password": "EDITME",
             "key": "ssh-ed25519 AAAA...",
             "groups": [
               "wheel"
             ]
           }
         ]
       }
     }
   }
   EOF
   
   sudo podman run \
       --rm \
       -it \
       --privileged \
       --pull=newer \
       --security-opt label=type:unconfined_t \
       -v $(pwd)/output:/output \
       quay.io/centos-bootc/bootc-image-builder:latest \
       --type qcow2 \
       --config /output/config.json \
       ghcr.io/ondrejbudai/fedora-bootc:39
   ```

4) Copy the `qcow2` over, write it to the hardrive and reboot:
   ```
   scp output/qcow2/disk.qcow2 root@[IP ADDRESS]:
   ssh root@[IP ADDRESS] "qemu-img convert -O raw disk.qcow2 /dev/sda && shutdown now"
   ```
5) Make a snapshot
6) Make as many fedora-bootc instances as you want! 🎉

Kudos to https://major.io/p/deploy-fedora-coreos-in-hetzner-cloud/
