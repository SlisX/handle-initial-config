# handle-initial-config

## synopsis

When you create a set of packages in order to built services on 
top of a debian dist, you may have to configure your services 
automatically through a set of scripts and a config file.

It becomes complex when you want your service to be included in 
a debian package. For example, you need to configure a database. 
But on a fresh install, the DB shall be inactive. Same thing for ldap.

A usual way to do that would be to have configuration scripts as 
executables in the debian package. They shall be launched manually,
or the postinst should ask wether or not he must launch the config
when in interactive mode.

Or you should use handle-initial-config

The idea was given by the Freexian Team when they worked on one of
the project I used with debian. 

In postinst script, you shall launch a config handler. This config handler
see if the configuration is possible (is the database launched ? The ldap
launched or populated ? ), if it is needed (don't launch the script if the 
config is already done). If the config is possible *and* needed, the do it !

## how to

Imagine you want to have a coffee service on you machine.
In order to use handle-initial config you should have a coffee-config script

It's structure should be
```shell
#!/bin/sh
case "$1" in 
    possible)
			# return 0 if possible, 1 if not
  	;;
    needed)
    		# return 0 if possible, 1 if not
        # if config is idempotent, then Always needed 
        # just do 'exit 0'
        exit 0
    ;;
    configure)
			coffee --initDB || [ "$?" = "1" ] ## $?=1 => db exist
    ;;
    *)
			echo "usage: $0 {configure|possible|needed}" >&2
			exit 1
    ;;
esac

```

In your postinst file you just have to call
handle-initial-config /usr/sbin/coffee-config 30
where 30 is the priority level 
(the smallest number are runned first)

Here the priority is 30 because the DB was initialised by another script 
(with priority 20)

handle-initial-config will try to configure the coffee service. If it's not possible, 
it will launch a daemon (hicd - handle initial config daemon) that will periodically try to
configure the coffee service.

The debian package will install cleanly, and configuration is made when possible.