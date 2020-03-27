# Instalación de Consul

Los pasos para la instalación de Consul en local se han tomado de la propia web de Consul (https://docs.consulproject.org/docs/english-documentation/introduction/local_installation), con comentarios en algunos de los pasos.

## Requisitos:

1. Instalar Git (última versión)
2. Instalar Ruby 2.4.9 (importante que sea esta versión)
3. Instalar PostgreSql (última versión). 
   1.  ```sudo apt install -y libpq-dev postgresql-common postgresql-contrib postgresql ```
4. Cambiar la password de postgres.
   1. ```sudo -u postgres psql```
   2. ```\password postgres```
   3. Introduce el nuevo password. Por ejemplo: Postgres123
5. Instalar PgAdmin para monitorizar las bases de datos
   1. Añadir el repositorio ```wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
 sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list' ```
   2. ```sudo apt install -y pgadmin4 pgadmin4-apache2```
   3. Cuando pida el usuario, ```postgres@localhost``` y escoge un password cualquiera. Este te servirá para entrar en PgAdmin4.
   4. Entra en http://localhost/pgadmin4
   5. Agregar la base de datos. Host:localhost, user:postgres, password: Postgres123
6. Instalar Bundle 1.17.1 - ```gem install bundler:1.17.1```
7. Instalar tzinfo-data. ```gem install tzinfo-data```
8.  Instalar NodeJs. ```sudo apt install -y nodejs```
9.  Instalar ChromeDriver. 
   1. Instalar Chrome.

## Pasos:

1. Crear un usuario en la PostgreSql con permisos de administrador.
2. Ejecutar git clone https://github.com/katalkanas/consul.git
3. Ejecutar cd consul
4. Ejecutar bundle install
5. Ejecutar cp config/database.yml.example config/database.yml
   1. Modificar el fichero y cambiar el usuario y el password al creado anteriormente. 
6. Ejecutar cp config/secrets.yml.example config/secrets.yml
7. Ejecutar rake db:create
8. Ejecutar rake db:migrate
9.  Ejecutar rake db:dev_seed
10. Ejecutar RAILS_ENV=test rake db:setup
11. Ejecutar bin/rails s

## Instalar Ruby en Ubuntu

1. sudo apt install -y build-essential libssl-dev libreadline-dev zlib1g-dev
2. wget -q https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer -O- | bash
3. echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
4. rbenv init
5. echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bashrc
6. source ~/.bashrc
7. rbenv install 2.4.9
8. rbenv global 2.4.9