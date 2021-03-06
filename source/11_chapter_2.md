# Implementation

## Introduction

In this chapter I will first outline the flow of information through Victor's
services in it's most general use case.  Then, working from the bottom up, I'll detail the implementation process including how working parts of the services are tied together and how each of the three services, the garden, the api, and the dashboard, interact.

## Flow Control

Victor uses a role based access control system in which only two types of users are considered to exist. The vast majority of users that might access the application are uncredentialed, outside users. This group makes up person who wishes to view the data, but has not been given access to make any changes. These users will almost always be accessing the site remotely, meaning requests to the dashboard will not come from the same network that the garden is connected to. The other group is much smaller and is made up of garden maintainers who want to and have the access to make changes to the garden remotely. These credentialed admin users have access to all of the same functions as the outside users; however, once their access has been properly vetted, they gain the additional power of sending post requests and commands to be evaluated on the garden machines.

As a outside user, the main point of interest in in consuming data about the garden's environment. Outside user's can use this a point of reference or comparison for their own gardens, so they want an easy way to consume recent relevant data. To do this the user must access the URL of the dashboard, which they determine either through word of mouth or over the internet via blog posts, Twitter, or other social media. Upon accessing the main page, the dashboard service immediately knows that it needs to request the most recent data from the API. The API seeing a request coming from the user, knows that read requests from an uncredentialed user are allowed, so it gathers the most recent set of data from the garden and sends it back as a JSON object to the dashboard. Once the dashboard gets the data it cleans it and constructs the appropriate graphs and widgets. Knowing that more relevant data is saved and stored with the API service every few minutes, the dashboard makes sure the check back in periodically to maintain relevance.

An outside user has no direct communication with the garden network itself, but rather the data that is sent from the garden's sensors is summoned and displayed by the dashboard on request. Communication between the garden and the API occurs only through one head device. The garden can be constructed using any number of machines, but communication is limited to being sent and received by a single elected proxy. Each machine that is a member of the garden's network hosts sensors contained by Docker containers. These contained processes continuously, generally based on a time increment though there is no enforced standard, take measurements about the garden environment and relay them to garden's head communicator device. The communicator maintains a service for relaying data points to the API to be stored. Data points are sent in the same manner regardless of which sensor they came from. This is a implementation decision meant to allow for the addition of any number of arbitrary sensors and controls. Doing so made the load a bit heavier for the dashboard service because data needs to be sorted and accounted for, but because there are large extensibility gains and the data is purely textual and relatively light weight it seemed to be worth it.

As a credentialed user this is still the most commonly used flow of data. Sensor data is sent to the API which is sent to dashboard after being asked for by the user. The control is augmented, however, by being able to send messages through the api directly to the garden. As an admin user after accessing the garden's dashboard page I have the option of logging in to the service. Logging in requires only a username and password. Submitting credentials sends a message to the api, which handles identification and authorization. If the user exists and the password is correct a token is sent back to the dashboard and access is granted. credentialed access allows the user to view another page that facilitates additional control interfaces. For instance, if the user wants to turn on the water pump manually for some amount of time they can elect to send a command to the API that is then relayed to garden's communicator and executed. Sending this message requires only two things aside from using the dashboards UI to send the proper message. First the token value that was received previously after signing in must match one of those that is currently valid and dispatched. Secondly, to prevent sniffing token values, the API must also be sent a One Time Password from an authorized two factor authentication key. Ultimately this means, to control the garden you need both a username and password as well as a physical device like a Yubikey to command the garden components. Once the API verifies that your request is valid it then sends the appropriate request to the communicator, which is then parsed and dispatched to the proper process.

These two roles and the parties that are allowed to communicate maintain a very robust and secure service despite the number of moving parts.

### Subsection 1 with example code block

This is the first part of the methodology. Cras porta dui a dolor tincidunt placerat. Cras scelerisque sem et malesuada vestibulum. Vivamus faucibus ligula ac sodales consectetur. Aliquam vel tristique nisl. Aliquam erat volutpat. Pellentesque iaculis enim sit amet posuere facilisis. Integer egestas quam sit amet nunc maximus, id bibendum ex blandit.

For syntax highlighting in code blocks, add three "`" characters before and after a code block:

```python
mood = 'happy'
if mood == 'happy':
    print("I am a happy robot")
```
