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

Step 5c : Created docker VMA based on Go application

  #######
  #######
  [root@ip-10-0-4-55 hashicorp]# docker run -d --name=VMA  -e LISTEN_PORT="8080" hashicorp-go-container:latest
  [root@ip-10-0-4-55 hashicorp]# docker ps
  CONTAINER ID   IMAGE                           COMMAND      CREATED          STATUS          PORTS     NAMES
  c0e787184f8f   hashicorp-go-container:latest   "/app/bin"   11 seconds ago   Up 10 seconds             VMA
  [root@ip-10-0-4-55 hashicorp]# docker logs c0e787184f8f
  2025/07/29 07:18:59 Server listening on http://:8080
  [root@ip-10-0-4-55 hashicorp]#
  #######
  #######

Step 6a : Created a dockerfile to dockerize the application using multi-stage builds (alpine)
