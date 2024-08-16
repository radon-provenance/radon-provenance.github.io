---
title: Radon Data Management System
permalink: /usage/cli.html
layout: default
sidebar_menu: Usage
sidebar_sub: Client Command Line Interface
---


{% include toc.html %}


# Presentation

The [Radon Command Line Interface](https://github.com/radon-provenance/radon-cli){:target="_blank"}
is a client project written in Python that can be installed on any machine. 
It accesses a Radon cluster through its RESTful APIs (CDMI and Admin interface).


# Installation

## Install needed packages

```
$ sudo apt-get install python3-venv
```

## Create And Activate A Virtual Environment

```
$ python3 -m venv ~/ve/radon-cli-1.0
...
source ~/ve/radon-cli-1.0/bin/activate
```

## Clone the radon-cli project

```
$ mkdir ~/ve/radon-cli-1.0/src
$ cd ~/ve/radon-cli-1.0/src
$ git clone https://github.com/radon-provenance/radon-cli.git
$ cd radon-cli
```

## Install Dependencies

```
$ pip install -r requirements.txt
```

## Install Radon Client

```
$ pip install -e .
```

# Usage

Don't forget to activate the virtual environment which has been created during
install. It will provide the _**radon**_ command.

```
$ source ~/ve/radon-cli-1.0/bin/activate
```

# Commands

## Connection

* Connect to a Radon Cluster as an anonymous user

```
$ radon init --url=http://radon-2.apps.l:8000
```

* Connect to a Radon Cluster as a registered user

```
$ radon init --url=http://radon-2.apps.l:8000 --username=USER [--password=PASS]
```

* Disconnect from the cluster

```
$ radon exit
```

* Show current authenticated user

```
$ radon whoami
```

## Manage Collections and Data Objects

* Show current working container

```
$ radon pwd
```

* List a container

```
$ radon ls <path>
```

* List a container with ACL information

```
$ radon ls -a <path>
```

* Move to a new container

```
$ radon cd <path>
```

* Move to the parent container

```
$ radon cd ..
```

* Create a new container

```
$ radon mkdir <path>
```

* Put a local file

```
$ radon put <src> [<dst>]
```

* Create a reference object

```
$ radon put --ref <url> <dest>
```

* Provide the MIME type of the object (if not supplied
  ``radon put`` will attempt to guess)

```
$ radon put --mimetype="text/plain" <src>
```

* Fetch a data object from the archive to a local file

```
$ radon get [--force] <src> [<dst>]
```

* Get the CDMI json dict for an object or a container

```
$ radon cdmi <path>
```

* Remove an object or a container

```
$ radon rm <src>
```

* Add or modify an ACL to an object or a container

```
$ radon chmod <path> (read|write|null) <group>
```

## Manage Metadata


* Set (overwrite) a metadata value for a field

```
$ radon meta set <path> "org.dublincore.creator" "S M Body"
$ radon meta set . "org.dublincore.title" "My Collection"
```

* Add another value to an existing metadata field

```
$ radon meta add <path> "org.dublincore.creator" "A N Other"
```

$ List metadata values for all fields

```
$ radon meta ls <path>
```

* List metadata value(s) for a specific field

```
$ radon meta ls <path> org.dublincore.creator
```

* Delete a metadata field

```
$ radon meta rm <path> "org.dublincore.creator"
```

* Delete a specific metadata field with a value

```
$ radon meta rm <path> "org.dublincore.creator" "A N Other"
```

## Manage Users and Groups

* Create a user

```
$ radon admin mkuser <username>
```

* List all users

```
$ radon admin lu
```

* Display information on a specific user

```
$ radon admin lu <username>
```

* Modify a user

```
$ radon admin moduser <username> (email | administrator | active | password) [<value>]
```

For instance to change the email of the user user10

```
$ radon admin moduser user10 email user10@radon.com
```

* Remove a user

```
$ radon admin rmuser <username>
```

* Create a group

```
$ radon admin mkgroup <groupname>
```

* List all groups

```
$ radon admin lg
```

* Display information on a specific group

```
$ radon admin lg <groupname>
```

* Add user(s) to a group

```
$ radon admin atg <groupname> <userlist> ...
```

* Remove user(s) from a group

```
$ radon admin rfg <groupname> <userlist> ...
```

* Remove a group

```
$ radon admin rmgroup <groupname>
```
