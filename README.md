# README

# eCryptfs

Desde Ubuntu 18.04 no se recomienda más el uso de eCryptfs porque sus defectos pueden
provocar pérdida de datos.

Durante varios años Ubuntu ofreció la posibilidad de cifrar el HOME del usuario mediante eCryptfs. A partir de Bionic (18.04) esa opción no existe más.

Lo recomendado es utilizar cifrado a nivel de dispositivo de bloques, por ejemplo para toda la partición HOME, /, etc.

Por motivos de interoperabilidad con equipos portátiles que usaban eCryptfs se desarrolló el siguiente tutorial que permite realizar una 
instalación de Ubuntu 18.04 **sin cifrado en el HOME de un usuario dado** y pasarlo a una instalación **con cifrado en el HOME de un usuario dado**.

* [Cómo pasar de HOME sin cifrar a HOME cifrado](escryptfs.md)
* http://ecryptfs.org/ homepage
* https://bugs.launchpad.net/ubuntu/+source/ecryptfs-utils/+bug/1756840 Buggy, under-maintained, not fit for main anymore; alternatives exist / 2018-03-19


#  Block device encryption such as via dm-crypt 

TODO

#  Block device encryption such as via LUKS

TODO

#  fscrypt front-end to filesystem-native encryption (ext4)

TODO

# Encrypted Private Directory

TODO

https://help.ubuntu.com/community/EncryptedPrivateDirectory

# How to Encrypt Linux Partitions with VeraCrypt on Ubuntu

TODO

https://vitux.com/how-to-encrypt-linux-partitions-with-veracrypt-on-ubuntu/
