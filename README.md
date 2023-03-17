# WEB-SOLUTION-WITH-WORDPRESS
This repository gives a detailed step by step guide on how to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress.

# Three-tier Architecture
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers:
- Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
- Business Layer (BL): This is the backend program that implements business logic. Application or Webserver.
- Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File       System Server such as FTP server, or NFS Server.

# The Setup:
- A Laptop or PC to serve as a client
- An EC2 Linux Server as a web server (This is where you will install WordPress)
- An EC2 Linux server as a database (DB) server

In previous projects we used ‘Ubuntu’, but for this projects we will use a very popular distribution called ‘RedHat’.
Note: for Ubuntu server we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>


