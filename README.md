# hashicorp
Q1 : Use an official Go base image (e.g., golang:1.22-alpine) for the build stage.
Ans : Followed below steps. 
Step 1 : Downloaded dockerhub and installed it on EC2 machine (VMA and VMB). 
  [+] https://docs.docker.com/engine/install/rhel/
Step 2 : Installed go 
  [+] sudo yum install golang
  [+] official documentation https://go.dev/doc/install
Step 3 : Created go module "docker-go-app" - VMB
  $ go mod init docker-go-app 
  $ touch main.go
  $ vi main.go 

Step 4 : Added the code to listen on port passed as LISTEN_PORT to the main.go. By default code was bind to localhost, modified it to get the ListenAndServe bind to all address.
  [+] https://pkg.go.dev/net/http#ListenAndServe 

Step 5 : Tested app locally. 
  $ go run main.go         

Step 5a : Created dockerfile to dockerize the application (using golang:1.24 image)

FROM golang:1.24

WORKDIR /app

COPY go.mod .
COPY main.go .

RUN go get
RUN go build -o bin .

ENTRYPOINT [ "/app/bin" ]

         
Step 5b : Created the container image.
  $ docker build . -t hashicorp-go-container:latest 


Step 6a : Created a dockerfile to dockerize the application using multi-stage builds (alpine)


FROM golang:1.24-alpine AS build
WORKDIR /app
COPY main.go go.mod ./
RUN go mod download
RUN go build -o /app/bin .

FROM alpine:latest
WORKDIR /app
COPY --from=build /app/bin .
ENTRYPOINT ["/app/bin"]


Step 6b : Creaate the containser image
  $ docker build . -t hashicorp-go-container-multistage:latest


Q2. Create the container onb VMB from image  hashicorp-go-container-multistage
Ans. 
Q2. Create the container go-app-multistage on VMB (hashicorp-go-container-multistage)
Ans.


Step 2.1 : Created docker container VMB based on docker image nhashicorp-go-container-multistage

 $ docker run -d  --name=go-app-multistage --restart always  -e LISTEN_PORT="8080" -p 8080:8080 hashicorp-go-container-multistage

  [root@ip-10-0-4-55 hashicorp]# docker run -d  --name=go-app-multistage --restart always  -e LISTEN_PORT="8080" -p 8080:8080 hashicorp-go-container-multistage
  cfaf9e50da634b661c8f5a501770ffc382b2d3ee3882da0556907ff2bcefcfa1
  [root@ip-10-0-4-55 hashicorp]# docker ps
  CONTAINER ID   IMAGE                               COMMAND      CREATED         STATUS         PORTS                                         NAMES
  cfaf9e50da63   hashicorp-go-container-multistage   "/app/bin"   3 seconds ago   Up 3 seconds   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   go-app-multistage
  [root@ip-10-0-4-55 hashicorp]# docker logs cfaf9e50da63
  2025/07/30 07:39:29 Server listening on http://:8080
  [root@ip-10-0-4-55 hashicorp]#


