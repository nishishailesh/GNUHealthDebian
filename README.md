# GNUHealthDebian

# Introduction
  There are many guides to help install GNU Health in Linux. Some are outdated, Some are confusing (because author knows a lot and newbee needs help at every steps).
  
# Requirements and Assumptions
  This guide can help install GNU Health in Debian 11.  
  However, (like other guides) it can be used for other linux distributions, too.  
  Most guides create a user called gnuhealth  
  But, This guide uses bare root user
  
  
# .deb required

```
apt install postgresql-all  
apt install gunicorn  
apt install python3-pip  
apt install postgresql-client  
```

# Changes required in postgresql configuration

## Find configuration file location
```
su - postgres -c "psql -t -P format=unaligned -c 'show hba_file'"
```

Response will be:
```
/etc/postgresql/13/main/pg_hba.conf
```

## Edit postgresql configuration
```
pico /etc/postgresql/13/main/pg_hba.conf
```

At last line add  following 

```
#for GNUHealth SMPatel (A comment to remember changes in future)
local all all trust
```
## Restart service
```
service postgresql restart
```

## Create gnuhealth user in postgresql ( I donot know if this is required )
```
su - postgres -c "createuser --createdb --no-createrole --no-superuser gnuhealth"
```

## Get GNUHealth, extract it and cd to its folder 

```
wget https://ftp.gnu.org/gnu/health/gnuhealth-latest.tar.gz
tar xzf gnuhealth-latest.tar.gz
cd gnuhealth-3.8.0/
```

## Install (Some guide ask to download gnuhealth-setup file, but proper file is already present
```
./gnuhealth-setup install
```

## Change Tryton Configuration
```
cd /root/gnuhealth/tryton/server/config  
pico  trytond.conf  
```

## Changes in trytond.conf   
```
[database]  
uri = postgresql:///localhost:5432  
path = /home/gnuhealth/attach  
```
Note that there are three slashes /// in uri  

## Create database (nchshmis is my example)
```
su - postgres 
createdb nchshmis  
```

## Setup root capacity for postgre roles (needed if GNUHealth is to be installed as root)

### Create root user in postgrsql
```
cd /root/gnuhealth/tryton/server/trytond-5.0.36/bin  
su - postgres  
createuser root -s  
psql nchshmis
```

###  Use postgresql client to set root password

psql (13.3 (Debian 13.3-1))  
Type "help" for help.  

```
nchshmis=# alter role root with password 'secretpass';
ALTER ROLE
nchshmis=# \q
```
##  exit posgres user and restart postgrsql service
```
exit
service postgresql restart
```

## Initialize database ( as root user )
```
cd /root/gnuhealth/tryton/server/trytond-5.0.36/bin  
python3 ./trytond-admin --all --database=nchshmis  -c /root/gnuhealth/tryton/server/config/trytond.conf  
"admin" email for "nchshmis": xyz@abc.com
"admin" password for "nchshmis": 
"admin" password confirmation:  
```

## open ports 5432 and 8000 in firewall (if required due to drop policy in firewall)
```
root@debian:~# nft list ruleset
table inet filter {
	chain input {
		type filter hook input priority filter; policy drop;
		tcp dport 1119 accept
		tcp dport 80 accept
		tcp dport 5432 accept
		tcp dport 8000 accept
		tcp dport 3306 accept
		ct state established,related accept
		icmp type { echo-reply, echo-request } accept
	}

	chain output {
		type filter hook output priority filter; policy accept;
	}
}
```


## Start GNUHealth service
```
/root/start_gnuhealth.sh
```

## Use tryton client to access GNUHealth

#### install  tryton-client (in same or any other computer)
```
 apt install tryton-client  
 
```
#### start tryton client
application->office  

#### Login at tryton client

write host address/name    
write database name (my example: nchshmis)  
write username: admin (always, when first time)  
password is the one given at "Initialize database ( as root user )" topic above  

#### Install Modules
to be continued...


## Install web based client service
to be continued...


