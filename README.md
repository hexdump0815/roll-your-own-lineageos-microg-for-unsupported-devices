# how to build lineageos with microg for unsupported devices?

in case you do not know what lineageos and/or microg is:
* lineageos: https://lineageos.org/
* lineageos with microg: https://lineage.microg.org/

these here are some of my notes on how to build lineageos with microg for devices which are no longer or were never officially supported by lineageos and thus for which there are no regularly updated new offical builds available. 

the build is based on and done with https://github.com/lineageos4microg/docker-lineage-cic

for the build an intel x86_64 machine with working docker is required. 8gb (or more) of ram are recommended, but it will also work with 4gb and a lot of swap (and then swapping of course) - i did my lineage 14.1 builds on an old and slow thinkpad x301 with 4gb of ram and an ssd with 8gb of swap configured and the build took around 12 hours per device. with a modern machine, a faster cpu and more ram it should be significantly faster. what is required as well is a lof of disk space as a lot of sources are getting downloaded before and for the build - something around 200-300gb of free disk space are the minimum.

as i initially had only 4gb of swap configured (which is not enough with 4gb of ram) i added some more (4gb in my case) swap temporarily (all commands are to be run as root):
```
dd if=/dev/zero of=/tmp/extra-swap bs=1024k count=4096 status=progress
chmod 600 /tmp/extra-swap
mkswap /tmp/extra-swap
swapon /tmp/extra-swap 
free
```

i did the build in a directoy "microg" in my home directory and prepared it this way:
```
mkdir ${HOME}/microg
cd ${HOME}/microg
mkdir -p lineage zips logs cache keys manifests
```

next one needs to prepare and copy the manifest file for your device into the manifests directory. in this repo you can find the manifest files which worked for me for six devices:
* athene - motorola g4 plus mobile - https://wiki.lineageos.org/devices/athene
* harpia - motorola g4 play mobile - https://wiki.lineageos.org/devices/harpia
* n1awifi - samsung galaxy note 10.1 2014 tablet - https://wiki.lineageos.org/devices/n1awifi
* bullhead - google nexus 5x mobile (no official 14.1 builds anymore) - https://wiki.lineageos.org/devices/bullhead
* hammerhead - google nexus 5 mobile - https://wiki.lineageos.org/devices/hammerhead
* potter - motorola g5 plus mobile
* pme - htc 10 mobile - https://wiki.lineageos.org/devices/pme

besides that the android_prebuilts_prebuiltapks.xml manifest file should be copied into the manifests file. i'm not sure anymore if the potter-16.0 manifest has been really tested, so please keep this in mind in case you want to use it.

some notes about how to prepare the manifest files in case one does not have one yet: first find some device repo for the device you want to build for - either an old and not longer actively built lineageos repo or a non official lineageos repo. this repo and all the following need to have a cm-xy or lineage-xy branch for building them in lineageos version xy (cm-xy up to 14.1 and lineageos-xy afterwards). if you have found this (example for potter 14.1: https://github.com/boype/android_device_motorola_potter), then add it to your new manifest file (use some other manifest file as an example for its format). next take a look into the file lineage.dependencies inside of ithat repo (example for potter 14.1: https://github.com/boype/android_device_motorola_potter/blob/cm-14.1/lineage.dependencies) and add those repos as well. if the device repo has dependencies to other device repos, then do the same for those as well. then search (mostly near the device repo) for the corresponding kernel and vendor repos and add them too. sometimes some repos will depend on others as well (like in the potter example the kernel repo on android_kernel_motorola_msm8953) - worst case you find them according to the errors you will get during the build and maybe some searching on the net. in case you do not find a proper vendor repo, maybe have a look at https://github.com/TheMuppets ...

some notes on keys and signing: by default builds are signed with android test keys - if you want to sign your builds with your own developer keys (which is recommended), simply set the build option SIGN_BUILDS to true and as part of the build some new developer keys will be generated. IMPORTANT: save those created keys in the "keys" directory somewhere and use them for all new builds you do, so that they will be singed with the same keys - otherwise you will run in to problems when you update your device and want to keep your /data partition (in case you run into that problem, have a look at https://download.lineage.microg.org/extra and adjust it accordingly with your old and new keys, which is no real fun).

ok - now we are ready to start the actual build (only build for one device at a time - building for multiple devices in one run did not work too well for me): simply run the following command as your user (not as root - this is not required):
```
docker run \
    -e "BRANCH_NAME=cm-14.1" \
    -e "DEVICE_LIST_CM_14_1=potter" \
    -e "SIGN_BUILDS=true" \
    -e "SIGNATURE_SPOOFING=restricted" \
    -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \
    -e "INCLUDE_PROPRIETARY=false" \
    -v "${HOME}/microg/lineage:/srv/src" \
    -v "${HOME}/microg/zips:/srv/zips" \
    -v "${HOME}/microg/logs:/srv/logs" \
    -v "${HOME}/microg/cache:/srv/ccache" \
    -v "${HOME}/microg/keys:/srv/keys" \
    -v "${HOME}/microg/manifests:/srv/local_manifests" \
    lineageos4microg/docker-lineage-cicd
```

if everything runs well it should run through without errors and result in the final zip file to be installed on your device in the "zips" directory (check out the lineageos doc for the device or the net on how to install the zip). if you run into errors you will have to search the net to find out what went wrong - most probably something is not yet right with your manifest file in that case.

maybe this is useful for someone: in the release section you can find my builds for those six devices - athene and n1awifi are usuall tested to work perfectly fine for sure, the others might or might not be tested but should work anyway i hope.

good luck!
