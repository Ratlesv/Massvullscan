# MassVulScan
# Description
Script Bash qui combine la puissance du scanner Masscan pour trouver des ports ouverts, l'efficacité du scanner Nmap pour identifier les services ouverts et leur version et enfin le script NSE vulners.nse pour identifier les vulnérabilités potentielles (CVE). Un rapport HTML sera généré contenant le résultat de l'analyse ainsi qu'un fichier TXT permettant de se focaliser sur les hosts vulnérables.

![Example Menu](screenshots/Menu.PNG)

# Pré-requis
- Package xsltproc (pour la conversion d'un fichier XML vers HTML, pour le rapport final)
- Masscan, version >= 1.0.5 (https://github.com/robertdavidgraham/masscan)
- Nmap (https://nmap.org)
- Script NSE vulners.nse (https://github.com/vulnersCom/nmap-vulners)

Je vous invite à lire le fichier "requirements.txt" si vous avez des difficultés. Il vous indiquera la marche à suivre pour installer chacun des pré-requis.
# Fonctionnement (forces du script)
Le script Bash utilisera en premier Masscan pour identifier les ports ouverts, le résultat est stocké dans un fichier. Ce fichier est ensuite parcouru afin de trier et rassembler tous les ports à scanner par host, c'est là le premier avantage de ce script. Par la suite, il y aura autant de sessions Nmap qui seront lancées qu'il y aura de hosts à scanner. Deuxième force du script, toutes ces sessions fonctionneront en parallèle. Une fois les phases de scan terminées, les hosts (potentiellement) vulnérables seront affichés à l'écran et un fichier TXT sera généré avec tous les détails. Pour finir, un magnifique rapport HTML sera créé contenant tous les détails pour chacun des hosts, vulérables ou non. Ce dernier utilise une feuille de style bootstrap (https://github.com/honze-net/nmap-bootstrap-xsl).
# Comment l'utiliser ?
Il suffit d'indiquer en paramètre (-f|--include-file) le fichier contenant une liste de réseaux/hosts à scanner :
```
git clone https://github.com/choupit0/MassVulScan.git
cd MassVulScan
chmod +x MassVulScan.sh
(root user or sudo) ./MassVulScan.sh -f [input file]
```
Liste des paramètres/arguments disponibles :
```
-f [input file] = paramètre obligatoire qui contiendra la liste des réseaux/hosts à scanner
-e [exclude file] = paramètre optionnel afin d'exclure une liste de réseaux/hosts à scanner
-i (interactive mode) = paramètre optionnel pour choisir les ports à scanner et la vitesse (pkts/sec Masscan)
```
Par défaut le script ne scannera que les 1000 premiers ports TCP parmi les ports les plus courants. Vous pouvez retrouver la liste ici : /usr/local/share/nmap/nmap-services. De même, le taux ou le nombre de paquets par seconde est fixé à 5000 par défaut.

Pour le format des fichiers, vous trouverez deux exemples dans le répertoire dédié :
```
root@ubuntu:~/audit/MassVulScan# cat example/hosts.txt
# Private subnet
192.168.2.0/24
root@ubuntu:~/audit/MassVulScan# cat example/exclude.txt
# Gateway
192.168.2.254
```
# Quelques captures d'écran
![Example Masscan](screenshots/Masscan.PNG)

![Example Nmap](screenshots/Nmap.PNG)

![Example EOF](screenshots/End-of-script.PNG)

![Example Vulnerable-hosts](screenshots/Ex-vulnerable-host-found.PNG)

![Example HTML](screenshots/HTML.PNG)

# OS compatibles
Le script a seulement été testé sur Debian et Ubuntu mais devrait fonctionner sur la majorité des distributions Linux.
# Limites
Pour l'instant le script ne supporte que le scan des ports TCP.
# Remarques / Astuces
Le script est également compatible avec l'option "vuln" de Nmap permettant de rechercher davantage de vulnérabilités (les plus connues comme ms17-010, EternalBlue) en plus des CVE de vulners.com. Il vous suffit pour cela de modifier la ligne numéro 255 du script comme ceci :
```
255         nmap --max-retries 2 --max-rtt-timeout 500ms -p${port} -Pn -sT -sV -n --script vuln,vulners -oA ${nmap_temp}/${ip}_nmap-output ${ip}
```
A noter que l'avantage d'utiliser le script NSE vulners.nse et qu'il interroge systématiquement la base de données du site de vulners.com, il s'agira donc des dernières données disponibles. De même, ce dernier effectue un classement et un trie des CVE identifiés, les plus sévères en haut de la liste, ce qui est bien pratique.


