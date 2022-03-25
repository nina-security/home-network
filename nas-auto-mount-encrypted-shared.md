# Synology NAS: Automatic Mounting of Encrypted Shares

My Synology NAS supports encrypted folders. Using this security feature implies that the encrypted folders need to be mounted with the respective encryption password after startup. This can be tedious. Synology supports automatic mounting of encrypted shares using their Key Manager. However, I don't see a big benefit of the encrypted shares then, e.g. in case of theft.

Below is my custom solution for using encrypted shares which are automatically mounted on startup without leaving the keys on the same device.

## Actors

- Synology NAS (quite obvious) with a custom admin user `monkey` (Synology forced me to provide a name which is not admin ;) ). IP address is `192.168.x.y`.
- Raspberry Pi which is always running on my network with user `nas`. IP address is `192.168.a.b`.

## Idea 

On startup, the NAS connects via SSH to the Raspberry Pi, and starts a SSH connection back to mount all encrypted shares. This means, the folder's encryption keys are stored on the Raspberry Pi.
Even if an attacker gets both devices, the setup would not run in his environment (except that he would guess the IP addresses I use for both devices and assign them, but I hope that's negligible).

The final flow looks like this: 

1. NAS boots and root runs `run-mount-shares-on-boot-init.sh`.
1. This triggers `mount-shares-on-boot-init.sh` run by `monkey` which connects via SSH to the Raspberry Pi as user `nas` and...
1. ...runs `mount-shares-on-boot.sh` on the Raspberry Pi which connects to the NAS and mounts the encrypted shares.

## Preparation on Raspberry Pi

1. Create a user `nas`: `sudo useradd -m nas`
1. Provide a password: `sudo passwd nas`
1. Adapt home directory permissions: `sudo chmod 700 /home/nas/`
1. Copy the file `mount-shares-on-boot.sh` (see below) to `/home/nas/mount-shares-on-boot.sh`.
1. Adapt file permissions: `chmod 750 /home/nas/mount-shares-on-boot.sh`
1. As the user `nas`, create a SSH key pair: `ssh-keygen`
1. Later: Add the SSH public key on the NAS to the authorized keys file `/var/services/homes/monkey/.ssh/authorized_keys`.

### File mount-shares-on-boot.sh
```bash
#!/bin/bash

shares=("share-1" "share-2" "share-3")
passwords=("password-for-share-1" "password-for-share-2" "password-for-share-3")

mount_encrypted_shares() {
        echo "Retrieving mount status..." > mount-shares-on-boot.log
        output_mount=$(ssh -i /home/nas/.ssh/id_rsa monkey@192.168.x.y "mount")

        length=${#shares[@]}
        for (( j=0; j<${length}; j++ ));
        do
                share=${shares[$j]}
                password=${passwords[$j]}
                if [[ $output_mount == *"@${share}@"* ]]; then
                        echo "Share ${share} already mounted." >> mount-shares-on-boot.log
                else
                        echo "Share ${share} not mounted, mounting..." >> mount-shares-on-boot.log
                        ssh -i /home/nas/.ssh/id_rsa_ monkey@192.168.x.y "sudo /usr/syno/sbin/synoshare --enc_mount $share '$password'"
                fi
        done
}

mount_encrypted_shares
```

## Preparation on Synology NAS

1. In home directory of `monkey` (`/var/services/homes/monkey`), create the file `mount-shares-on-boot-init.sh` (see content below).
1. Set file permissions: `chmod 700 mount-shares-on-boot-init.sh`.
1. Create a SSH key pair: `ssh-keygen`
1. Add the SSH public key on the Raspberry Pi to the authorized keys file `/home/nas/.ssh/authorized_keys`. Remember to also add the public key to the NAS (see steps above for Raspberry Pi).
1. To run the script on startup, create the file `/usr/local/etc/rc.d/run-mount-shares-on-boot-init.sh` (see content below).
1. Make the file executable: `chmod +x /usr/local/etc/rc.d/run-mount-shares-on-boot-init.sh`.

### File mount-shares-on-boot-init.sh

```bash
#!/bin/bash

ssh -i /var/services/homes/monkey/.ssh/id_rsa nas@192.168.a.b '/bin/bash mount-shares-on-boot.sh'
```

### File run-mount-shares-on-boot-init.sh

```bash
su - monkey -c "/var/services/homes/monkey/mount-shares-on-boot-init.sh"
```

## Final Preparation 

Mutually connect from one device to the other via SSH and accept the servers SSH keys.

To ensure that everything's fine, you can run each script file and check that the shares are mounted. Remember to unmount the shares between your tests. 

Then reboot the NAS to check that everything's also working on startup. 
