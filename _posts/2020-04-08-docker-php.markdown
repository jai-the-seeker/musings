---
layout: post
title:  "Docker Infrastructure for Attacker Victim PHP-MySql Lab"
date:   2020-04-08 23:20:58 +0530
categories : [docker]
tags: [docker, web exploitation, Php, MySql, infrastructure]
---

{% include toc %}
# After Fresh Installation
Once we have installed all the CE-edition of docker and docker-compose from the docker website, we have following with us:
{% highlight console %}
{% endhighlight %}

{% highlight console %}
$ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
hello-world         latest              fce289e99eb9        15 months ago       1.84kB
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
efc1e1ca6409        bridge              bridge              local
34f81bd2e623        host                host                local
94d2f826785d        none                null                local
{% endhighlight %}

# Creating Dockerfile and Docker-Compose.yml
The Dockerfile caters for `pdo_mysql` extensions. The Dockerfile used in project is [here](https://github.com/jai-the-seeker/web-security/blob/master/infrastructure/app-machine/php/Dockerfile) 

After running the `docker-compose up` command, the sate of our docker containers, networks is as follows:

{% highlight console %}
{% raw %}$ docker container ls --format "{{.ID}}: {{.Names}}: {{.Ports}}" {% endraw %}
1f3cb046fe8c: app-web-php74: 0.0.0.0:5000->80/tcp
c715238c051c: app-phpmyadmin: 0.0.0.0:5001->80/tcp
b66a4f79e291: app-db-mysql: 3306/tcp, 33060/tcp
{% endhighlight %}

{% highlight console %}
$ docker network ls 
NETWORK ID          NAME                          DRIVER              SCOPE
5e9b89160c36        app-machine_app-network       bridge              local
efc1e1ca6409        bridge                        bridge              local
a2e9779a2337        docker-files_server-network   bridge              local
34f81bd2e623        host                          host                local
94d2f826785d        none                          null                local
{% endhighlight %}

{% highlight console %}
$ docker network inspect app-machine_app-network 
...
        "Containers": {
            "1f3cb046fe8cd8a1c6581d8c582d1acc6854b6af5badf17f6ed395a4f4acd934": {
                "Name": "app-web-php74",
                "EndpointID": "4807260f315b9b97cec457e73c5eec78ad70170d0e7b4e34fd30c57448b96e8f",
                "MacAddress": "02:42:ac:19:00:04",
                "IPv4Address": "172.25.0.4/16",
                "IPv6Address": ""
            },
            "b66a4f79e2911a56671a2c35543ac2d9bb2a4ef8826f2f426ba6a555ace46ff6": {
                "Name": "app-db-mysql",
                "EndpointID": "eff9246fa1ffd075ca5b7a66f2ab5a01177b7607a50b545a9814eda76b1e22e6",
                "MacAddress": "02:42:ac:19:00:02",
                "IPv4Address": "172.25.0.2/16",
                "IPv6Address": ""
            },
            "c715238c051c85f3516d55e70079b3fe85bad7a43c6bc9cc64601b81f9650f62": {
                "Name": "app-phpmyadmin",
                "EndpointID": "3626814d31c4909c05a41f73bb7afb06fded070d217c9ece38cf297e31626850",
                "MacAddress": "02:42:ac:19:00:03",
                "IPv4Address": "172.25.0.3/16",
                "IPv6Address": ""
            }
        },
...
{% endhighlight %}

After our containers are up, let us check the connectivity of mysql. Create [check-conn.php](https://github.com/jai-the-seeker/web-security/blob/master/infrastructure/check-conn.php) file and place it in appropriate location. Now, we can visit this page to verify our connectivity with the database.



 
