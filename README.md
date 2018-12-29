
## Get your server

Before Jumping to installition guide you must create AWS account and create lightsail virtual ubuntu server.

To make your server packages up to date you need to :

1.login to instail instanse via ssh (you must to download public key file from amazon instail  and psate the file in any folder you like and name it publickey.pem)

2. Run ```sudo apt-get update```

3. Run ```sudo apt-get upgrade```

##### I change the IP to static via AWS lightstail so the IP will not be change.



## Securing The Serve

1. login to instail instanse via ssh (you must to download public key file from amazon instail  and psate the file in any folder you like and name it publickey.pem)

2. cd to the file path and login by termenail by typing ssh -i publickey.pem ubuntu@serverIp -p 22

3. Type ```sudo nano /etc/.ssh/sshd_config```

4. Change ssh port from 22 to 2200 and restart ssh by typing ```sudo service ssh restart```

5. Allowing port 2200 for ssh on firewall by  typing sudo ufw allow 2200/tcp

6. Allowing port 80 for http on firewall by  typing sudo ufw allow www

7. Allowing port 123 for NTP on firewall by  typing sudo ufw allow 123/udp

8. Enable firewall by typing sudo ufw enable 

9. You will get this message 'Command may disrupt existing ssh connections. Proceed with operation (y|n)?' write y  
after that you will get 'Firewall is active and enabled on system startup'

## Add grader user

1. Add greader user by type ```adduser grader``` and fill his info.

2. To give him the right to be a sudoer you should type ```sudo visudo```

3. Under # User privilege specification 
copy root line and change root with grader and change last word ALL with NOPASSWD:ALL (this will give the grader the ability to do all sudo commands without providing the password each time)

4. Genreate key.pair in your local mashine by typing ```ssh-keygen``` and speivy the path for me I write it like ```/Users/'yourusername'/.ssh/grader```

5. Create .ssh folder for your grader user in your server by typing
```sudo mkdir /home/grader/.ssh```

6. Create authorized_keys file by typing ```sudo touch /home/grader/.ssh/authorized_keys```

7. Copy contintet of grader.key.pup from your local mashine to file on path ```/home/grader/authorized_keys``` on your server and save it.

8. Try to login by the new user by typing 
```ssh -i pathofyourkeypair grader@serverId -p 2200```
for me it was ssh ```-i ~/.ssh/grader.key grader@3.122.74.245 -p 2200```

## Configure the local timezone to UTC.

1. change timezone on the server to utc by writing ```sudo dpkg.reconfigure tzdata```

2. select  None of the above  after that select UTC , you will get a message that the default time zone has been changed to 'Etc/UTC'.


## Configure Postgres db
1. Install PostgreSQL db by writing ```sudo apt-get install postgresql```

2. Type ```sudo -u postgres psql``` to login to postgrasesql 

3.  create new database  by typing ```create database nasserzon;```

4. Create new user ```create user catalog with password 'catalog';```

5. Give catalog all priviliges on nasserzon db ```Grant all privileges on database NasserZon to catalog ;```

## Deploy new flask application to the server

1.  Install apache by typing ```sudo apt-get install apache2```

2. Install mod.wsgi by typing ```sudo apt-get install libapache2-mod-wsgi```

3. create new file in /etc/apache2/sites.enabled/NasserZon.conf by typing ```sudo nano /etc/apache2/sites.enabled/NasserZon.conf``` 

4. Add the following lines 
```
<VirtualHost *:80>
ServerName 3.122.74.245.xip.io
ServerAlias 3.122.74.245
ServerAdmin nasserfj58@gmail.com
WSGIScriptAlias / /var/www/NasserZon/NasserZon.wsgi
<Directory /var/www/NasserZon/>
Order allow,deny
Allow from all
</Directory>
Alias /static /var/www/NasserZon/static
<Directory /var/www/NasserZon/static/>
Order allow,deny
Allow from all
</Directory>
ErrorLog ${APACHE_LOG_DIR}/error.log
LogLevel warn
CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
5. Restart appace ``` sudo service apache2 restart ```
6. Create new dirctory ''' sudo mkdir /var/www/NasserZon '''
7. Move to the new drictory ``` cd /var/www/NasserZon ``` and create NasserZon.wsgi ```sudo nano NasserZon.wsgi ``` and past 
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/NasserZon/venv")
from myproject import app as application
application.secret_key = 'Add your secret key'
```
#### Get Flask App and add all required dirctories and files

11. Install git ```sudo apt-get install git```
14. Install python-pip ```sudo apt-get install python-pip```
15. Install virtual enviroment which will hold all NasserZon dependinces 
``` 
sudo pip install virtualenv 
````
16. Create the virtaul enviroment ```sudo virtualenv venv```
17. Activate the virtual envirmnet 
``` 
source venv/bin/activate 
```
12. Install all needed dependnsises 
```
sudo pip install sqlalchemy
sudo pip install bcrypt
sudo pip install validate_email
pip install psycopg2
sudo pip install requests
sudo pip install httplib2
sudo pip install oauth2client
sudo pip install sqlalchemy
```
13. Clone Nasserzon project fom https://github.com/nasserfj58/NasserZon.git

14. Remove all unwated files. ```sudo rm filename ```

15. Move all python files to venv dirctory by typing ```
sudo mv *.py venv``

16. Configre db on all python files by changing database engine from sqllite to postgrasesql db by change ``` sqlite:///nasserzon.db?check_same_thread=false ``` to ``` postgresql+psycopg2://catalog:catalog@localhost:5432/nasserzon ```

17. Update google api autoirzed domains and redirct and download client_scretes key as json.

18. Add client_secrets.json ``` sudo touch client_secrets.json ``` file and paste your google api clinet secret.

19. Change CLIENT_ID in ```venv/myproject.py ``` to ``` json.loads(open(r'/var/www/NasserZon/venv/client_secrets.json, 'r').read())[
    'web']['client_id'] ``` 
19. Run and seed the database by running ``` sudo python storedb_setup.py``` and ``` sudo python fillDbData.py```
20. enable NasserZon Application by typing `` sudo a2ensite NasserZon` ```

You can find the application in 
http://3.122.74.245.xip.io

### References

- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
- https://www.a2hosting.com/kb/developer-corner/postgresql/connect-to-postgresql-from-the-command-line
- https://stackoverflow.com/questions/36020374/google-permission-denied-to-generate-login-hint-for-target-domain-not-on-localh
- https://www.a2hosting.com/kb/developer-corner/postgresql/remote-postgresql-connections
- https://stackoverflow.com/questions/23327293/flask-raises-templatenotfound-error-even-though-template-file-exists
- https://developers.google.com/api-client-library/python/guide/aaa_oauth
