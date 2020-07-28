Una forma segura de transferir documentos a través de correo electrónico, chats, usb o cualquier dispositivo que pueda tener un intermediario en el medio es usando GPG

Que funciona en forma general de la siguiente forma

El usuarioA crea una llave publica y privada a partir del comando:
```bash
[root@clienteA ~]# gpg --gen-key
gpg (GnuPG) 2.0.22; Copyright (C) 2013 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Jose Manuel Calvo
Email address: jcalvo@redhat.com
Comment: Llave para envio y recepcion de documentos
You selected this USER-ID:
    "Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.
```

> NOTA
>
> En este punto es importante que si estamos conectados a la maquina a través de ssh, este proceso necesita el X11Forward, por lo que el ssh se debe hacer con el parámetro -X, ejemplo
>
>`[root@clienteA jmanuel$ ssh 10.96.97.162 -l root -X` y por otro lado, es requerido también que la maquina tenga instalado el paquete pinentry

```bash
      ┌─────────────────────────────────────────────────────┐
      │ Enter passphrase                                    │
      │                                                     │
      │                                                     │
      │ Passphrase ********________________________________ │
      │                                                     │
      │       <OK>                             <Cancel>     │
      └─────────────────────────────────────────────────────┘

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key B447D35A marked as ultimately trusted
public and secret key created and signed.
```

Durante este proceso de entropy el S.O esta intentando generar números aleatorios para la generación de la clave, es por esto que se recomienda que mientras se esta generando la clave, en otra terminal intentemos escribir en disco o generar algún tipo de trabajo para que estos números sean lo mas aleatorios posibles.
```bash
gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
pub   2048R/B447D35A 2020-07-28
      Key fingerprint = 0C3C 3F7A 2DB3 DDC5 A768  F74B D8BE 443B B447 D35A
uid                  Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>
sub   2048R/2718F60F 2020-07-28
```
Por ultimo la Clave del usuarioA es generada y se almacenan sobre la carpeta ~/.gnupg de cada usuario

>IMPORTANTE
>
>Se debe hacer una copia de seguridad de esta carpeta ya que aquí se contiene la clave privada (la cual no se debe compartir con NADIE)

```bash
[root@clienteA ~]# ls ~/.gnupg/
gpg.conf  private-keys-v1.d  pubring.gpg  pubring.gpg~  random_seed  secring.gpg  S.gpg-agent  trustdb.gpg
```
Una vez el usuario tenga su par de llaves importadas, ahora puede compartir su llave publica para que quien la tenga, pueda firmar los documentos, una vez firmados/encriptados dichos documentos la ÚNICA forma de desencriptarlos es con la llave privada

Para compartir la llave publica el clienteA puede ejecutar estos comandos:

1. Listar las llaves y obtener los datos como el Fingerprint o el correo 
```bash
[root@clienteA ~]# gpg --list-keys
/root/.gnupg/pubring.gpg
------------------------
pub   2048R/B447D35A 2020-07-28
uid                  Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>
sub   2048R/2718F60F 2020-07-28
```
2. Con dicha información exportar la llave privada a un archivo .key el cual debemos entregarle al clienteB para que firme los documentos
```bash
[root@clienteA ~]# gpg --output ~/jcalvo.key --armor --export B447D35A
[root@clienteA ~]# file jcalvo.key
jcalvo.key: PGP public key block
```
3. Enviar la clave al clienteB para que este haga la importacion y firme el documento

#######################

Una vez contamos con el archivo jcalvo.key en la maquina del clienteB, procedemos ha realizar los siguientes pasos:

1. Validar las llaves actuales
```bash
[jmanuel@clienteB ~]$ gpg --list-keys
gpg: /home/jmanuel/.gnupg/trustdb.gpg: trustdb created
```

2. Importar la nueva clave y validar
```bash
[jmanuel@clienteB ~]$ gpg --import jcalvo.key
gpg: key B447D35A: public key "Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
[jmanuel@clienteB ~]$ gpg --list-keys
/home/jmanuel/.gnupg/pubring.gpg
--------------------------------
pub   2048R/B447D35A 2020-07-28
uid                  Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>
sub   2048R/2718F60F 2020-07-28
```

3. Encriptar cualquier archivo, en mi caso el archivo cifrado se llamara README.txt.gpg que sera el que envie al clienteA y el origien sera mi archivo README.txt
```bash
[jmanuel@clienteB ~]$  gpg --output README.txt.gpg --encrypt --recipient B447D35A README.txt
gpg: 2718F60F: There is no assurance this key belongs to the named user

pub  2048R/2718F60F 2020-07-28 Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>
 Primary key fingerprint: 0C3C 3F7A 2DB3 DDC5 A768  F74B D8BE 443B B447 D35A
      Subkey fingerprint: CDD8 6542 C5AC 2F78 2390  E314 4E09 0865 2718 F60F

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y
```

El archivo de resultado UNICAMENTE podra ser visualizado por quien tenga la clave privada, en este caso el clienteA, el cual si queire visualizar este documento debe ejecutar el siguiente comando:
```bash
[root@clienteA ~]# gpg --decrypt README.txt.gpg > README.txt
    lqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqk
    x Please enter the passphrase to unlock the secret key for the OpenPGP certificate:     x
    x "Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>"  x
    x 2048-bit RSA key, ID 2718F60F,                                                        x
    x created 2020-07-28 (main key ID B447D35A).                                            x
    x                                                                                       x
    x                                                                                       x
    x Passphrase ********__________________________________________________________________ x
    x                                                                                       x
    x             <OK>                                                   <Cancel>           x
    mqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqj
                                      
                                      
You need a passphrase to unlock the secret key for
user: "Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>"
2048-bit RSA key, ID 2718F60F, created 2020-07-28 (main key ID B447D35A)

gpg: encrypted with 2048-bit RSA key, ID 2718F60F, created 2020-07-28
      "Jose Manuel Calvo (Llave para envio y recepcion de documentos) <jcalvo@redhat.com>"
[root@clienteA ~]# more README.txt
Archivo de Reame
```


https://www.howtogeek.com/427982/how-to-encrypt-and-decrypt-files-with-gpg-on-linux/
