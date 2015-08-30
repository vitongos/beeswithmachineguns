Bees with Machine Guns!
=======================

A utility for arming (creating) many bees (micro EC2 instances) to attack (load test) targets (web applications).

Dependencies
------------

* Python 2.6
* boto
* paramiko

Installation for developers (w/ virtualenv + virtualenvwrapper)
---------------------------------------------------------------

<pre>
git clone git://github.com/vitongos/beeswithmachineguns.git
cd beeswithmachineguns
mkvirtualenv --no-site-packages bees
easy_install pip
pip install -r requirements.txt
</pre>

Configuring AWS credentials
---------------------------

Bees uses boto to communicate with EC2 and thus supports all the same methods of storing credentials that it does.  These include declaring environment variables, machine-global configuration files, and per-user configuration files. You can read more about these options on "boto's configuration page":http://code.google.com/p/boto/wiki/BotoConfig.

At minimum, create a .boto file in your home directory with the following contents:

<pre>
[Credentials]
aws_access_key_id = <your access key>
aws_secret_access_key = <your secret key>
</pre>

The credentials used must have sufficient access to EC2.

Make sure the .boto file is only accessible by the current account:

<pre>
chmod 600 .boto
</pre>

Creación del archivo de URLs
----------------------------

Antes de realizar el test de carga, debe crearse un archivo de URLs aleatorias.
 
El objetivo es que los tests realicen peticiones sobre un conjunto de direcciones, simulando la carga real del servidor.

<pre>
./populate-urls -e endpoint-name -c 300 -l 75
</pre>

Se generará un archivo *beeswithmachineguns/urls* con 300 URLs aleatorias con el formato /endpoint-name/{hash}?os=YYYY&ip=ZZZZZZ.

El valor de 'hash' estará entre 0 y 75. Los parámetros de querystring 'os' e 'ip' no son obligatorios.

For complete options type:

<pre>
./populate-urls -h
</pre>

Uso
---

Para realizar un test de carga hay que levantar la flota de servidores y lanzar el ataque:

<pre>
./bees up -s 4 -g load-test-sg -k my-keypair -i ami-1ccae774 -z us-east-1a -l ec2_user
./bees attack -n 10000 -c 250 -u http://www.your-domain.com/
./bees down
</pre>

Con 'bees up' se iniciarán 4 servers en el grupo de seguridad 'load-test-sg' usando el keypair 'my-keypair', que debe estar almacenada en ~/.ssh/my-keypair.pem.

El tipo de instancia será 'ami-1ccae774', la zona de disponibilidad 'us-east-1a' y el usuario para la conexión SSH 'ec2_user'.

Al ejecutar 'bees attack' los 4 servidores lanzarán 10000 requests, 250 concurrentes, sobre las urls aleatorias http://www.your-domain.com/endpoint-name/{hash}?os=YYYY&ip=ZZZZZZ.

Por último, 'bees down' termina las instancias. **Importante: de no terminar las instancias de EC2, Amazon continuará facturando por ellas**.

For complete options type:

<pre>
./bees -h
</pre>

The caveat! (PLEASE READ)
-------------------------

(The following was cribbed from our "original blog post about the bees":http://blog.apps.chicagotribune.com/2010/07/08/bees-with-machine-guns/.)

If you decide to use the Bees, please keep in mind the following important caveat: they are, more-or-less a distributed denial-of-service attack in a fancy package and, therefore, if you point them at any server you don’t own you will behaving *unethically*, have your Amazon Web Services account *locked-out*, and be *liable* in a court of law for any downtime you cause.

You have been warned.

Credits
-------

The bees are a creation of the News Applications team at the Chicago Tribune--visit "our blog":http://apps.chicagotribune.com/ and read "our original post about the project":http://blog.apps.chicagotribune.com/2010/07/%2008/bees-with-machine-guns/.

Initial refactoring code and inspiration from "Jeff Larson":http://github.com/thejefflarson.

Thanks to everyone who reported bugs against the alpha release.

License
-------

MIT.
