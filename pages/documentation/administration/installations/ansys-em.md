# ANSYS ElectroMagnetics

**VERSION=2024R1**

**SHORT_VERSION=241**

From your laptop copy `ELECTRONICS_${VERSION}_LINX64.tgz` to `/local` of the login node.

As easybuild on login node with X forwarding enabled:

```shell
cd /local/ANSYS-EM
tar -xzf ../ELECTRONICS_${VERSION}_LINX64.tgz
cd Electronics_231_linx64
cp VerifyOS.bash VerifyOS.bash.orig 
echo "exit 0" > VerifyOS.bash 
mkdir /sw/rl8/zen/app/ANSYS-EM/${VERSION}/
./install
    Install Electromagnetcs Suite
    installpath: /sw/rl8/zen/app/ANSYS-EM/${VERSION}
    use /tmp (no read/write for all)
    shared application: /sw/rl8/zen/app/ANSYS-EM/${VERSION}/${SHORT_VERSION}/Linux64
    license server: ansys-research.lic.tue.nl 1055

cd /sw/rl8/zen/app/ANSYS-EM/${VERSION}/${SHORT_VERSION}/Linux64
cp VerifyOS.bash VerifyOS.bash.orig 
echo "exit 0" > VerifyOS.bash 
# Create a module : /sw/rl8/zen/mod/all/ANSYS-EM/${ANSYS-EM_VERSION}
# Check the path to the license server : ansys-research.lic.tue.nl
# Create a module : /sw/rl8/zen/mod/all/ANSYS-EM/${ANSYS-EM_VERSION}-edu
# Check the path to the license server : ansys-edu.lic.tue.nl
```

The module files for ANSYS-EM/<VERSION> need to be created.