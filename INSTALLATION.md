# Instalación de Consul

Los pasos para la instalación de Consul en local se han tomado de la propia web de Consul (https://docs.consulproject.org/docs/english-documentation/introduction/local_installation), con comentarios en algunos de los pasos.

## Prerrequisitos:

1. Se está utilizando Ubuntu 18.04.4 LTS
2. Actualizar el sistema 
   ```bash
   sudo apt update
   ```
3. Instalar Git
   ```bash
   sudo apt install git
   ```
4. Instalar Ruby 2.4.9 (importante que sea esta versión)
   ```bash
   sudo apt install libssl-dev autoconf bison build-essential libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm5 libgdbm-dev
   wget -q https://github.com/rbenv/rbenv-installer/raw/master/bin/rbenv-installer -O- | bash
   echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
   rbenv init
   echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.bashrc
   source ~/.bashrc
   rbenv install 2.4.9
   rbenv global 2.4.9
   ```
5. Instalar PostgreSql 
   1. Ejecuta:
      ```bash
      sudo apt install -y libpq-dev postgresql-common postgresql-contrib postgresql
      ```
   1. Crear usuario `consul` de la base de datos para la aplicación 
      ```bash
      sudo -u postgres createuser consul --createdb --superuser --pwprompt
      ```
   2. Cambiar configuración a UTF-8 
      ```bash
      sudo nano /etc/profile.d/lang.sh
      ``` 
      A continuación: 
      ```bash
      export LANGUAGE="en_US.UTF-8"
      export LANG="en_US.UTF-8"
      export LC_ALL="en_US.UTF-8"
      ``` 
   1. Reconfigura PostgreSQL
      ```bash
      sudo su - postgres
      psql
      update pg_database set datistemplate=false where datname='template1';
      drop database Template1;
      create database template1 with owner=postgres encoding='UTF-8'
      lc_collate='en_US.utf8' lc_ctype='en_US.utf8' template template0;
      update pg_database set datistemplate=true where datname='template1';
      \q
      exit
      ```
6. Cambiar la password de postgres. Introduce el nuevo password. Por ejemplo: `Postgres123`
   ```bash
   sudo -u postgres psql
   \password postgres
   ``` 
7. Instalar pgadmin4 para monitorizar las bases de datos
   1. Añadir el repositorio 
   ```bash
   wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
   sudo apt install -y pgadmin4 pgadmin4-apache2
   ```
   1. Cuando pida el usuario, introduce `postgres@localhost` y escoge un password cualquiera, por ejemplo `PgAdmin4.`. 
   2. Entra en http://localhost/pgadmin4 e introduce las credenciales anteriores `postgres@localhost` y `PgAdmin4.`.
   3. Agrega la base de datos de consul. Host:`localhost`, user:`postgres`, password: `Postgres123`.
8. Instalar Bundle 
   ```bash
   gem install bundler
   ```
9. Instalar NodeJs. 
   ```bash
   sudo apt install -y nodejs
   ``` 
10. Instalar ImageMagick 
   ```bash
   sudo apt install -y imagemagick
   ```
11. Instalar ChromeDriver. 
   ```bash
   sudo apt install -y chromium-chromedriver
   sudo ln -s /usr/lib/chromium-browser/chromedriver /usr/local/bin/
   ``` 

## Pasos:

1. Ejecuta el siguiente código 
   ```bash
   git clone https://github.com/katalkanas/consul.git
   cd consul
   bundle install
   cp config/database.yml.example config/database.yml
   cp config/secrets.yml.example config/secrets.yml
   vim config/database.yml
   ```
2. Cambia el usuario por `consul` y la contraseña por la creada anteriormente en el paso 5.2.
3. Ejecuta los siguientes comandos
   ```bash
   rake db:create
   rake db:setup
   rake db:dev_seed
   rake db:test:prepare
   ``` 
4. Comprueba que todo es correcto ejecutando el siguiente comando. Ten en cuenta que al haber muchos tests, esto puede llevar mucho tiempo (aproximadamente 1 hora). Los `.` son tests con éxito, `F` son fallidos y `*` son omitidos.
   ```bash
   bin/rspec
   ```
5. Para ejecutar la aplicación, ejecuta lo siguiente:
   ```bash
   RAILS_ENV=test rake db:setup
   bin/rails s
   ```

## Herramientas útiles

### Dbeaver

Sirve para acceder a cualquier tipo de base de datos.

```bash
sudo apt-get update
sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
sudo apt-get update
apt-cache policy docker-engine
sudo apt-get install -y docker-engine
sudo curl -o /usr/local/bin/docker-compose -L "https://github.com/docker/compose/releases/download/1.15.0/docker-compose-$(uname -s)-$(uname -m)"
sudo chmod +x /usr/local/bin/docker-compose
```

La base de datos que utiliza consul es `consul_development`

### Docker

Instalar docker

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
apt-cache policy docker-ce
sudo apt install -y docker-ce
sudo systemctl status docker
```

Ejecutar docker con el usuario local

```bash
sudo usermod -aG docker ${USER}
su - ${USER}
```

Probar que es capaz de ejecutar una imagen de docker

```bash
docker run hello-world
```

## Comandos útiles

### PostgreSql

1. Parar el servicio
   ```bash
   systemctl stop postgresql
   ``` 
2. Comenzar el servicio
   ```bash
   systemctl start postgresql
   ``` 

## Personalización

En cuanto a la personalización, debe seguirse la guía de [Consul](https://docs.consulproject.org/docs/english-documentation/customization/). Los cambios NUNCA deben hacerse sobre los ficheros propios de Consul, sino siempre dentro de las carpetas "custom" o ficheros que contengan la palabra "custom". Esto permite que se siga actualizando Consul a través de merges con el repositorio git original sin que se causen conflictos. 

En cualquier caso, a modo de resumen, Ruby on Rails guarda los principales ficheros de la aplicación en la carpeta app/assets. Dentro de esta carpeta, se encuentran las fuentes, imágenes, scripts de javascript y hojas de estilo. En principio, para cambiar el diseño, no es necesario cambiar ficheros de otra carpeta. 

Asimismo, la carpeta config/locales/custom, contiene los textos que quieren sobreescribirse con respecto al original.

### Ejemplos de personalización

#### Textos

Si queremos cambiar el texto que pone de "Menú" para acceder a la administración por "Menú de administración", debemos hacer lo siguiente:
1. Encontrar dónde se evalúa ese texto. En este caso, el texto se encuentra en `app/views/shared/_admin_login_items.html.erb` y bajo la clave `layouts.header.administration_menu`
2. Ahora es necesario buscar esa clave entre los locales. En este caso se encuentra en `config/locales/XX/general.yml` donde `XX` es el locale.
3. Para sobreescribirlo, creamos un fichero en `config/locales/custom/XX/general.yml` (si no existe la carpeta, es necesario crearla) con el siguiente contenido:
```yml
es:
  layouts:
    header:
      administration_menu: Menú de Administrador
```
4. Habría que hacer lo mismo para el resto de locales.

#### Estilos

Los estilos se deben guardan en la carpeta `app/assets/stylesheets`. Dentro de esta carpeta, existen multitud de ficheros scss que NO deben modificarse. Los únicos que deben modificarse son:
1. `_custom_settings.scss` si se quiere sobreescribir cualquier valor de `_consul_settings.scss` o de `_settings.scss`.
2. `custom.scss` si se quiere sobreescribir cualquier valor de los otros ficheros `scss`.

##### Ejemplo 1 - Cambiar el color del fondo de la cabecera

Para ello, buscamos en el fichero `_header.html.erb` que utiliza la clase `top-bar`. Esta clase se define en el fichero `layouts.scss` e indica que se utiliza el color de fondo `$brand`. Como el color es una variable, ésta estará definida en el fichero `_consul_settings.scss`. Modificamos entonces el fichero `_custom_settings.scss` y añadimos la línea:
```scss
$brand:             #f7d117;
```

##### Ejemplo 2 - Cambiar la fuente por defecto. 

En el fichero `_consul_settings.scss` se encuentra la variable `$body-font-family` que contiene el nombre de la fuente. Para cambiarla, modificamos el fichero `_custom_settings.scss` y añadimos la siguiente línea:
```scss
$body-font-family:  "Corbel", serif !important;
```

Si queremos usar una fuente personalizada, debemos primero adjuntar los ficheros de la fuente (e.g. eot, ttf, woff) y definir la fuente en la hoja de estilos. Para ello, metemos los ficheros de fuente en `app/assets/fonts/custom` y modificamos el fichero `app/assets/stylesheets/custom.scss` incluyendo la nueva fuente:
```scss
@font-face {
    font-family: "Corbel";
    src: font-url("custom/corbel.ttf") format("truetype");
    font-weight: normal;
    font-style: normal;
}
  
@font-face {
   font-family: "Corbel";
   src: font-url("custom/corbelb.ttf") format("truetype");
   font-weight: bold;
   font-style: normal;
}
```