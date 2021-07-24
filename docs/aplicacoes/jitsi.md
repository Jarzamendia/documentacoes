# Jitsi Meet

## Githubs

- https://github.com/jitsi/jitsi-meet
- https://github.com/jitsi/docker-jitsi-meet

## Como iniciar?

```
# Clonar o ambiente, nesse caso usaremos o oficial.
git clone https://github.com/jitsi/docker-jitsi-meet.git

cd docker-jitsi-meet

# Criaremos o arquivo de variaveis de ambiente.
mv env.example .env

chmod +x gen-passwords.sh

# Gerar as variaveis de segurança.
./gen-passwords.sh

```

Após isto, edite as variaveis necessarias no arquivo .env. As mais importantes são:

```
# Local onde os arquivos de configuração serão salvos (incluindo certificados)
CONFIG=./jitsi-meet-cfg

# Portas do frontend
HTTP_PORT=80
HTTPS_PORT=8443

# URL que será usada para acessar o Meet, é obrigatória!
PUBLIC_URL=http://jitsi.localhost

# IP do host Docker
DOCKER_HOST_ADDRESS=10.0.0.1
```

É importante notar que os endereços marcados como .meet.jitsi são de uso interno e podem/devem ser mantidos como estão.

Para iniciar usando Docker-compose:

```
docker-compose up -d
```

## Etherpad

O Etherpad é um gestor de arquivos compartilhados para uso em grupo que pode ser utilizado dentro de uma reunião do meet.

Para utilizá lo, adicione o seguinte serviço no docker-compose.yml:

```
    etherpad:
        image: etherpad/etherpad:1.8.6
        restart: ${RESTART_POLICY}
        environment:
            - TITLE=${ETHERPAD_TITLE}
            - DEFAULT_PAD_TEXT=${ETHERPAD_DEFAULT_PAD_TEXT}
            - SKIN_NAME=${ETHERPAD_SKIN_NAME}
            - SKIN_VARIANTS=${ETHERPAD_SKIN_VARIANTS}
            - SUPPRESS_ERRORS_IN_PAD_TEXT
        networks:
            meet.jitsi:
                aliases:
                    - etherpad.meet.jitsi
```

E adicione as seguintes variáveis de ambiente no arquivo .env.

```
#
# Etherpad integration (for document sharing)
#

# Set etherpad-lite URL in docker local network (uncomment to enable)
ETHERPAD_URL_BASE=http://etherpad.meet.jitsi:9001

# Set etherpad-lite public URL (uncomment to enable)
ETHERPAD_PUBLIC_URL=http://jitsi.localhost/etherpad/p/

# Name your etherpad instance!
ETHERPAD_TITLE=Video Chat

# The default text of a pad
ETHERPAD_DEFAULT_PAD_TEXT=Welcome to Web Chat!\n\n

# Name of the skin for etherpad
ETHERPAD_SKIN_NAME=colibris

# Skin variants for etherpad
ETHERPAD_SKIN_VARIANTS=super-light-toolbar super-light-editor light-background full-width-editor

# To remove a warning in pad texts
SUPPRESS_ERRORS_IN_PAD_TEXT=true
```

É importante notar que o ETHERPAD_URL_BASE refere-se ao endereço interno do Etherpad, o Nginx do frontend criará uma rota apontando o endereço de /etherpad para a URL interna. Da mesma forma é importante manter a ETHERPAD_PUBLIC_URL com o mesmo host do seu ambiente, para evitar erros de CORS.

## Autenticação

É possível usar o meet com autenticação, incluindo em modo híbrido com usuários autenticados e não autenticados, segue algumas coisas interessantes:
 
- Quando definimos a autenticação, apenas usuários logados podem criar salas.
- Caso definamos ENABLE_GUESTS como true, usuários deslogados poderão entrar em salas já criadas.
- Usuários deslogados têm o terrível hábito de não colocarem um nome no seu perfil. As opções ENABLE_WELCOME_PAGE, ENABLE_PREJOIN_PAGE, ENABLE_CLOSE_PAGE e ENABLE_REQUIRE_DISPLAY_NAME cuidam muito bem disso.

### LDAP

Está é uma configuração padrão de autenticação por LDAP para um servidor de Active Directory com Windows.

```
ENABLE_AUTH=1
ENABLE_GUESTS=1
AUTH_TYPE=ldap
LDAP_URL=ldap://adserver:389
LDAP_BASE=OU=users,DC=contoso,DC=com
LDAP_BINDDN=CN=user,OU=users,DC=contoso,DC=com
LDAP_BINDPW=password
LDAP_FILTER=(samaccountname=%u)
LDAP_AUTH_METHOD=bind
LDAP_USE_TLS=0
```

### Keycloak

Vamos tentar usar o Keycloak como metodo de autenticação.

## Métricas

Podemos verificar as metricas de uso do JVB adicionando os seguintes endpoints na variavel JVB_ENABLE_APIS:

```
JVB_ENABLE_APIS=rest,colibri,stats
```

Após isto, podemos acessar o container do JVB na porta 8080 (http://jvb:8080/colibri/stats); é importante notar que ele gera as metricas em formato Json, isso pode ser util em alguns casos, porém em geral gostariamos que ele nos entregasse algo no formato do Prometheus. Para isso podemos usar um workaround interessante:

### Prometheus

Podemos usar um [container utilitario](https://github.com/systemli/prometheus-jitsi-meet-exporter) para traduzir as metricas:

```
    exporter:
        image: systemli/prometheus-jitsi-meet-exporter:latest
        restart: ${RESTART_POLICY}
        command: -videobridge-url http://jvb:8080/colibri/stats
        ports:
            - 9888:9888
        networks:
            meet.jitsi:
```