# SAMBA
## @edt Sergi Muñoz Carmona ASIX M06-ASO Curs 2018-2019

Podeu trobar les imatges docker al Dockerhub de [sergimc](https://hub.docker.com/u/sergimc/)

### Imatge:

* **sergimc/samba:18homes**  servidor samba amb ldap com a backend.
Per posar en funcionament aquest model es necessàri un server ldap+hostpam+samba

### Arquitectura:
Per a que dins d'un host es muntin els homes dels usuaris unix i ldap via samba necessitem:
  - Una xarxa propia per als conjunt de containers que utilitzem: sambanet.
  - Un servidor ldap amb els usuaris de xarxa: sergimc/ldapserver:18homes.
  - Un servidor samba que exporti els homes dels usuaris samba.
  
Configuracio d'acceś al servidor LDAP:

-Per usuaris unix:
  - Samba requereix que els usuaris unix existeixin, poden ser locals o de xarxa via LDAP.
   El servidor samba ha d'estat configurat amb nscd i nslcd per poder accedir al ldap. Per
   poder confirmar i provar que tot està ben configurat, utilitzarem les eines getent  per poder
   llistar tots els usuaris i grups de xarxa.

- Per als homes:
  - Cal que els usuaris tinguin un directori home. Els usuaris locals ja tenen un directori al crear-se,
   cal crear els directoris als usuaris LDAP i assignar-li la propietat i el grup apropiat.
   
- Per als usuari samba:
  - Cal crear els comptes d'usuari samba (han d'existir el mateix usuari unix o ldap).
   Per a cada usuari crearem el seu compte amb l'ordre *smbpasswd* i assignant-li el passwd de samba.
   Es desarà en la base de dades ldap.
  
 - El hostpam:
   - Necessitarem un hostpam ben configurat per tal d'accedir als usuaris locals i als LDAP i utilitzant pam_mount.so
   per tal de muntar dins del home dels usuaris un home de xarxa via samba. 
   Necessitarem configurar el pam_mount.conf.xml que està a la ruta: */etc/security/pam_mount.conf.xml*
   per muntar el recurs samba dels homes.
   
   
   
### Execució:

```
docker network create sambanet

docker run --rm --name ldap -h ldap --net sambanet -d sergimc/ldapserver:19smb

docker run --rm --name host -h host --net sambanet -it sergimc/hpamsmb:19smb

docker run --rm --name samba -h samba --net sambanet -it sergimc/serversmb:19smb

```

### Configuració de fitxers de samba:

/etc/samba/smb.conf
```
[global]
        workgroup = SAMBA
        security = user

        passdb backend = tdbsam

        printing = cups
        printcap name = cups
        load printers = yes
        cups options = raw

[homes]
        comment = Home Directories
        valid users = %S, %D%w%S
        browseable = No
        read only = No
        inherit acls = Yes

[printers]
        comment = All Printers
        path = /var/tmp
        printable = Yes
        create mask = 0600
        browseable = No

[print$]
        comment = Printer Drivers
        path = /var/lib/samba/drivers
        write list = @printadmin root
        force group = @printadmin
        create mask = 0664
        directory mask = 0775


