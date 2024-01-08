---
title: Setting up Burp Suite on Android 7/8/9/10
date: 2021-02-15
draft: false
---

NOTE: You will need a rooted phone for this tutorial.  
  
If you’ve recently tried to use Burp for Android versions Nougat or higher, you’ll know that many apps have simply stopped functioning through Burp.

Most notably, apps like the play store complain about not having internet, even though you can connect to HTTP sites and probably get a cert error going to HTTPS sites, but it still works.

Even after you resolve these errors by installing your certificate in the system store, HTTPS still will not function due to Burp’s cert having an invalid length of time.

The solution to this is to generate your own certificate, use the Android Debug Bridge utility to install it in your phone or emulator, and finally import the keystore into Burp so you can decrypt the traffic. You’ll need to have OpenSSL, ADB, and obviously Burp Suite installed to complete this tutorial. I recommend using Kali Linux and simply installing ADB with:

```
apt install adb -y
```

After you’ve installed ADB, you’re all set to start. Generate a new certificate and key pair using the following command:

```
openssl req -x509 -newkey rsa:2048 -keyout private.key -out public.cer -days 365  
openssl pkcs12 -export -out pair.pfx -inkey private.key -in public.cer
```

Make sure to set a password on the key, and give the certificate an Organization Name. The password will be used when we import the key into Burp, and the Organization Name will be what shows once we have installed the cert in our Android device.

Now we have to format the cert correctly. Use the following command to get a hash of the certificate, and then run the second command using the output of the first command to rename it so Android will recognize it as a system cert.

```
openssl x509 -inform PEM -subject_hash_old -in public.cer | head -1  
cp public.cer <INSERT_HASH_HERE>.0
```

We should now have four files, our original cert, our original private key, a pair of the two (.pfx), and our system cert.

Now we’re ready to install the certificate. If you’re using a VM, make sure that you’ve passed the USB connection through to your virtual machine. Also make sure that the device has “USB debugging mode” enabled from the developer settings.

I’m using Nox player for this example. Nox easily allows you to enable root, and then do adb connect 127.0.0.1:62001 to talk to the emulator.

If you’re using a physical device, use adb devices to ensure that your device is attached properly, then use adb root to connect to it for a root shell. If you have a consumer device that doesn’t allow you to use root, you can still perform these steps as long as your device is rooted and you can su into a root shell from an unprivileged shell.

Now that we’re this far, you’re going to want to get into your root shell and perform the following commands

```
adb root  
adb shell avbctl disable-verification  
adb disable-verity  
adb remount  
adb reboot  
adb root  
adb remount  
adb push <SYSTEM CERT> /system/etx/security/cacerts/<SYSTEM CERT>  
adb shell chmod 644 /system/etx/security/cacerts/<SYSTEM CERT>
```

Note: If using an android studio emulator, it helps to set -writable-system and -http-proxy from the command line (./emulator/emulator @Pixel_XL_API_30 -verbose -writable-system -http-proxy localhost:8080)

**Last step!** 
Within the Burp Suite proxy options, import the PKCS12 keystore using the file and password you generated earlier.

If all went well, you should start seeing traffic appear 🙂

[https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/](https://web.archive.org/web/20220520135811/https://blog.ropnop.com/configuring-burp-suite-with-android-nougat/)