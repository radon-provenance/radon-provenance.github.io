# Admin Command Line Interface


## Presentation

When installed, the Radon lib provides a minimal set of commands
that can be used to configure the system. They have to be executed
on an installed Radon node (radon-web or radon-admin docker image).


## Commands

### Initialise Cassandra

This command has to be executed once to set up the keyspace and the tables in
Cassandra.

```
$ radmin init
```

### Drop the database

This command drops the keyspace and all the tables in Cassandra.

```
$ radmin drop
```

### Create a user

```
$ radmin mkuser <username>
```

### Create a LDAP user

```
$ radmin mkldapuser <username>
```

**_Note: An LDAP user doesn't have a password in Radon, the username should be 
the same than the ldap username. When an LDAP user try to log on Radon, it 
first fails as it doesn't exists in the Radon database but its username/password
is then checked on LDAP server._**

### List existing users

* List all users

```
$ radmin lu
```

* Display information on a specific user

```
$ radmin lu <username>
```

### Modify a user

```
$ radmin moduser <username> (email | administrator | active | password | ldap) [<value>]
```

For instance to change the email of the user user10

```
$ radmin moduser user10 email user10@indigo.com
```

### Remove a user

```
$ radmin rmuser <username>
```

### Create a group

```
$ radmin mkgroup <groupname>
```

### List existing groups

* List all groups

```
$ radmin lg
```

* Display information on a specific group

```
$ radmin lg <groupname>
```

### Add user(s) to a group

```
$ radmin atg <groupname> <userlist> ...
```

### Remove user(s) from a group

```
$ radmin rfg <groupname> <userlist> ...
```

### Remove a group

```
$ radmin rmgroup <groupname>
```

### List the content of a collection

```
$ radmin ls [<path>] [-a] [--v=<VERSION>]
```

* The '-a' option also shows the ACL associated to the collection

* The '-v' option specify a version for the collection

### Change the current working dir

```
$ radmin cd [<path>]
```

### Create a collection

```
$ radmin mkdir <path>
```

### Download a copy of a resource on the local filesystem

```
$ radmin get <src> [<dest>] [--force]
```

* If the dest is not specified the resource name is used

* The '--force' option can be used to replace an existing local file 

### Put a local file to Radon

```
$ radmin put <src> [<dest>] [--mimetype=<MIME>]
```

* If the dest is not specified the local filename is used

* The '--mimetype' option can be used to specify the mimetype of the data object

### Create a reference in Radon

```
$ radmin put --ref <url> <dest> [--mimetype=<MIME>]
```

* The '--mimetype' option can be used to specify the mimetype of the data object

### Display the current working dir

```
$ radmin pwd
```

### Remove a collection or a resource

```
$ radmin rm <path>
```