Step 2.2 : Access application running on VMB (container hashicorp-go-container-multistage) from VMA

  [ec2-user@ip-10-0-4-155 ~]$ curl -v http://10.0.4.55:8080
  *   Trying 10.0.4.55:8080...
  * Connected to 10.0.4.55 (10.0.4.55) port 8080
  * using HTTP/1.x
  > GET / HTTP/1.1
  > Host: 10.0.4.55:8080
  > User-Agent: curl/8.12.1
  > Accept: */*
  >
  * Request completely sent off
  < HTTP/1.1 200 OK
  < Date: Wed, 30 Jul 2025 07:47:17 GMT
  < Content-Length: 30
  < Content-Type: text/plain; charset=utf-8
  <
  cfaf9e50da63 : Hello, World!
  * Connection #0 to host 10.0.4.55 left intact
  [ec2-user@ip-10-0-4-155 ~]$

Step 2.3 : Logs of the application when curl was run from VMA

  [root@ip-10-0-4-55 ~]# docker logs cfaf9e50da63
  2025/07/30 07:39:29 Server listening on http://:8080
  [root@ip-10-0-4-55 ~]#


Q3. Configure Proxy that will request application running on VMB
A

Step 3.1 : Create Proxy configuration file (nginx.conf)
 upstream backend {
     server 10.0.4.55:8080;
 }

 server {
     listen 8080;

     server_name VMA;

     location / {
         proxy_pass http://backend;
     }
 }

Step 3.2 : Run proxy as a container in VMA

$ docker run -d -p 8080:8080 --restart always --name nginx-VMA -v /root/hashicorp/nginx.conf:/etc/nginx/conf.d/default.conf nginx

  [root@ip-10-0-4-155 hashicorp]# docker run -d -p 8080:8080 --restart always --name nginx-VMA -v /root/hashicorp/nginx.conf:/etc/nginx/conf.d/default.conf nginx
  Unable to find image 'nginx:latest' locally
  latest: Pulling from library/nginx
  59e22667830b: Pull complete
  140da4f89dcb: Pull complete
  96e47e70491e: Pull complete
  2ef442a3816e: Pull complete
  4b1e45a9989f: Pull complete
  1d9f51194194: Pull complete
  f30ffbee4c54: Pull complete
  Digest: sha256:84ec966e61a8c7846f509da7eb081c55c1d56817448728924a87ab32f12a72fb
  Status: Downloaded newer image for nginx:latest
  0f7390ca3688cf4280adfdaf94faee73cbd3c1e17af2a2c3bdf9da1d228fa3ee
  [root@ip-10-0-4-155 hashicorp]#
  [root@ip-10-0-4-155 hashicorp]#
  [root@ip-10-0-4-155 hashicorp]# docker ps -a
  CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                                 NAMES
  0f7390ca3688   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   80/tcp, 0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   nginx-VMA
  [root@ip-10-0-4-155 hashicorp]# docker logs 0f7390ca3688
  /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
  /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
  10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
  10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
  /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
  /docker-entrypoint.sh: Configuration complete; ready for start up
  2025/07/30 07:56:29 [notice] 1#1: using the "epoll" event method
  2025/07/30 07:56:29 [notice] 1#1: nginx/1.29.0
  2025/07/30 07:56:29 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14+deb12u1)
  2025/07/30 07:56:29 [notice] 1#1: OS: Linux 6.12.0-55.22.1.el10_0.x86_64
  2025/07/30 07:56:29 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
  2025/07/30 07:56:29 [notice] 1#1: start worker processes
  2025/07/30 07:56:29 [notice] 1#1: start worker process 28
  2025/07/30 07:56:29 [notice] 1#1: start worker process 29
  [root@ip-10-0-4-155 hashicorp]#


Step 3.3 Access the application via proxy host from outside VMA


PS C:\Users\nba> curl http://ec2-13-239-227-254.ap-southeast-2.compute.amazonaws.com:8080/


StatusCode        : 200
StatusDescription : OK
Content           : cfaf9e50da63 : Hello, World!

RawContent        : HTTP/1.1 200 OK
                    Connection: keep-alive
                    Content-Length: 30
                    Content-Type: text/plain; charset=utf-8
                    Date: Wed, 30 Jul 2025 07:59:18 GMT
                    Server: nginx/1.29.0

                    cfaf9e50da63 : Hello, World!

Forms             : {}
Headers           : {[Connection, keep-alive], [Content-Length, 30], [Content-Type, text/plain; charset=utf-8], [Date, Wed, 30 Jul 2025 07:59:18 GMT]...}
Images            : {}
InputFields       : {}
Links             : {}
ParsedHtml        : mshtml.HTMLDocumentClass
RawContentLength  : 30



PS C:\Users\nba>



Q4. Basic understanding of TLS, configure nginx over TLS

Step4.1 Create TLS private key, CSR and certificate.

$ openssl genrsa -des3 -out priv.key 2048
$ openssl req -key priv.key -new -out hashicorp.csr
$ openssl x509 -signkey priv.key -in hashicorp.csr -req -days 365 -out hashicorp.crt


  [root@ip-10-0-4-55 hashicorp]#
  [root@ip-10-0-4-55 hashicorp]#
  [root@ip-10-0-4-55 hashicorp]# openssl genrsa -des3 -out priv.key 2048
  Enter PEM pass phrase:
  Verifying - Enter PEM pass phrase:
  [root@ip-10-0-4-55 hashicorp]# openssl req -key priv.key -new -out hashicorp.csr
  Enter pass phrase for priv.key:
  You are about to be asked to enter information that will be incorporated
  into your certificate request.
  What you are about to enter is what is called a Distinguished Name or a DN.
  There are quite a few fields but you can leave some blank
  For some fields there will be a default value,
  If you enter '.', the field will be left blank.
  -----
  Country Name (2 letter code) [XX]:AU
  State or Province Name (full name) []:VIC
  Locality Name (eg, city) [Default City]:MEL
  Organization Name (eg, company) [Default Company Ltd]:TEST
  Organizational Unit Name (eg, section) []:
  Common Name (eg, your name or your server's hostname) []:
  Email Address []:

  Please enter the following 'extra' attributes
  to be sent with your certificate request
  A challenge password []:
  An optional company name []:
  [root@ip-10-0-4-55 hashicorp]# openssl x509 -signkey priv.key -in hashicorp.csr -req -days 365 -out hashicorp.crt
  Enter pass phrase for priv.key:
  Certificate request self-signature ok
  subject=C=AU, ST=VIC, L=MEL, O=TEST

[root@ip-10-0-4-155 assessment]# cat nginx-ssl.conf
upstream backend {
    server 10.0.4.155:8080;
    # Add more backend servers as needed
}

server {
    listen 8443 ssl;

    server_name VMASSL;

    location / {
        proxy_pass http://backend;
    }
   ssl_certificate /etc/nginx/conf.d/hashicorp.crt;
   ssl_certificate_key /etc/nginx/conf.d/priv.key;
   ssl_password_file /etc/nginx/conf.d/ssl_password_file.txt;
}


Step 4.2 Create nginx container(VMA) to listen over SSL/TLS

$docker run -d -p 8443:8443 --restart always --name nginx-ssl-VMA -v /root/hashicorp/nginx-ssl.conf:/etc/nginx/conf.d/default.conf -v /root/hashicorp/hashicorp.crt:/etc/nginx/conf.d/hashicorp.crt -v  /root/hashicorp/priv.key:/etc/nginx/conf.d/priv.key -v /root/hashicorp/ssl_password_file.txt:/etc/nginx/conf.d/ssl_password_file.txt nginx

Step 4.3 Access the application via proxy host from outside VMA - Over SSL

  [root@ip-10-0-4-155 hashicorp]# docker run -d  --restart always -p 8443:8443 --name nginx-ssl1-VMA -v /root/hashicorp/nginx-ssl.conf:/etc/nginx/conf.d/default.conf -v /root/hashicorp/hashicorp.crt:/etc/nginx/conf.d/hashicorp.crt -v  /root/hashicorp/priv.key:/etc/nginx/conf.d/priv.key -v /root/hashicorp/ssl_password_file.txt:/etc/nginx/conf.d/ssl_password_file.txt nginx
  406750a616a2d37c90d74a4324232310b461c042baa73186abde7c0ea16878f1
  [root@ip-10-0-4-155 hashicorp]#
  [root@ip-10-0-4-155 hashicorp]#
  [root@ip-10-0-4-155 hashicorp]# docker ps -a
  CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                                 NAMES
  406750a616a2   nginx     "/docker-entrypoint.…"   5 seconds ago    Up 4 seconds    80/tcp, 0.0.0.0:8443->8443/tcp, [::]:8443->8443/tcp   nginx-ssl1-VMA
  0f7390ca3688   nginx     "/docker-entrypoint.…"   30 minutes ago   Up 30 minutes   80/tcp, 0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   nginx-VMA
  [root@ip-10-0-4-155 hashicorp]# docker logs 406750a616a2
  /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
  /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
  10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
  10-listen-on-ipv6-by-default.sh: info: /etc/nginx/conf.d/default.conf differs from the packaged version
  /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
  /docker-entrypoint.sh: Configuration complete; ready for start up
  2025/07/30 08:26:32 [notice] 1#1: using the "epoll" event method
  2025/07/30 08:26:32 [notice] 1#1: nginx/1.29.0
  2025/07/30 08:26:32 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14+deb12u1)
  2025/07/30 08:26:32 [notice] 1#1: OS: Linux 6.12.0-55.22.1.el10_0.x86_64
  2025/07/30 08:26:32 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
  2025/07/30 08:26:32 [notice] 1#1: start worker processes
  2025/07/30 08:26:32 [notice] 1#1: start worker process 29
  2025/07/30 08:26:32 [notice] 1#1: start worker process 30
  [root@ip-10-0-4-155 hashicorp]#
  [root@ip-10-0-4-155 hashicorp]# curl --insecure https://10.0.4.155:8443/
  cfaf9e50da63 : Hello, World!
  [root@ip-10-0-4-155 hashicorp]#
  [root@ip-10-0-4-155 hashicorp]#


Note : On laptop curl was not working with self-signed cert (Powershell Invoke-WebRequest is of old version and do not support self-signed cert), have accessed applicatoin running from VMB using VMA IP address.

  [root@ip-10-0-4-55 ~]#  curl --insecure https://10.0.4.155:8443/
  cfaf9e50da63 : Hello, World!
  [root@ip-10-0-4-55 ~]#

