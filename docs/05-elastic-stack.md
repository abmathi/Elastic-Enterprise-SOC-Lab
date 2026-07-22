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

| Directory | Purpose |
|-----------|---------|
| /etc/elasticsearch | "    "|
| /usr/share/elasticsearch | "  " | 
| /var/lib/elasticsearch | "  " |
| /var/log/elasticsearch | "  " |