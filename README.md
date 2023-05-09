## Opdrachtomschrijving:
```
Almalinux > Mariadb/Mysql database server.
	software:
		epel-release
		MariaDB-server
		MariaDB-server
    		python3-PyMySQL.noarch
    		MariaDB-common

Debian 11 > Webserver (Wordpress) 
	software:
		nginx
		(Installeer de Sury repository voor Debian) https://packages.sury.org/php/
		php7.0
   		php7.0-mysql
     		php7.0-gd
    		php-ssh2
    		php7.0-mcrypt
    		php7.0-fpm
		MariaDB-client

debian 11 > Minecraft server

Doelen:
-	Volledig werkende wordpress installatie met remote database verbinding.
-	Werkende Minecraft server
```

## Toelichting voor het opbouwen van de AlmaLinux machine.
Na het opstarten van de machine en het verzorgen van een netwerkverbinding is het belangrijk deze te updaten met het volgende commando
```bash
dnf update -y
```

Daarna kun je de benodigde software installeren:
```bash
dnf install epel-release -y
dnf install MariaDB-server python3-PyMySQL
```

Zodra dit gedaan is kun je nog niet gelijk met de database werken omdat de service nog niet draait. Deze kunnen wij als volgt activeren en schikken zodat hij altijd opstart wanneer de server opstart. 
```bash
systemctl enable --now MariaDB
```

Als je alleen de service wil starten zonder dat deze altijd online komt bij het opstarten van de server kun je de start operand gebruiken, en zo ook 'stop'.
```bash
systemctl stop MariaDB
systemctl start MariaDB
```

Zo kun je ook de status van een service opvragen, om bijvoorbeeld te zien of het gecrasht is.
```bash
systemctl status MariaDB
```

Nu de service draait is het belangrijk om de onnodige zaken uit de standaard databases te verwijderen. Dit kan met het volgende commando:
```bash
mysql_secure_installation
```
Let hierbij goed op, er worden Yes/No vragen gesteld en deze kun je beantwoorden met 'y' of 'n'
De vragen kun je als volgt beantwoorden:
```bash
Switch to unix_socket authentication [Y/n]  
N

You already have your root account protected, so you can safely answer 'n'.
Change the root password? [Y/n]
N

Remove anonymous users? [Y/n]
Y

Disallow root login remotely? [Y/n] 
Y

Remove test database and access to it? [Y/n] 
Y

Reload privilege tables now? [Y/n] 
Y
```

Nu is de database server klaar om in gebruik genomen te worden, hier moet echter nog een database en user voor aangemaakt worden.
Eerst log je in op de MariaDB server dmv je de root user met bijbehorend wachtwoord
```bash
mysql -uroot -p
```
Nu krijg je een andere prompt te zien zoals:
```bash
MariaDB [(none)]> 
```
Nu kun je de database aanmaken, en vergeet de ';' niet aan het eind van de regel
```mysql
CREATE DATABASE dit_is_mijn_database;
```
Dan voor het aanmaken van de gebruiker
```mysql
CREATE USER 'dit_is_mijn_username'@'localhost';
```
En tot slot is het de bedoeling dat de nieuwe gebruiker rechten krijgt op de database.
```mysql
GRANT ALL ON dit_is_mijn_database.* TO 'dit_is_mijn_username'@'localhost';
```
Om dan uit de MariaDB terug te gaan naar de standaard terminal kun je het 'quit' commando gebruiken.


## Toelichting Debian 11 webserverDebian 11 > Webserver (Wordpress) 
Het is belangrijk bij deze machine dat het een IP adres heeft dat in hetzelfde subnet zit als de Alma machine. Zorg er ook voor dat deze elkaar kunnen bereiken met een ping.

Update de machine en installeer de software voor het hosten van een website:
```bash
apt-get update 
apt-get dist-upgrade -y
apt-get install nginx MariaDB-client -y
```

Installeer daarna de Sury repository (Deze is nodig om recente php versies te kunnen installeren) en update de apt repositories.
```bash
Voer de commando's uit op https://packages.sury.org/php/README.txt
cd
wget https://packages.sury.org/php/README.txt
chmod +x README.txt
./README.txt
apt update && apt dist-upgrade
```

