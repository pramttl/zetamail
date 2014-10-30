## Zetamail: Open source cloud based email delivery software
Next generation cluster based MTA, A full stop to your MTA woes


### Introduction

The common challenges faced with present day mail transfer agents or MTAs are:

* Low throughput
* Less scalability
* Complex deployment

Throughput and scalability are related in that existing  MTA software are designed to be deployed on a single server environment which inherently makes these software non-scalable.

Existing open source MTA software like Postfix and Exim are examples of excellent MTAs but come with the same problem, i.e. MTA software designed to run on a single node which makes them slow and limited in throughput, irrespective of the server they are deployed on.


This isn’t an MTA software issue alone. SMTP itself is a slow protocol and blocking in nature. As a result, there are instances of SMTP server deployments where emails drop simply because of a flow control problem where the MTA is busy sending emails when it receives request to send more emails.

This blocking nature of SMTP does not imply that the problem of avoiding dropped connections cannot be solved. A simple solution is to have a layer around the SMTP server that allows asynchronous calls and queues them for the SMTP server to process. This solves the problem of flow control.

![image](https://dl.dropboxusercontent.com/u/46611567/zetamail_static/arch1.png)

### Goal of this project

The goal of this project is to implement a simple and scalable email solution that you can deploy on your own server stack at a single click and make it super easy to add and drop SMTP servers to the mailing cluster conveniently.

The idea is analogous to a cloud cluster where new nodes can be added to an existing cluster or dropped out. When the world has moved to the cloud then why should emails stay behind? That is the underlying philosophy of this project.


#### Why is this project of significance to you?

Let’s answer this question by asking some questions for you to ponder upon.


Most email sending services that allow sending emails via an API like Mailgun (Rackspace) or Mandrill provide it as a service and charge a per email cost. Why pay a per email cost when you could run your own emailing stack?


Have you ever felt the frustration caused due to a restricted email rate? If you have ever created your account on Mandrill, AWS SES or Mailgun, you must have noticed that these services place a per second or hourly quota for sending emails and sometimes that is not good enough for you. Are we right?


Have you ever had problems removing your existing SMTP node and re-deploying a new one?


#### What is this project about?


An application that can be deployed on your cluster of nodes and will act as a high throughput scalable MTA system which can be accessed via an HTTP API.

#### What is this project NOT about?


A complete email inbox solution: (sending+receiving mails). Check Mail-in-a-box project if you want to implement your own inbox solution like gmail, hotmail, etc.

A spam machine. This project does not encourage or aid spamming. Spam detection and protection technologies have grown rapidly and are now at a stage where any IP or domain which sends spam can be easily spotted and blacklisted based on response from users. There are DNS and IP blacklists that list Domains or IP addresses that send spam and it is not hard to land in one of these lists if your server sends unsolicited spam mails. This project cannot help you if your IP or server gets blacklisted, but it would make it easy for you to “update” your cluster by dropping that server and adding a new one.


### Solution Architecture

![image](https://dl.dropboxusercontent.com/u/46611567/zetamail_static/arch2.png)

Technical Details and Functionality

1. To make the setup and deployment of an SMTP server a one click process.
Devops (Automation in setup up of SMTP server). Common configuration done automatically.

2. Defining how the Load Balancer interacts with the SMTP child nodes
Load balancer opens an SMTP connection to each SMTP child node to send an email or multiple emails.
All the emails are maintained in a queue on the load balancer.
Setting up a load balancer should also be a one click process, with minimum manual configuration to do, having Chef/Ansible scripts to automatically spawn the load balancer instance. (Devops)


3. Defining how users interact with the load balancer (HTTP API + Web Admin Interface)
HTTP API is the most effective way to interact with the load balancer or master node since HTTP can allow us to make calls both asynchronous as well as blocking as suitable. For a high throughput system allowing asynchronous calls is quite important.
Security of the HTTP API. Multiple users can be added via the web interface by organization administrators.
Using Django for the web backend.



4. Working of SMTP servers.
The SMTP servers are simply auto-configured MTAs responsible for delivering emails. They only speak SMTP.


#### Iterative Development

Rough architecture explanation for iteration 1:

The master node maintains a queue Q whenever an emailing request is made the request enters the queue as an emailing task, each of the SMTP server is a worker. The master will select which ever SMTP server is free to send the email.  (The concept of Distributed Task Queues will be applied here)

Software to be used (One of the most popular free AMQP software): rabbitmq-server
The protocol is responsible for selecting the available worker and it abstracts the functionality for us. So for the initial prototype we will not worry about how to handle the delegating of email tasks to independent child nodes, we will simply add it to the distributed task queue and design worker software that are capable of processing tasks from the queue.

A safe way to do this is to use a good AMQP client: Celery or Pika (for Python)


#### HTTP API Specification v1.0

##### Sending a single email

    http://<server_ip>/sendemail

```
{
  “from”: “someone@example.com”
  “to”: “someone@someotherexample.com”,
  “text”: “email_text_content”,
  “html”: “email_html_content”,
  “reply-to”: “...”,
}
```

##### Sending multiple emails

http://<server_ip>/sendmultiemails

    http://localhost:8000/sendmultimails


#### Why is this an unsolved problem? (Links)

* https://news.ycombinator.com/item?id=203242
* https://signalvnoise.com/posts/3096-behind-the-scenes-giving-away-the-secrets-of-email-delivery
