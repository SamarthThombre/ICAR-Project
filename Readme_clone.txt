set up & install git 
set up & install docker 
install wsl ubuntu

#1 clone the repo form git to your system

git clone https://github.com/SamarthThombre/ICAR-Project.git

#2  setup docker image in your system

sudo docker compose up -d --build

#3 install important Dependencies 

sudo docker compose exec drupal composer install

## if u get this error 

The repository at "/var/www/html" does not have the correct ownership and git refuses to use it:

fatal: detected dubious ownership in repository at '/var/www/html'
To add an exception for this directory, call:

git config --global --add safe.directory /var/www/html

## then run this  

sudo docker compose exec drupal git config --global --add safe.directory /var/www/html

## and then run pervious command again 

sudo docker compose exec drupal composer install

#3 run this as well for removing errors

sudo docker compose exec drupal cp web/sites/default/default.settings.php web/sites/default/settings.php

sudo docker compose exec drupal mkdir -p web/sites/default/files

sudo docker compose exec drupal chown -R www-data:www-data web/sites/default

sudo docker compose exec drupal chmod -R 775 web/sites/default


#5 set drupal in browser

Open your web browser and go to: http://localhost:8080

You should see the Drupal installation wizard again. Just like last time, choose your language, select the "Standard" installation profile, and when you reach the Database Configuration screen, remember our crucial Docker fix:

    Click Advanced Options.

    Change the Host from localhost to db.

    Database name: drupal_db

    Database username: drupal_user

    Database password: drupal_password