Installeer de volgende php pakketten
```bash
apt-get install php7.0 php7.0-mysql php7.0-gd php-ssh2 php7.0-mcrypt php7.0-fpm
```	

Zorg ervoor dat nginx, mariadb en php-fpm opstarten bij het starten van de machine en dat dus de services beschikbaar zijn.
```bash
systemctl enable --now nginx php7.0-fpm
```

Nu kunnen wij testen of de machine in basis goed werkt. Ga naar het ip adres van de machine en als alles goed is zie je een nginx webpagina voor je.

De machine is nu gereed voor de configuratie van de web-server
   		
Ga naar de /var/www/ map
```bash
cd /var/www/default
```

Download de Wordpress tarball en pak deze uit, verwijder daarna de tarball.
```bash
wget https://wordpress.org/wordpress-6.1.tar.gz
tar -xvf wordpress-6.1.tar.gz
rm -f wordpress-6.1.tar.gz
```

Zorg ervoor dat de owner van de bestanden www-data is, anders kan de webserver de site niet tonen omdat het geen rechten heeft op de files.
```bash
chown www-data.www-data /var/www/default/*
```

Pas de wp-config.php file aan en zet de database gegevens goed. Kopieer niet een op een over, de gegevens zijn verschillend.
```
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'wpdatabase');

/** MySQL database username */
define('DB_USER', 'wpuser');

/** MySQL database password */
define('DB_PASSWORD', 'datenewachtwoordjedanmaar');

/** MySQL hostname */
define('DB_HOST', '10.0.94.152');

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');
```

Als laatste stap moet nginx nog weten hoe hij php moet aansturen en dat regelen wij als volgt in:
Ga naar de map waar de config moet staan en haal de file op met wget. Herstart daarna nginx zodat de config ingeladen wordt.
```bash
cd /etc/nginx/ && wget https://raw.githubusercontent.com/tucsonlabs/ansible-playbook-wordpress-nginx/master/roles/nginx/templates/nginx-wp-common.conf
systemctl restart nginx
```

Nu zou wordpress moeten starten zodra je het webadres volgt.    		
    		




## Minecraft server
Pak de debian11 machine en update deze.

installeer java en test of deze werkt
```bash
apt install openjdk-17-jre-headless openjdk-17-jdk-headless
javac -version
java -version
```

Resultaat moet lijken op onderstaand
```bash
root@minecraft:~# javac -version
javac 17.0.6
root@minecraft:~# java -version
openjdk version "17.0.6" 2023-01-17
OpenJDK Runtime Environment (build 17.0.6+10-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 17.0.6+10-Debian-1deb11u1, mixed mode, sharing)
```

Maak de minecraft user aan met /opt/minecraft als homefolder
```bash
useradd -m -d /opt/minecraft minecraft
```

Haal het minecraft pakket op hernoem deze en voer uit, deze gaat fout. En dat is helemaal OK. Er worden nu namelijk files opgehaald/aangemaakt die benodigd zijn voor de server.
```bash
cd /opt/minecraft
wget https://piston-data.mojang.com/v1/objects/8f3112a1049751cc472ec13e397eade5336ca7ae/server.jar
mv server.jar minecraft_server.jar
java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui
```

Zet dan de rechten goed van alle files zodat de minecraft user de eigenaar is
```bash
chmod 0700 minecraft_server.jar
chown -R minecraft.minecraft /opt/minecraft
```

Pas nu het eula.txt bestand aan en zet 'false' op 'true' en pas de inhoud van server.properties aan.
```bash
sed -i 's/false/true/g' /opt/minecraft/eula.txt
nano /opt/minecraft/eula.txt
nano /opt/minecraft/server.properties
```

Dan zijn er nog twee files die hand in hand gaan, de ops.json en de whitelist.json.
ops voor de server admins en de whitelist zodat vreemden niet inloggen.
```bash
nano /opt/minecraft/ops.json
nano /opt/minecraft/whitelist.json
```

Start daarna de server en probeer te verbinden!
```bash
java -Xmx1024M -Xms1024M -jar minecraft_server.jar nogui 
```
