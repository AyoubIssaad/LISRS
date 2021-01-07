LISRS Administration NOTES
---

> **PLEASE NOTE**: This is just a <span style="color:red;">**DRAFT**</span>
> ATTENTION, Il y a peut etre des fautes

### Services
##### - Search services by name
		get-Service -DisplayName "*String*"
##### - Stop service
		stop-service
##### -
		get-services | get-member
---
### ADDS
##### How to move NTDS DB
		ntdsutil.exe
        activate instance ntds
        files
        move logs to %s


##### NTDS.dit has 4 partitions
1. partition de domain (stock les informations sur les objets)
2. Config () [Site et service active directory]
3. Schema ()
4. Application ()

#### Comment sauvegarder le DLL schemas

	regsvr32 schemmgmt.dll
##### Trouver les partages local
		net share
##### identifier les DCs avec roles FSMO
		netdom query fsmo

**FSMO ROLES**
* Controleur de schéma
* maitre des noms de domaine
* controleur domaine princ
* gestionnaire du pool RID
* Maitre d'infrastructure

Dans chaque domaine on a un Gestionnaire de pool RID, un gestionnaire dedomaine principale et un maitre d'infrastructure, par contre dans une foret on a un seul controleur de schema et maitre des noms de domaines

---
##### Command Line
		dsadd /? : aide pour l’ajout d’objets.
        dsget /? : aide pour l’affichage d’objets.
        dsmod /? : aide pour la modification d’objets.
        dsmove /? : aide pour le déplacement d’objets.
        dsquery /? : aide pour la recherche d’objets correspondant aux critères.
        dsrm /? : aide pour la suppression d’objets.
##### ADD Command
**ADD OU**

		dsadd ou "ou=TANGER,dc=formation,dc=lan"

**ADD USER**

	dsadd user "cn=taha,ou=TANGER,dc=formation,dc=lan" -samid taha1 -upn taha1 -pwd P@ssw0rd -mustchpwd yes

*INFO*
Pour ajouter plusieur utilisateur automatiquement on peut creer un fichier .bat avec le code suivant

	dsadd user "cn=%1,ou=TANGER,dc=formation,dc=lan" -samid %1 -upn %1 -pwd P@ssw0rd -mustchpwd yes

et dans la ligne de commande executé `file.bat username`

**Modify Data**

	dsmod user "cn=taha,ou=TANGER,dc=formation,dc=lan" -fn rachid -ln issaad -desc " IT Admin"

**Move User**

	dsmove "cn=taha,ou=TANGER,dc=formation,dc=lan" -newparent "ou=tanger,dc=formation,dc=lan"

**Search everything**

	dsquery * "dc=formation,dc=lan"

**Search user**

	dsquery user "ou=tanger,dc=formation,dc=lan"

	dsquery user -name t* | dsget user -desc

**Search and modify**

	dsquery user -name t* | dsmod user -desc "stagiaire"

**Remove user**

	dsrm "cn=taha,ou=TANGER,dc=formation,dc=lan"

**write into a file**
add at the end of command `> filename.txt`

### POWERSHELL COMMANDS

##### Syntax
`verb-noun`

		get-childitem

		get-alias ls

		get-command *-disk // Pour trouver les commandes

		get-disk | get-memeber

		get-service | where-object {$_.status -like "*stopped*"} //get running services

		invoke-command -computername clt-1 -scriptblock {get-services -Displayname "*impress*" | start-service}

##### read secure password
		$passuser = READ-HOST -asSecureString
** **

		New-ADOrganizationalUnit -Name fes -Path "dc=formation,dc=lan

		New-ADUser -name karim -SamAccountName karim -UserPrincipalName karim -AccountPassword (convertto-securestring 	"Pass@45" -AsPlainText -Force)

		Disable-ADAccount -Identity karim
		enable-Account -identity Karim
		Unlock-ADAccount -Identity karim
		disable..
		Search-ADAccount -Accountdisabled | Enable-ADAccount
		get-aduser Karim
		remove-aduser karim

##### IN case of approbation relationship problems we can test with the following command

		test-computersecurechannel -Server "mo2dc.lenumero.local

##### ADD COMPUTER to AD

	Add-Computer -DomainName "formation.lan" -Credential formation\tech1

##### add machine to domain without being connected to it
2008 server this became possible
offline join
**COMMAND DOS**
1. *on the server*

		djoin /provision /domain formation.lan /machine clt-1 /SAVEFILE c:\OFFJoin.txt

2. Copy the file to the desired machine
3. Execute the following on the client

        djoin.exe /REQUESTDJ /LOADFILE c:\OOFJoin.txt /WINDOWSPATH %systemroot% /localos

