---
layout: post
title:  "Setup Mosquitto Broker via Docker compose"
description: "Setup and run eclipse mosquitto broker in a docker container"
date:   2022-09-02 00:00:00 +0500
categories: tech
image: mosquitto-syste-diagram.jpg
---

One of my office collegue got an assignment to setup <strong>Mosquitto</strong> in test environment. He was not aware of Mosquitto and how to setup, asked me to help him. After doing some research, I figured out, Mosquitto can be setup using <strong> DOCKER </strong> containers

Before diving in deep, You should know what is Mosquitto:

<blockquote>Eclipse Mosquitto is an open source message broker that implements the MQTT protocol. The MQTT protocol provides a lightweight method of carrying out messaging using a publish/subscribe model. This makes it suitable for Internet of Things messaging such as with low power sensors or mobile devices such as phones, embedded computers or microcontrollers -- <a href="https://en.wikipedia.org/wiki/Modulo_operation">Wikipedia</a></blockquote>


Following is the <strong>docker-compose.yml</strong> I used to setup Mosquitto:

<pre class="EnlighterJSRAW" data-enlighter-language="docker">
version: "3"
services:
    mosquitto:
        container_name: mqtt
        image: eclipse-mosquitto
        ports:
            - 1883:1883
        volumes:
            - "./mosquitto/config/broker.conf:/mosquitto/config/mosquitto.conf"
            - "./mosquitto/config/password.passwd:/mosquitto/config/mosquitto.passwd"
            - "./mosquitto/data:/mosquitto/data"
        networks:
            - app-network
networks:
    app-network:
        driver: bridge
}</pre>

<strong>Here is some explanation:</strong>

I gave my container a user-friendly name : "mqtt" so that I can access it and run some commands
<pre class="EnlighterJSRAW" data-enlighter-language="docker">
container_name: mqtt
</pre>

"eclipse-mosquitto" is the official image mentioned on their website. I used the latest image, thats why did not mentioned any version.
<pre class="EnlighterJSRAW" data-enlighter-language="docker">
image: eclipse-mosquitto
</pre>

"1883" is default port used by Mosquitto to enqueue/dequeue messages in broker. Since we are running in container, We need to also expose the port to host machine.
<pre class="EnlighterJSRAW" data-enlighter-language="docker">
ports:
    - 1883:1883
</pre>

Three directories, I used as volume to preconfigure the setup.
    
<pre class="EnlighterJSRAW" data-enlighter-language="docker">
volumes:
    - "./mosquitto/config/broker.conf:/mosquitto/config/mosquitto.conf"
    - "./mosquitto/config/password.passwd:/mosquitto/config/mosquitto.passwd"
    - "./mosquitto/data:/mosquitto/data"
</pre>

<ul>
    <li>To pass the configuration file</li>
    <li>To pass the list of users with their credentials.</li>
    <li>Since containers are volatile, Everything will be lost if container restarted. To make data persist, I mounted directory to keep the data on host machine.</li>
</ul>

<strong>Configuration file:</strong>

<pre class="EnlighterJSRAW" data-enlighter-language="conf">
listener 1883
password_file /mosquitto/config/mosquitto.passwd
allow_anonymous false

persistence true
persistence_location /mosquitto/data
autosave_interval 10s
max_queued_messages 100</pre>

<strong>Dockerization</strong>

To up the container, you can run following command in command line or terminal:
<pre>docker-compose up -d</pre>

To start interactive session, run following command:
<pre>docker exec -it mqtt /bin/sh</pre>

<u>Note:</u> Image does not support "bash" alias

<strong>Security measurement:</strong>

I disabled anonymous usage in configuration file by changing <strong>allow_anonymous</strong> to false and provided the location of password file by <strong>password_file</strong>. 

After starting the container, You need to add a user with password so that pub/sub can connect with your broker. Since, directory was mounted added user will presist even after restarting the container. Use 
<pre>mosquitto_passwd -c password username</pre> 
to store passwords in file. For example, command 
<pre>mosquitto_passwd -c golang golang</pre> 
will write following content in password file.

<pre class="EnlighterJSRAW" data-enlighter-language="conf">
golang:$7$101$ccJ/dnmr0jLqzO6B$V4Y36vs2XlMFRsCe7zyLJdDeL1s++YSCm7ZbR1cVCA592o0td3hZrQ91J5w0cFSiM/3oBHnnT9gUO5xYSSheMQ==
</pre>

<strong>End to End connectivity:</strong>

Installation of Mosquitto comes with some utilities to check end to end connectivity. These utilities will be accessable after starting up the docker container successfully.

<u>Mosquitto Producer:</u>
This utility helps to enqueue messages in plain text. After opening the interactive bash session in mqtt container, You can run following command to enqueue the message:

<pre>mosquitto_pub -h localhost -p 1883 -u golang -P golang -t my-mqtt-topic -m "Test Message"</pre>

<u>Mosquitto Subscriber:</u>
This utility helps to dequeue messages in plain text. After opening the interactive bash session in mqtt container, You can run following command to dequeue the message:

<pre>mosquitto_sub -h localhost -p 1883 -u golang -P golang -t my-mqtt-topic</pre>

Note: Make sure that credentials you set through password utility are correct. 

<strong>Conclusion:</strong>
I hope this post gave you a useful overview of getting an MQTT Mosquitto Broker up and running using Docker.
