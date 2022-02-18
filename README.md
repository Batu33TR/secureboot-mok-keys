Create the public and private key for signing the kernel:

openssl req -config ./mokconfig.cnf \
        -new -x509 -newkey rsa:2048 \
        -nodes -days 36500 -outform DER \
        -keyout "MOK.priv" \
        -out "MOK.der"

Convert the key also to PEM format (mokutil needs DER, sbsign needs PEM):
openssl x509 -in MOK.der -inform DER -outform PEM -out MOK.pem

Enroll the key to your shim installation:
sudo mokutil --import MOK.der

You will be asked for a password, you will just use it to confirm your key selection in the next step, so choose any.

Restart your system. You will encounter a blue screen of a tool called MOKManager. Select "Enroll MOK" and then "View key". Make sure it is your key you created in step 2. Afterwards continue the process and you must enter the password which you provided in step 4. Continue with booting your system.

Verify your key is enrolled via:
sudo mokutil --list-enrolled

Sign your installed kernel (it should be at /boot/vmlinuz-[KERNEL-VERSION]):
sudo sbsign --key MOK.priv --cert MOK.pem /boot/vmlinuz-[KERNEL-VERSION] --output /boot/vmlinuz-[KERNEL-VERSION].signed

Copy the initram of the unsigned kernel, so we also have an initram for the signed one.
sudo cp /boot/initrd.img-[KERNEL-VERSION]{,.signed}

Update your grub-config
sudo update-grub

Reboot your system and select the signed kernel. If booting works, you can remove the unsigned kernel:
sudo mv /boot/vmlinuz-[KERNEL-VERSION]{.signed,}
sudo mv /boot/initrd.img-[KERNEL-VERSION]{.signed,}
sudo update-grub

Now your system should run under a signed kernel and upgrading GRUB2 works again. If you want to upgrade the custom kernel, you can sign the new version easily by following above steps again from step seven on. Thus BACKUP the MOK-keys (MOK.der, MOK.pem, MOK.priv).
