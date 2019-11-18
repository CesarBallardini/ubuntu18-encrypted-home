# README

## 1. Creo la VM de pruebas

```bash
time vagrant up
vagrant ssh -c "sudo sed -i 's/PasswordAuthentication no/PasswordAuthentication yes/' /etc/ssh/sshd_config"
vagrant reload

time vagrant snapshot save encryptedhome recien-instalado
time vagrant snapshot list


vagrant snapshot restore --help
vagrant snapshot delete --help
```

## 2. Creo una cuenta y HOME sin cifrar

Se debe realizar desde una cuenta con permisos de `root` que no sea la
CUENTA que se desea cifrar su HOME. En nuestro caso usaremos la cuenta 
`vagrant` para realizar las tareas administrativas, que es la 
predeterminada cuando usamos Vagrant boxes.

* creamos la cuenta, y su home sin cifrar; agregamos algunos archivos de prueba

```bash
vagrant ssh -c '

export CUENTA=cballard31
export PASSWD=perico

sudo useradd -s /bin/bash -m -m "${CUENTA}"
sudo usermod -aG sudo "${CUENTA}"
echo "${CUENTA}:${PASSWD}" | sudo chpasswd 
'
```

* ahora desde el host:

```bash
export CUENTA=cballard31
export PASSWD=perico

# verificamos que la cuenta y su contraseña funcionan
sshpass -p "${PASSWD}" vagrant ssh -c  "id"    --plain -- -l "${CUENTA}"

# copiamos algunos archivos desde el host hacia la VM:
sshpass -p "${PASSWD}" vagrant ssh -c 'mkdir ~/archivos/ ; find  /srv/usr-share/doc/ -maxdepth 1 -type d  -wholename "*/[a-cv-z]*" | while read dir ; do rsync -Pav $dir ~/archivos/ ; done ' --plain -- -l "${CUENTA}"

```

## 3. Instalo el paquete de cifrado y cifro el HOME de CUENTA

* http://ecryptfs.org/ 
* https://bugs.launchpad.net/ubuntu/+source/ecryptfs-utils/+bug/1756840 Buggy, under-maintained, not fit for main anymore; alternatives exist / 2018-03-19


* Dentro de la VM:

```bash
export CUENTA=cballard31
export PASSWD=perico

sudo apt-get install -y ecryptfs-utils

printf "%s" "${PASSWD}" | ecryptfs-add-passphrase --fnek -


sudoecryptfs-migrate-home -u "${CUENTA}"

# Al final muestra el siguiente mensaje de advertencia:

# ************************************************************************
# YOU SHOULD RECORD YOUR MOUNT PASSPHRASE AND STORE IT IN A SAFE LOCATION.
#   ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase
# THIS WILL BE REQUIRED IF YOU NEED TO RECOVER YOUR DATA AT A LATER TIME.
# ************************************************************************

...

# ========================================================================
# Some Important Notes!
# 
#  1. The file encryption appears to have completed successfully, however,
#     cballard MUST LOGIN IMMEDIATELY, _BEFORE_THE_NEXT_REBOOT_,
#     TO COMPLETE THE MIGRATION!!!
# 
#  2. If cballard can log in and read and write their files, then the migration is complete,
#     and you should remove /home/cballard.cnuHh3or.
#     Otherwise, restore /home/cballard.cnuHh3or back to /home/cballard.
# 
#  3. cballard should also run 'ecryptfs-unwrap-passphrase' and record
#     their randomly generated mount passphrase as soon as possible.
# 
#  4. To ensure the integrity of all encrypted data on this system, you
#     should also encrypt swap space with 'ecryptfs-setup-swap'.
# ========================================================================

# login antes del reboot para completar la migración
sshpass -p "${PASSWD}" vagrant ssh -c  "ls -la ~/"    --plain -- -l "${CUENTA}"

# obtener la randomly generated mount passphrase y guardarla en sitio seguro
sshpass -p "${PASSWD}" vagrant ssh -c  "ecryptfs-unwrap-passphrase ~/.ecryptfs/wrapped-passphrase"    --plain -- -l "${CUENTA}"

# En mi caso me otorgo la siguiente: (esto no debe publicarse pues es un secreto)
# 1adaa5d234cb3c324539bdf73c45e477
# 1adaa5d234cb3c324539bdf73c45e477

# ahora se puede eliminar el directorio de respaldo que se habia creado automaticamente
vagrant ssh -c "sudo rm -rf /home/${CUENTA}.*"

# cruza los dedos y:
vagrant reload
sshpass -p "${PASSWD}" vagrant ssh -c  "ls -la ~/"    --plain -- -l "${CUENTA}"


```

