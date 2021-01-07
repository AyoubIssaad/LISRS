LISRS Administration NOTES

---



> **PLEASE NOTE**: This is just a <span style="color:red;">**DRAFT**</span>



### Services

##### - Search services by name

		get-Service -DisplayName "*String*"

##### - Stop service

		stop-service

##### - Get members of services

		get-services | get-member

---

### ADDS

##### How to move NTDS DB

		ntdsutil.exe

        activate instance ntds

        files

        move logs to %s

##### Trouver les partages local

		net share

##### Trouver les FSMO DS

		netdom query fsmo

---

##### Command Line

		dsadd /? : aide pour l’ajout d’objets.

        dsget /? : aide pour l’affichage d’objets.

        dsmod /? : aide pour la modification d’objets.

        dsmove /? : aide pour le déplacement d’objets.

        dsquery /? : aide pour la recherche d’objets correspondant aux critères.

        dsrm /? : aide pour la suppression d’objets.

##### ADD Command

**ADD OU**



		dsadd ou "ou=TANGER,dc=formation,dc=lan"



**ADD USER**



	dsadd user "cn=taha,ou=TANGER,dc=formation,dc=lan" -samid taha1 -upn taha1 -pwd P@ssw0rd -mustchpwd yes



*INFO*

Pour ajouter plusieur utilisateur automatiquement on peut creer un fichier .bat avec le code suivant



	dsadd user "cn=%1,ou=TANGER,dc=formation,dc=lan" -samid %1 -upn %1 -pwd P@ssw0rd -mustchpwd yes



We can execute then in command line using `file.bat username`



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



##### IN CAS of aprobation relationship problems



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



	>addusers.csv<

    DN,objectClass,sAMAccountName,userPrincipalName

    "CN=user1,OU=TANGER,DC=formation,dc=lan",user,user1,user1

    "CN=user2,OU=TANGER,DC=formation,dc=lan",user,user2,user2



**EXPORTING**



	csvde -f c:\\ALLADACCOUNTS.csv



**IMPORTING**



	csvde -i -f c:\\adusers.csv -k



### ADMINISTRATION DES GROUPES



##### Change to reaming user profiles



dsmod -profile



##### ADD group with command line

	dsadd group "D_G_Tanger, ou=tanger,dc=formation,dc=lan" -scope U -secgrp yes /* no if you want a distibution group*/



* *by default this command create the group in as Security Global, if no arguments were given*

---

##### Permissions & Autorisations

* Refus implicite VS refus explicite (Explicit means it was refused with a name) (explicit is more powerful



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