#### Export/Import CSV
    * CSV FILES MUST CONTAIN THE HEADERS
	>addusers.csv<
    DN,objectClass,sAMAccountName,userPrincipalName
    "CN=user1,OU=TANGER,DC=formation,dc=lan",user,user1,user1
    "CN=user2,OU=TANGER,DC=formation,dc=lan",user,user2,user2

**EXPORTING**

	csvde -f c:\\ALLADACCOUNTS.csv

**IMPORTING**

	csvde -i -f c:\\adusers.csv -k

### ADMINISTRATION DES GROUPES

##### Change to Roaming user profiles
simply add `-Profile` as an argument

##### ADD group with command line
	dsadd group "D_G_Tanger, ou=tanger,dc=formation,dc=lan" -scope U -secgrp yes /* no if you want a distibution group*/

* *by default this command create the group in as Security Global, if no arguments were given*
---
##### Permissions & Autorisations
* Refus implicite VS refus explicite, explicit is more powerful

1. Write only => can't access (read) folder but can copy files into it
2. Read =>
3. Write =>

Quand un dossier et partages, les utilisateur connecté Réseau peuvent executé les droit partage sur leur dossier proprietaire, mais les droits ntfs sur les dossier existant

---
## File Servers

* **SMB** for windows machines
* **NFS** for non windows machines (linux)

### C'est quoi une imprimante
il y une difference entre:
- une imprimante
- un périphérique d'impression
- une taches d'impression
- Serveur d'impression
- pool d'impression
- service spooler


Une imprimmante ajouté par addresse IP est tjrs considérée imprimante local

en administration des imprimantes je peux gere
- les autorisation (1. imprimer, 2. gestion des documents, 3. gestion d'impression [ gerer l'imprimante ]
- priorité
- dispo
- le pool
---
### LES STRATEGIES DE GROUPES
---
On peut appliquer les GPO sur SITE|DOMAINE|OU

ORDER => MOST SPECIFIC OU first ---- Domain ---- Site Last

##### Pour forcer l'application des GPO sur une machine

	gpupdate /force

We add the ADMX files to `c:\windows\policydefinitions`

Updates are done every 90 minutes otherwise, and it's pushed by the DC, you can modify it of course by GPO itself

#### Traitement par boucle de rappel

Pour applique a un ordinateur des strategie d'utilisateur on utilise ce qu'on appele loopback processing (traitement par boucle de rappel) ( we apply it on the user section and then activate loopbackprocessing on the computer section)
Ordi -> Modele d'adminisration -> System -> Strategie de groupe -> traitement par boucle de rappelle


in cas of conflict between loopback processing and user configuration, loopback processing comes first

#### DNS, DHCP , controleur de domaine supplimentaire
##### When does replication happen between PDCs and DCSs

* **SI les DC existe dans le meme site**
	* Replication is instant

* **SI les DC existe dans des sites differents**
	* Replication is done every 180 minutes (can be edited, 15m min, 10080m max)

**When installing a sumplemental DC", this can be done using a database extracted from the PDC**

	ntdsutil
    activate instance
    ifm
    create Sysvol Full c:\NTDSDATABASE

IN case one of my DCs is down, I have to disable it in DNS to avoid round robin errors

MULTIMETRE = Replications

#### How to upgrade to a new primary DC
1. Install OS
2. Install ADDS
3. make it CDS
4. Transfer FSMO roles

On the command line, in case the PDC is unreachable

    ntdsutil
    activate instance ntds
    role
    connection
    connect to server localhost
    quit


[![infra.png](https://wiki.aubiss.xyz/uploads/images/gallery/2021-01/scaled-1680-/infra.png)](https://wiki.aubiss.xyz/uploads/images/gallery/2021-01/infra.png)

After forcig the transfer of fsmo roles, there will be some cleaning to do, on the DNS etc, the solution is to clean the metadata, can be done by deleting the dead server from the domain


### DHCP

* Creation de bail: DORA => Discovery, Offer, Request, Ack
* Renouvelement de bail: after 50% from the timer given by DHCP, we send a request to renew the timer

##### Les solutions DHCP
* augmentation d'etendu
* classeur avec basculement
* Basculement DHCP

[![dhcp.png](https://wiki.aubiss.xyz/uploads/images/gallery/2021-01/scaled-1680-/dhcp.png)](https://wiki.aubiss.xyz/uploads/images/gallery/2021-01/dhcp.png)

##### To install on multiple servers
Invoke-command -computerName PDC.S1 -ScriptBlock {get-service -Name "*spoo*"}

ipconfig /setclassid "Ethernet0" net

#### Basculement DHCP
A/A Active Active
A/S Active Standby

---

### WDS