# Notas

```text
ls -la /home/
total 28
drwxr-xr-x  7 root     root     4096 Nov 18 15:43 .
drwxr-xr-x 24 root     root     4096 Nov 18 15:38 ..
drwxr-xr-x  3 root     root     4096 Nov 18 15:43 .ecryptfs
dr-x------  2 cballard cballard 4096 Nov 18 15:43 cballard
drwx------  5 cballard cballard 4096 Nov 18 15:41 cballard.cnuHh3or
drwxr-xr-x  3 ubuntu   ubuntu   4096 Nov 18 15:30 ubuntu
drwxr-xr-x  5 vagrant  vagrant  4096 Nov 18 15:41 vagrant

# la copia de seguridad se guarda en este directorio
# hace falta 2.5 veces el espacio ocupado de /home/CUENTA 
# para poder convertir el HOME en cifrado
#
sudo ls -la /home/cballard.cnuHh3or/
total 44
drwx------   5 cballard cballard  4096 Nov 18 15:41 .
drwxr-xr-x   7 root     root      4096 Nov 18 15:43 ..
-rw-------   1 cballard cballard    59 Nov 18 15:41 .Xauthority
-rw-r--r--   1 cballard cballard   220 Apr  4  2018 .bash_logout
-rw-r--r--   1 cballard cballard  3771 Apr  4  2018 .bashrc
drwx------   2 cballard cballard  4096 Nov 18 15:41 .cache
drwx------   3 cballard cballard  4096 Nov 18 15:41 .gnupg
-rw-r--r--   1 cballard cballard   807 Apr  4  2018 .profile
drwxrwxr-x 242 cballard cballard 12288 Nov 18 15:41 archivos


sudo ls -la  /home/.ecryptfs/cballard/
total 16
drwxr-xr-x 4 cballard cballard 4096 Nov 18 15:43 .
drwxr-xr-x 3 root     root     4096 Nov 18 15:43 ..
drwx------ 5 cballard cballard 4096 Nov 18 15:41 .Private
drwx------ 2 cballard cballard 4096 Nov 18 15:43 .ecryptfs


/home/.ecryptfs/cballard/.Private/ aqui se ocupa el espacio de los archivos


sudo ls -la  /home/.ecryptfs/cballard/.ecryptfs/
total 20
drwx------ 2 cballard cballard 4096 Nov 18 15:43 .
drwxr-xr-x 4 cballard cballard 4096 Nov 18 15:43 ..
-rw------- 1 cballard cballard   15 Nov 18 15:43 Private.mnt
-rw------- 1 cballard cballard   34 Nov 18 15:43 Private.sig
-rw-r--r-- 1 cballard cballard    0 Nov 18 15:43 auto-mount
-rw-r--r-- 1 cballard cballard    0 Nov 18 15:43 auto-umount
-rw------- 1 cballard cballard   58 Nov 18 15:43 wrapped-passphrase
```






# Referencias

* https://www.lifewire.com/should-you-encrypt-home-folder-2202069

* https://www.howtogeek.com/116032/how-to-encrypt-your-home-folder-after-installing-ubuntu/

* https://www.linuxuprising.com/2018/04/how-to-encrypt-home-folder-in-ubuntu.html

* https://help.ubuntu.com/community/EncryptedPrivateDirectory

* https://vitux.com/how-to-encrypt-linux-partitions-with-veracrypt-on-ubuntu/

* para averiguar cuáles directorios copiar y no todo el `/usr/share/doc/` del host, usé:

```bash
find  /usr/share/doc/ -maxdepth 1 -type d  -wholename  "*/[a-c]*" | xargs du -sckh
find  /usr/share/doc/ -maxdepth 1 -type d  -wholename  "*/[v-z]*" | xargs du -sckh
find  /usr/share/doc/ -maxdepth 1 -type d  -wholename  "*/[a-cv-z]*" | xargs du -sckh

```
