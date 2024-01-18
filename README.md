
# Nginx Web Server Configuration and DDoS Simulation (This is for home practice in a controlled/virtual environment)


## Table of Contents

- [Introduction](#introduction)

This project details the setup of a web server with Apache, migration to Nginx, configuration for traffic handling, creation of a test webpage (`index.html`), and the execution and analysis of a simulated DDoS attack.

## - [Initial Setup](#initial-setup)
## - [Apache Installation](#apache-installation)

Install and configuration:

`sudo apt install apache2`

`sudo nano /etc/apache2/apache2.conf`

`ServerName server_domain_or_IP`

UFW Firewall configuration for Apache

`sudo ufw allow 'Apache Full'`

`sudo ufw reload`

Testing Apache

`http://server_domain_or_IP in browser`

  - [Nginx Installation](#nginx-installation)
- [Nginx Configuration](#nginx-configuration)

## Install Nginx

`sudo apt install nginx`

Configuration Nginx

`sudo nano /etc/nginx/nginx.conf`

Nginx server block configuration fail

`sudo nano /etc/nginx/sites-available/your_domain`

Placeholder

`server_name your_domain.com;`

Testing configuration and restart Nginx:

`sudo nginx -t`

`sudo systemctl restart nginx`

## Create Zones of Limitation

Open configuration file

`sudo nano /etc/nginx/nginx.conf`

Add in the http block:

`limit_req_zone $binary_remote_addr zone=mylimit:10m rate=10r/s;`

(Here, $binary_remote_addr represents the IP address of the client, mylimit is the name of the limiting zone, 10m is the amount of memory allocated for monitoring sessions (which allows monitoring about 16000 IP addresses), and rate=10r/s limits requests to 10 requests per second.)

Application of Limitation to Server or Locations

`location / { limit_req zone=mylimit burst=20 nodelay;`

Configuration testing

`sudo nginx -t`

If the test passes without errors, restart Nginx to apply the changes:

`sudo systemctl restart nginx`

(NOTE: Rate limiting will not protect you from all types of DDoS attacks, especially if the attack comes from a distributed network with a large number of IP addresses or if the attack overwhelms the network infrastructure before the traffic reaches your Nginx server. For more advanced protection, it is necessary to use additional tools and strategies, such as network firewalls, DDoS mitigation services (such as Cloudflare, AWS Shield, etc.), and hardware appliances specialized for DDoS mitigation.)

## - [Test Webpage](#test-webpage)

- [Test Page](#test-page)


        www.PANDA Edjukejsn.org

        Testiranja alata za generisanje saobracaja
        Forma za unos podataka
         
         Name
         Email
         Send



  - [Simulation Setup](#simulation-setup)

Setting up DDoS Attack Protection

If your server block is in a separate file within sites-available, add the limit_req directive to the corresponding location block. If it's in nginx.conf, add it directly there.
For example, if you add to nginx.conf, it should look like this:


     location / {
         root /var/www/html;
         index index.html;
         limit_req zone=mylimit burst=5 nodelay;
     }


  - [Attack Execution](#attack-execution)

Load Testing Tool

ApacheBench (ab)
`sudo apt-get install apache2-utils`

Run Load Testing
`ab -n 10000 -c 100 http://localhost/ (instead of 10000 you can put any number to make sure of the real load)`

This command will send a total of 10,000 HTTP GET requests to the server http://localhost/ with 100 competing requests at any time. This can generate a significant load on the server.

Running the test from another Ubuntu machine:

Running the test from another machine (or multiple machines) gives a more realistic simulation of a DDoS attack because it comes from multiple different sources.
This is also a good way to test the network infrastructure and how it handles traffic coming from outside the local environment.
The goal is to test the basic performance and capacity of our server.


  - [ApacheBench Results](#apachebench-results)

_Attack machine_:

`ab -n 90000 -c 100 http://192.168.100.nn/`

Result:

This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 192.168.100.nn (be patient)
* Completed 9000 requests
* Completed 18000 requests
* Completed 27000 requests
* Completed 36000 requests
* Completed 45000 requests
* Completed 54000 requests
* Completed 63000 requests
* Completed 72000 requests
* Completed 81000 requests
* Completed 90000 requests
* Finished 90000 requests

* Server Software: nginx/1.18.0
* Server Hostname: 192.168.100.nn
* Server Port: 80

* Document Path: /
* Document Length: 573 bytes

* Concurrency Level: 100
* Time taken for tests: 15,765 seconds
* Complete requests: 90000
* Failed requests: 89979
    * (Connect: 0, Receive: 0, Length: 89979, Exceptions: 0)
* Non-2xx responses: 89979
* Total transferred: 28900374 bytes
* HTML transferred: 14588631 bytes
* Requests per second: 5708.99 [#/sec] (mean)
* Time per request: 17.516 [ms] (mean)
* Time per request: 0.175 [ms] (mean, across all concurrent requests)
* Transfer rate: 1790.28 [Kbytes/sec] received

* Connection Times (ms)
              * min mean[+/-sd] median max
* Connect: 0 7 10.5 5 179
* Processing: 1 11 12.1 8 271
* Waiting: 1 10 9.5 7 271
* Total: 3 17 19.5 13 315

* Percentage of the requests served within a certain time (ms)
   - 50% 13
   - 66% 16
   - 75% 18
   - 80% 19
   - 90% on the 27th
   - 95% 39
   - 98% 61
   - 99% 109
  - 100% 315 (longest request)


## _The results from ApacheBench provide important information about how your Nginx server reacts to high load. Here's what each segment of the score means_:

## Server Software: 
The version of Nginx that is currently running.

## Complete requests: 
Total number of requests that ab successfully completed (90,000).

## Failed requests: 
Number of requests that were not successfully executed (89,979). A very high number of failed requests indicates that the server could not process a large number of requests, which is often the case under a DDoS attack.

## Non-2xx responses: 
The number of responses that were not successful HTTP responses (also 89,979), which usually indicates that the server returned a large number of 503 Service Unavailable responses, which is normal when rate limiting is activated.

## Requests per second: 
Average number of requests per second that the server was able to process during the test (5708.99). It is a measure of server performance under load.

## Time per request: 
Average time required to process one request. 17,516 ms per request for one user, and 0.175 ms per request distributed over all competing requests.

## Transfer rate: 
Data transfer rate achieved during the test.

## Connection Times: 
Displays the minimum, average, median, and maximum request times, including connection and processing times.

## Percentage of the requests served within a certain time: 
Distribution of requests by response time. For example, 50% of all requests are processed within 13 ms.

## Conclusion:

Most of the requests failed, indicating that the server is overloaded or that the rate limiting is too restrictive.
A high number of requests per second (Requests per second) can be a good sign, but only if the server can handle such a load without a large number of errors.
The response time is relatively short, which indicates that the server responds quickly to requests until it reaches its capacity.


- [Nginx/error.log](#nginx/error.log)

`sudo tail -n 100 /var/log/nginx/error.log`

`2024/01/18 16:28:08 [error] 9782#9782: *340089 limiting requests, excess: 5.788 by zone "mylimit", client: 192.168.100.29, server: 192.168.100.28, request: "GET / HTTP/ 1.0", host: "192.168.100.28"
2024/01/18 16:28:08 [error] 9782#9782: *340089 open() "/usr/share/nginx/html/50x.html" failed (2: No such file or directory), client: 192.168 .100.29, server: 192.168.100.28, request: "GET / HTTP/1.0", host: "192.168.100.28"
2024/01/18 16:28:08 [error] 9782#9782: *340090 limiting requests, excess: 5.788 by zone "mylimit", client: 192.168.100.29, server: 192.168.100.28, request: "GET / HTTP/ 1.0", host: "192.168.100.28"`

## Rate Limiting:
The logs show that Nginx applies the request limiting you configured. Messages like:
limiting requests, excess: 5,788 by zone "mylimit"

## Problem with the error page (Custom Error Page):
The second set of logs indicates a problem with the custom error page:
open() "/usr/share/nginx/html/50x.html" failed (2: No such file or directory)

## Solutions:

For the problem of limiting requests:
Increasing the rate value or burst parameter in the limit_req_zone directive if too many requests are rejected. This will allow more flexibility before requests start to be rejected.

