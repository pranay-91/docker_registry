## Setting up local docker registry

Deploying a registry server on a local instance/server. This example demonstrates native basic auth and using TLS or similar to webserver with SSL.

#### Directory structure:

"auth" is  to store credentials. "certs" is for certificates using TLS and "data" is for storing images on registry.
  ```bash
    -./root
      -./auth
      -./certs
      -./data
  ```

#### Steps:
- Get a certificate
  - Create a certificate directory. Make sure to use the correct name as a CN.
  Eg: localhost or myregistrydomain.com

    ```bash
    mkdir -p certs
    openssl req -newkey rsa:4096 -nodes -sha256 -keyout certs/domain.key -x509 -days 365 -out certs/domain.crt
    ```
  - Copy the ```domain.crt``` file to ```/etc/docker/certs.d/myregistrydomain.com:5000/ca.crt```
    on every Docker host. In this example directory ```certs```.
- Basic Authentication
  - Create a auth directory and generate a user "testuser" with password "testpassword".
    ```bash
    mkdir auth
    docker run --entrypoint htpasswd registry:2 -Bbn testuser testpassword > auth/htpasswd
    ```
  - In order to add more users.
    ```bash
    docker run --entrypoint htpasswd registry:2 -Bbn testuser2 testpassword2 >> auth/htpasswd
    ```
- Run docker registry
  - Pull dokcker registry2 image
    ```bash
    docker pull registry:2
    ```
  - Using docker run
    ```bash
    docker run -d -p 6000:5000 --restart=always --name registry \
    -v `pwd`/auth:/auth \
    -e "REGISTRY_AUTH=htpasswd" \
    -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
    -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
    -v `pwd`/certs:/certs \
    -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt \
    -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key \
    registry:2
    ```
  - Using docker compose
    ```yml
    registry:
      restart: always
      image: registry:2
      ports:
        - 6000:5000
      environment:
        REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
        REGISTRY_HTTP_TLS_KEY: /certs/domain.key
        REGISTRY_AUTH: htpasswd
        REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
        REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
      volumes:
        - ./data:/var/lib/registry
        - ./certs:/certs
        - ./auth:/auth

    ```
      Note: Replace volumes ```.\data .\certs .\auth``` with your own local directory.

- Using local registry
  - Login. Enter credentials "testuser" and password "testpassword"
    ```bash
    docker login localhost:6000
    ```
  - Access local registry
    ```bash
    docker pull node
    docker tag node localhost:6000/node
    docker push localhost:6000/node
    docker pull localhost:6000/node
    ```
  - Logout from local registry
    ```bash
    docker Logout localhost:6000
    ```
  - Similarly you can login using different credentials via ```docker login ``` command.


#### Useful Links
- Deploying a registry server:
  - [https://docs.docker.com/registry/deploying/](https://docs.docker.com/registry/deploying/)
- Test an insecure registry: -
  - [https://docs.docker.com/registry/insecure/](https://docs.docker.com/registry/insecure/)
- Building private Docker registry with basic authentication by self-signed certificate, using it from OSX:
  - [https://medium.com/@deeeet/building-private-docker-registry-with-basic-authentication-with-self-signed-certificate-using-it-e6329085e612](https://medium.com/@deeeet/building-private-docker-registry-with-basic-authentication-with-self-signed-certificate-using-it-e6329085e612)
