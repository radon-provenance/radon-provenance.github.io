---
layout: default
---


* auto-gen TOC:
{:toc}


# Radon Command Line Interface


## Description

A Python client and command-line tool for the Radon data management system.
Includes a rudimentary client for any CDMI enabled cloud storage.

After Installation, connect to a Radon node

```shell
radon init --url=http://radon.example.com
```

(or if authentication is required by the archive):

```shell
radon init --url=http://radon.example.com --username=USER --password=PASS
```

(if you don't want to pass the password in the command line it will be asked if
you don't provide the --password option)

```shell
radon init --url=http://radon.example.com --username=USER
```

Close the current session to prevent unauthorized access:

```shell
radon exit
```

Show current working container:

```shell
radon pwd
```

Show current authenticated user:

```shell
radon whoami
```

List a container:

```shell
radon ls <path>
```

List a container with ACL information:

```shell
radon ls -a <path>
```

Move to a new container:

```shell
radon cd <path>
...
radon cd ..  # back up to parent
```

Create a new container:

```shell
radon mkdir <path>
```

Put a local file, with eventually a new name:

```shell
radon put <src>
...
radon put <src> <dst>
```

Create a reference object:

```shell
radon put --ref <url> <dest>
```

Provide the MIME type of the object (if not supplied ``radon put`` will attempt
to guess):

```shell
radon put --mimetype="text/plain" <src>
```

Fetch a data object from the archive to a local file:

```shell
radon get <src>

radon get <src> <dst>

radon get --force <src> # Overwrite an existing local file
```

Get the CDMI json dict for an object or a container

```shell
radon cdmi <path>
```

Remove an object or a container:

```shell
radon rm <src>
```

Add or modify an ACL to an object or a container:

```shell
radon chmod <path> (read|write|null) <group>
```

## Advanced Use - Metadata

Set (overwrite) a metadata value for a field:

```shell
radon meta set <path> "org.dublincore.creator" "S M Body"
radon meta set . "org.dublincore.title" "My Collection"
```

Add another value to an existing metadata field:

```shell
radon meta add <path> "org.dublincore.creator" "A N Other"
```

List metadata values for all fields::

```shell
radon meta ls <path>
```

List metadata value(s) for a specific field:

```shell
radon meta ls <path> org.dublincore.creator
```

Delete a metadata field:

```shell
radon meta rm <path> "org.dublincore.creator"
```

Delete a specific metadata field with a value:

```shell
radon meta rm <path> "org.dublincore.creator" "A N Other"
```

## Advanced Use - Administration

List existing users:

```shell
radon admin lu
```

List information about a user:

```shell
radon admin lu <name>
```

List existing groups:

```shell
radon admin lg
```

List information about a group:

```shell
radon admin lg <name>
```

Create a user:

```shell
radon admin mkuser [<name>]
```

Modify a user:

```shell
radon admin moduser <name> (email | administrator | active | password) [<value>]
```

Remove a user:

```shell
radon admin rmuser [<name>]
```

Create a group:

```shell
radon admin mkgroup [<name>]
```

Remove a group:

```shell
radon admin rmgroup [<name>]
```

Add user(s) to a group:

```shell
radon admin atg <name> <user> ...
```

Remove user(s) from a group:

```shell
radon admin rfg <name> <user> ...
```


## Installation


### Create And Activate A Virtual Environment


```shell
$ python3 -m venv ~/ve/radon-cli
...
$ source ~/ve/radon-cli/bin/activate
```

### Install Dependencies

```shell
pip install -r requirements.txt
```

### Install Radon Client

```shell
pip install -e .
```

## License

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.

Parts of this work were supported by the Software Sustainability Institute with 
funding from EPSRC, BBSRC, ESRC, NERC, AHRC, STFC and MRC through grant EP/S021779/1.


