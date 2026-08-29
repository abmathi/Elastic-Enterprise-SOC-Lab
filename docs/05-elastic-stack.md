# Building the Elastic Stack

### What is Elastic Stack?

Elastic Stack is the combination of the tools Elasticsearch, Kibana, Beats, and Logstash and is also known as the ELK Stack. The Elastic Stack allows you to take data from any source and format for the purpose of organizing, searching through, analyzing, and visualizing it. 

### What is Kibana? 

Kibana is the tool in the stack that allows you to take the data and turn it into visualizations such as bar, line, or scatter charts, pie charts, maps, and many more different ways to arrange your visualized data into dashboards.

### Why use package repositories instead of random installers?

Downloading random installers through the internet increases the risk of installing malicious or outdated software. Using package repositories is a good way to combat this as it reduces that risk since they distribute software from trusted, verified sources and often check for authenticity and automatic updates.

### What is a GPG signing key?

GPG signing keys are cyrptographic keys that are used to  verify software or packages were created by trusted souces and haven't been altered with. Each package has a digital signature and when you install software from a package repository, it checks for that digital signature to confirm it's valid, meaning you can be more confident the software is authentic.

### Why are repository-specific keyrings more secure than one global trusted keyring?

A repository-specific keyring would be more secure because each repository only trusts its own signing key, which limits the scope of trust. If a key is compromised, it can't be used to sign packages for other repositories whereas a single trusted keyring allows any trusted key to authenitcate packages from any repository. 

### SOC Relevance

One of the fastest growing attack vectors is from the supply chain. Examples of this can include:

- Malicious package repositories
- Compromised update services    
- Dependency confusion
- Stolen signing keys

Secure software installation helps protect systems from malware and unauthorized software. It's easier for analysts to maintain system integrity when using repository-specific keyrings, knowing that trust is limited to the correct repository.

## Trusting the Elastic Repository

### Command

```bash
ls -ls /etc/apt/keyrings
```

### Prediction

I predicted that this command would show me whether or not there was a keyring directory

### Reason

I ran this command in order to find out if the keyring directory already existed or if one needed to be created.

### SOC Relevance

Adding the public key for Elastic to the keyring is a crucial step in the building process as it will be used to setup the software.

---

### Command

```bash
gpg --show-keys /etc/apt/keyrings/elastic.gpg
```

### Prediction

I predicted that this command would show me whether the key was added correclty.

### Reason

This command was used in order to verify that the key exists after running the curl command to download it.

### SOC Relevance

Knowing how to double check and confirm the steps you are taking is a valuable skill that will save you a lot of time in the long run.

---

## Adding the Elastic Repository

### Command

```bash
echo "deb [signed-by=/etc/apt/keyrings/elastic.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | \
sudo tee /etc/apt/sources.list.d/elastic-9.x.list
```

### Predction

Before running this command, I predicted that it would be used to add the elastic repository to the specified file.

### Reason

This command is used to write the cofiguration to a new file with root priveleges and only trust packages signed by this specific key.

### SOC Relevance

It's important for analysts to clarify what packages they are trusting when building a system that is using least privilege and limited trust. 

---

### Command

```bash
sudo apt update
```

### Reason 

We used this command again to download the latest package metadata after adding the package repo.

### SOC Relevance

This command is like refreshing the catalog of a library. You know what books are available but you haven't checked any out yet. 

---

### Command

```bash
apt-cache policy elasticsearch
```

### Reason 

This command was used to verify that elasticsearch is available before installing it.

### SOC Relevance 

This is just a precautionary step to answer questions like if the package is available, which version would be installed, and what repository provides it. 

---

### Command

```bash
apt show elasticsearch
```

### Reason

This comand is used to explore the package further and see metadata like the version, who maintains it, install size, and more.

### SOC Relevance

This is a useful command to get a better sense of what you're downloading before you do it, as it provides a lot of useful information to better understand the package. 

---

## Installing Elasticsearch

### Command

```bash
sudo apt install elasticsearch
```

### Prediction

My prediction beforehand was that this command would start the install of elasticsearch.

### Reason

