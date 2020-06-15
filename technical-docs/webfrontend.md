# bwVisu web-interface and control plane

A [re-frame](https://github.com/Day8/re-frame) UI and the accompanying API


## Usage
Generally you will want to set environment variables according to `env.clj` to configure the application.  
The (maybe) simplest way to run this application is using it's docker container from the gitlab registry. Do the following to set it up.

### Download AppBundles:
Download bwVisu AppBundles containing application metadata into `appbundles/`.
```bash
git clone ssh://git@gitlab.urz.uni-heidelberg.de:urz-sb-fire/sg-dic/bwvisu/bwvisu-development/urz-sb-fire-bwvisu-apps.git appbundles
```

### Create a [docker-compose](https://github.com/docker/compose) file:
docker-compose.yml
```yaml
version: '3'

services:
  bwvisu-web:
    image: registry-gitlab.urz.uni-heidelberg.de/urz-sb-fire/sg-dic/bwvisu/bwvisu-development/urz-sb-fire-bwvisu-web
    restart: always
    volumes:
      - ./appbundles:/app/appbundles
    ports:
      - "80:80"
      - "5555:5555" # REPL server
    environment:
      - "BWVISU_JWT_SECRET=SOME_SECRET"
      - "BWVISU_ORCHESTRATION_ENGINE=infectoid"
      - "BWVISU_INFECTOID_ENDPOINT=https://infectoid.example.org"
      - "BWVISU_INFECTOID_IMPERSONATE=infectoid-user"
      - "BWVISU_LDAP_HOST=example.org"
      - "BWVISU_LDAP_PORT=389"
      - "BWVISU_LDAP_SSL=false"
      - "BWVISU_LDAP_STARTTLS=true"
      - "BWVISU_LDAP_BIND_DN=BIND-DN"
      - "BWVISU_LDAP_BIND_PASSWORD=PASSWORD"
      - "BWVISU_LDAP_USER_BASE_DN=USER-BASE-DN"
```

### Start the container
```bash
docker-compose up -d
```

### Update Appbundles
The available applications can be updated without restarting the whole bwvisu-web application:
- Apply your changes in `appbundles/`.
- Connect to REPL:
  ```bash
  nc localhost 5555
  ```
- Restart appbundles-service
  ```clojure
  (in-ns 'bwvisu-web.core) (mount/stop #'bwvisu-web.appbundle/appbundles) (mount/start)
  ```

## Minimal Demo
To play around with the user interface, debug appbundles and for demo purposes,
it may be useful to have a minimal working example of just the frontend without further
dependencies. This can be accomplished with the following `docker-compose.yml`:

```yaml
version: '3'

services:
  bwvisu-web:
    image: registry-gitlab.urz.uni-heidelberg.de/urz-sb-fire/sg-dic/bwvisu/bwvisu-development/urz-sb-fire-bwvisu-web
    volumes:
      - ./appbundles:/app/appbundles
    ports:
      - "8080:80"
      - "5555:5555" # REPL server
    environment:
      - "BWVISU_JWT_SECRET=Secret123"
      - "BWVISU_ORCHESTRATION_ENGINE=mock"
      - "BWVISU_LDAP_HOST=ldap"
      - "BWVISU_LDAP_BIND_DN=cn=admin,dc=farawaygalaxy,dc=net"
      - "BWVISU_LDAP_BIND_PASSWORD=passw0rd"
      - "BWVISU_LDAP_USER_BASE_DN=ou=users,dc=farawaygalaxy,dc=net"
    depends_on:
      - ldap

  ldap:
    image: kazhar/openldap-demo
```

To get it running, you have to:
- clone the appbundles (c.f. description above)
- run `docker-compose up -d`
- open [http://localhost:8080](http://localhost:8080)
- log in with user `asecura` and password `passw0rd`
- enjoy!

## License

Copyright © 2016-2020 University Computing Centre Heidelberg

## Exceptions

Exceptions have their message sent to the client and their stacktrace is logged to `stderr`.

