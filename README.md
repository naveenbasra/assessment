# hashicorp
Q1 : Use an official Go base image (e.g., golang:1.22-alpine) for the build stage.
Ans : Followed below steps. 
Step 1 : Downloaded dockerhub and installed it on EC2 machine. 
  [+] https://docs.docker.com/engine/install/rhel/
Step 2 : Installed go 
  [+] sudo yum install golang
  [+] official documentation https://go.dev/doc/install
Step 3 : Created go module "docker-go-app"
  $ go mod init docker-go-app 
  $ touch main.go
  $ vi main.go 

Step 4 : Added the code to listen on port passed as LISTEN_PORT to the main.go. By default code was bind to localhost, modified it to get the ListenAndServe bind to all address.
  [+] https://pkg.go.dev/net/http#ListenAndServe 

Step 5 : Tested app locally. 
  $ go run main.go         

Step 5a : Created dockerfile to dockerize the application (using golang:1.24 image)
         
Step 5b : Created the container image.
  $ docker build . -t hashicorp-go-container:latest 

Step 5c : Created docker VMA based on docker image hashicorp-go-container:latest

  [root@ip-10-0-4-55 hashicorp]# docker run -d --name=VMA  -e LISTEN_PORT="8080" hashicorp-go-container:latest
  [root@ip-10-0-4-55 hashicorp]# docker ps
  CONTAINER ID   IMAGE                           COMMAND      CREATED          STATUS          PORTS     NAMES
  c0e787184f8f   hashicorp-go-container:latest   "/app/bin"   11 seconds ago   Up 10 seconds             VMA
  [root@ip-10-0-4-55 hashicorp]# docker logs c0e787184f8f
  2025/07/29 07:18:59 Server listening on http://:8080
  [root@ip-10-0-4-55 hashicorp]#

Step 6a : Created a dockerfile to dockerize the application using multi-stage builds (alpine)

Step 6b : Creaate the containser image
  $

Step 6c : Create docker VMA1.1based on docker image hashicorp-go-container-vanila:latest

  [root@ip-10-0-4-55 part1.2]# docker run -d --name=VMA1.1  -e LISTEN_PORT="8080" hashicorp-go-container-vanila:latest
  41e3029a6a8ad59c54cf828d0be91f03f389806f4d6f540b5fa3f9adf09e491a
  [root@ip-10-0-4-55 part1.2]#
  [root@ip-10-0-4-55 part1.2]#
  [root@ip-10-0-4-55 part1.2]# docker ps
  CONTAINER ID   IMAGE                                  COMMAND      CREATED         STATUS         PORTS     NAMES
  41e3029a6a8a   hashicorp-go-container-vanila:latest   "/app/bin"   3 seconds ago   Up 2 seconds             VMA1.1
  c0e787184f8f   hashicorp-go-container:latest          "/app/bin"   3 hours ago     Up 3 hours               VMA
  [root@ip-10-0-4-55 part1.2]# docker logs 41e3029a6a8a
  2025/07/29 10:47:46 Server listening on http://:8080
  [root@ip-10-0-4-55 part1.2]#

Q2. Create the container image VMA (nginx) and VMB1 (hashicorp-go-container), VMB2 (hashicorp-go-container-vanila)
Ans. 

Step 2.1 : Created docker container VMA based on docker image nginx

  [root@ip-10-0-4-55 hashicorp]# docker run -d --name=VMA nginx
  bf9d6e3c9987a936f2e812633192ca978b88bb415f8a96d46b4989165917391d
  [root@ip-10-0-4-55 hashicorp]# docker ps -a
  CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
  bf9d6e3c9987   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 4 seconds   80/tcp    VMA
  [root@ip-10-0-4-55 hashicorp]# docker logs bf9d6e3c9987
  /docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
  /docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
  10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
  10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
  /docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
  /docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
  /docker-entrypoint.sh: Configuration complete; ready for start up
  2025/07/29 11:05:31 [notice] 1#1: using the "epoll" event method
  2025/07/29 11:05:31 [notice] 1#1: nginx/1.29.0
  2025/07/29 11:05:31 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14+deb12u1)
  2025/07/29 11:05:31 [notice] 1#1: OS: Linux 6.12.0-55.22.1.el10_0.x86_64
  2025/07/29 11:05:31 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1073741816:1073741816
  2025/07/29 11:05:31 [notice] 1#1: start worker processes
  2025/07/29 11:05:31 [notice] 1#1: start worker process 29
  2025/07/29 11:05:31 [notice] 1#1: start worker process 30
  [root@ip-10-0-4-55 hashicorp]#

