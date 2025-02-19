---
title: "Access Jenkins vagrant vm lab from host"
seoTitle: "jenkins lab"
seoDescription: "unable to access jenkins vagrant vm box lab"
datePublished: Tue Apr 18 2023 04:30:18 GMT+0000 (Coordinated Universal Time)
cuid: clglrnxbn000009mi2jlf8tt1
slug: access-jenkins-vagrant-vm-lab-from-host
tags: vagrant, docker, networking, jenkins, vagrantfile

---

When following the cloudbees jenkins administration study material, you will be required to set up your own private lab using a vagrant box. When you are doing it you migth have issue accessing the jenkins lab server hosted in the vagrant box. To fix those issues you need to

* Set a private network with vm IP in the vagrant file
    
* Ssh to the vm. You need to be in the folder containing the Vagrantfile then run the following command:
    
    ```plaintext
    vagrant ssh
    ```
    
* Edit the docker-compose file under /docker/docker-compose.yml
    

```plaintext
cd /docker
vi docker-compose.yml
```

* Edit jenkins service **"expose** to **ports"** the reason is that expose will allow services in the same network to access the ports without making them accessible to the host machine(vm) while ports will make them available on both with a random port number if you don't specified the host port to be linked on the docker port. You can find more explanation here [https://stackoverflow.com/questions/40801772/what-is-the-difference-between-ports-and-expose-in-docker-compose](https://stackoverflow.com/questions/40801772/what-is-the-difference-between-ports-and-expose-in-docker-compose)
    

![Change 'expose' to 'ports'](https://cdn.hashnode.com/res/hashnode/image/upload/v1681791319561/4686680b-4dbf-4856-9f85-53e6f48b58ee.png align="center")

Change it to ports

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681791471521/a202f3e7-2122-4344-8955-ee94e2fff3b0.png align="center")

* Save your changes and restart docker-compose
    

```plaintext
docker-compose up -d #when it's done check the port expose
docker ps
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1681791632115/945ae5dd-4b13-4552-a1ba-a169e669b085.png align="center")

On the previous image, check for the ports linked to ***jenkins container.*** You will now be able to access it through your host browser by specifying the ports number after the vm ip address. it will look like :

```plaintext
vm-ip:portnumber
```

Here you might burst on an unfamiliar page. just add jenkins

Example : *192.168.56.2:32769/jenkins*

You will then land on the login page.

### **Important to note you could just open Jenkins by using the url localhost:5000 in your web browser after launching the the cloudbees vagrant box.**

Thanks for reading :)