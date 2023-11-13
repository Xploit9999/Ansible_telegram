## Descripcion del playbook

* Este playbook fue pensado para reportar diariamente con eventos relacionados con kernel panic en los servidores y reportarlo por Telegram. 

### Requerimiento antes de usar el playbook

* Crear un bot en telegram y obtener informacion del chat room id o grupo id, visite el siguiente enlace: [Link](https://core.telegram.org/bots/api) para obtener mas informacion ya que es necesario para poder usar el modulo de Telegram en ansible.

### Modo de Uso del Playbook

* Tener creado nuestro inventario con los servidores o servidor donde se van a extraer los reportes. En este proyecto deje a modo de ejemplo un archivo llamado inventario.ini como referencia.

* Tener presente que la informacion solo la obtiene del /var/log/messages que esta pensado para distribuciones derivadas de RedHat, declarada en la variable **messages**

```vim
vars:
    messages: /var/log/messages
```
- Puedes reemplazar el contenido de la variable dependiendo de donde se aloje o se configure los mensajes relacionados a eventos criticos como los del kernel panic de acuerdo a la distribucion de su sistema operativo.

* En el directorio handlers se encuentra el archivo main.yml, en el es donde se invoca al modulo de telegram una vez se encuentre el evento del kernel panic en el archivo log declarado en la variable messages. Por lo tanto la informacion sobre el chat id, token, y el mensaje en el cual notificara atraves de telegram se modificaran en esa ruta.

```vim
# _Contenido del archivo handlers/main.yml_
---
- name: Enviar notificacion al telegram sobre el kernel panic sucedido en el servidor.
  telegram:
    token: "{{ bot_token }}"
    api_args:
      chat_id: "{{ bot_chat_id }}"
      text: "*!!![ CRITICAL ]!!!* El servidor: *{{ ansible_hostname }}* tuvo un __kernel panic__ hace *{{ tiempo }}* minutos."
      parse_mode: 'markdown'
```
* En el parametro: 
```vim
      text: "*!!![ CRITICAL ]!!!* El servidor: *{{ ansible_hostname }}* tuvo un __kernel panic__ hace *{{ tiempo }}* minutos."
```

- Es donde se parametriza el mensaje que va a notificar el evento sobre el kernel panic, en este caso, yo lo parametrice para que me reporte el nombre del servidor en el que se hallo el evento, y el tiempo del servidor en el que se encuentra actualmente, ya que si hubo un panic lo mas razonable es que el servidor tuvo un reinicio, por lo tanto es ideal conocer su tiempo arriba para poder actuar los mas pronto posible para restablecer los servicios alojados en el servidor, y poder diagnosticar oportunamente.


* Los datos sensibles como el token y el chat id en este caso del proyecto lo proyecte para que se use como archivo cifrado en la carpeta vars y el archivo secrets.yml. Se tiene pensado asi para proteger estos datos, y asi mismo se invoquen como extra-vars, ya que como se quiere dejar este proceso de forma programada para que todos los dias se ejecute, en el job quedaria de la siguiente forma:

```vim
# *Ejemplo del contenido del archivo vars/secrets.yml*
---
ansible_ssh_user: administrador     # Usuario con acceso a las maquinas
ansible_ssh_pass: "123456"       # Password del usuario con acceso a las maquinas
ansible_become_pass: "123456"    # Password para la escalacion de privilegios.
bot_token: "<token-134872829>" # El token que brinda telegram para el consumo de su API.
bot_chat_id: "<chat-1239271>"  # El numero de identificacion del chat donde el bot estara enviando las notificaciones.
```

- Como el contenido que hay en el vars/secrets.yml es sensible, lo mejor es usar el ansible-vault para que este lo cifre.

```vim
$ ansible-vault encrypt vars/secrets.yml
$
$ cat vars/secrets.yml
$ANSIBLE_VAULT;1.1;AES256
63363033306666373137343564333661343637353638326466656438636161653833366264303861
3936633962643635353763653234633866666234636364610a373433366338613031366132303533
37353866373032306239616531393434336533363437306534333861353931363532313136643338
6165383938623535630a613933626335313062343264623761386666316236633463666662366536
62393337633032623830396462323964643137373737653637376563653834626536386464306662
30663832316365623238656562643035333563623065613633313331646638613464613831376332
61303935333736626664363065306432613539346364636562666534316466336662616465626536
32626566363265383064396633303039613265613638343331373533373139396631653932303631
66323137323861303161613733633536656561386237376163366163333838333962643863643535
37343532666433363934366536363737346665366431353464666362336235323965626336303362
31663238616531303038626236303134666261393761346231316130313161653638613436626466
62363434333433306565366163636135346362323838386438383734363635323361313265643234
31626464313637393338316237303238373333306134353534623739393438613432
```

* Para su ejecucion de forma inatendida si lo queremos montar sobre una tarea cron, podemos hacer uso de esta sintaxis:

```vim
ansible-playbook -i inventario.ini envia_reportes.yml -e @vars/secrets.yml --vault-password-file /ruta/vault/passwd-vault.prv
```