Step 2.2 : Create docker container VMB1 based on docker image hashicorp-go-container-vanila:latest


  [root@ip-10-0-4-55 hashicorp]# docker run -d --name=VMB1 --restart always  -e LISTEN_PORT="8080" hashicorp-go-container-vanila:latest
  44085ff394723d85b431bf860f2d3907b0b9428ea07108991af351fa18bad275
  [root@ip-10-0-4-55 hashicorp]#
  [root@ip-10-0-4-55 hashicorp]# docker ps -a
  CONTAINER ID   IMAGE                                  COMMAND                  CREATED         STATUS         PORTS     NAMES
  44085ff39472   hashicorp-go-container-vanila:latest   "/app/bin"               5 seconds ago   Up 5 seconds             VMB1
  bf9d6e3c9987   nginx                                  "/docker-entrypoint.…"   7 minutes ago   Up 7 minutes   80/tcp    VMA
  [root@ip-10-0-4-55 hashicorp]# docker logs 44085ff39472
  2025/07/29 11:12:35 Server listening on http://:8080
  [root@ip-10-0-4-55 hashicorp]#

Step 2.2 : Created docker container VMB2 based on docker image hashicorp-go-container:latest

  [root@ip-10-0-4-55 hashicorp]# docker run -d --name=VMB2 --restart always  -e LISTEN_PORT="8080"  hashicorp-go-container:latest
  9fffce46b403922e9280c5c0450d25e8d708b8a83cfa4c0ca70d6fa24c774177
  [root@ip-10-0-4-55 hashicorp]# docker ps -a
  CONTAINER ID   IMAGE                                  COMMAND                  CREATED              STATUS              PORTS     NAMES
  9fffce46b403   hashicorp-go-container:latest          "/app/bin"               5 seconds ago        Up 5 seconds                  VMB2
  44085ff39472   hashicorp-go-container-vanila:latest   "/app/bin"               About a minute ago   Up About a minute             VMB1
  bf9d6e3c9987   nginx                                  "/docker-entrypoint.…"   8 minutes ago        Up 8 minutes        80/tcp    VMA
  [root@ip-10-0-4-55 hashicorp]# docker logs 9fffce46b403
  2025/07/29 11:14:17 Server listening on http://:8080
  [root@ip-10-0-4-55 hashicorp]#

Step 2.3 : Access application running on container VMB1 and VMB2 from VMA.


  [root@ip-10-0-4-55 hashicorp]# docker inspect VMA | grep IPAddress
              "SecondaryIPAddresses": null,
              "IPAddress": "172.17.0.2",
                      "IPAddress": "172.17.0.2",
  [root@ip-10-0-4-55 hashicorp]# docker inspect VMB1 | grep IPAddress
              "SecondaryIPAddresses": null,
              "IPAddress": "172.17.0.3",
                      "IPAddress": "172.17.0.3",
  [root@ip-10-0-4-55 hashicorp]# docker inspect VMB2 | grep IPAddress
              "SecondaryIPAddresses": null,
              "IPAddress": "172.17.0.4",
                      "IPAddress": "172.17.0.4",
  [root@ip-10-0-4-55 hashicorp]# docker ps -a
  CONTAINER ID   IMAGE                                  COMMAND                  CREATED          STATUS          PORTS     NAMES
  9fffce46b403   hashicorp-go-container:latest          "/app/bin"               4 minutes ago    Up 4 minutes              VMB2
  44085ff39472   hashicorp-go-container-vanila:latest   "/app/bin"               6 minutes ago    Up 6 minutes              VMB1
  bf9d6e3c9987   nginx                                  "/docker-entrypoint.…"   13 minutes ago   Up 13 minutes   80/tcp    VMA
  [root@ip-10-0-4-55 hashicorp]#
  [root@ip-10-0-4-55 hashicorp]# docker exec -it bf9d6e3c9987 /bin/bash
  root@bf9d6e3c9987:/# curl http://172.17.0.3:8080
  Hello, World!
  root@bf9d6e3c9987:/# curl http://172.17.0.4:8080
  Hello, World!
  root@bf9d6e3c9987:/#