The reason behind running this command was to install elasticsearch on the system.

### SOC Relevance

This is an important milestone and the start of building out the SIEM infrastructure.

---

### Command

```bash
dpkg -l | grep elasticsearch
```

### Prediction 

I thought this command would have something to do with checking if the install command worked properly.

### Reason 

The reason for running this command is to verify that the package is installed on the system.

### SOC Relevance

Knowing how to ask Debain if this package is installed is an important step to know. 

---

### Command

```bash
dpkg -L elasticsearch
```

### Prediction 

My prediction for this one is that the -L would be used in the dpkg command to list something to do with trhe elasticsearch repository.

### Reason

The reason for running this command is to discover all the files in the selected repository.

### SOC Relevance

An important thing for analyts to do is not treat applications like black boxes. It is important to know how to discover what lives inside the things you add to your systems.

---

## Filesystem Hierarchy Standard

| Directory | Purpose |
|-----------|---------|
| /etc/elasticsearch | Configuration Files|
| /usr/share/elasticsearch | Application Binaries and supporting files | 
| /var/lib/elasticsearch |Indexed data and cluster state |
| /var/log/elasticsearch | Servicing log files|

---

## Starting Elasticsearch

### Command

```bash
systemctl status elasticsearch
```

### Purpose

This command is used to check the current status of Elasticsearch and return whether it's running, enabled, etc.

### Result

This command returned the status as being disabled and inactive.

### SOC Relevance

Knowing how to check if the application is actually running will help in troubleshooting. Before looking in any directories, you can run this command to see the status of the application.

---

### Command

```bash 
systemctl enable elasticsearch
```

### Purpose

This command tells linux to start the service automatically every time that the server boots up. 

### SOC Relevance

This is a quality of life type of command, as you don't want to manually start the service every time you reboot the server.

---

### Command

```bash
systemctl start elasticsearch
```

### Purpose 

This is the command that actually starts the service. 

---

## Secure Elasticsearch API Access

    HTTP request to port 9200
    
        ↓

    curl error 52 / empty reply

        ↓

    verified port 9200 was listening

        ↓

    tested HTTPS

        ↓

    received authentication-required response

        ↓

    identified TLS + authentication as expected behavior

        ↓

    used http_ca.crt + elastic credentials

        ↓

    successful API response

        ↓

    cluster health: green

---

## Kibana Installation 

### What Kiabana Does

Kibana is the user interface and server for visualizing the data stored in elasticsearch. It provides dashbaords, visualizations, and tools to manage patterns and objects. 

### Why Kibana and Elasticsearch are seperate services

Elasticsearch stores and searches data; Kibana presents and queries data through the UI.

### Why Kibana gets its own Linux server

Least Privilege: Kibana process should run with an account limited to only what it needs.

## Kibana Initial Setup

### What an enrollment token is

An enrollment token is a short lived credential used to connect to a new Kibana instance to the elasticsearch cluster.

### Enable VS Start

systemctl enable: sets the service to start automaticaly on boot

systemctl start: starts the service in the current session

### What the local curl test proved

The local curl test cofirmed that the kibana http endpoint was listening and responding to requests. 

## Kibana Secure Environment

### Kibana Enrollment Token 

The enrollment token securely connected Kibana to the Elasticsearch cluster during setup.

### Why Kibana uses service identity instead of elastic

elastic is a highly privileged admin account and should not be used for routin operations in applications.

### How Kibana was verified after enrollment

Kibana was verified by confimring it started successfully and could communicate with elasticsearch.

~~~
Internet / updates
       │
     NAT
   enp0s3

SOC telemetry
       │
 SOC-LAB-NET
   enp0s8


Admin access
       │
 Host-Only
   enp0s9
~~~

## Kibana Interface Exploration


| Kibana area| What it is |
|-----|--------|
| Stack Management| Data stored in Elasticsearch |
|Index Management | Maintains performance and ensures effecient data storage |
|Discover | Explore and search documents in elasticsearch  |
|Elastic Security| Build security oriented workflows |
|Fleet | Centralized agent management plane |
| Dev Tools | Console  |

