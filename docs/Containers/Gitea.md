# Gitea

Gitea is a self-hosted software development service similar to GitHub, Bitbucket and GitLab. The IOTstack implementation runs as a pair of containers using a MariaDB database as the back-end.

## references { #references }

* [Gitea Home](https://about.gitea.com)
* [Gitea on Dockerhub](https://hub.docker.com/r/gitea/gitea)
* [Gitea documentation](https://docs.gitea.com)
* [GitHub](https://github.com/go-gitea/gitea)

## environment variables  { #envVars }

Environment variables need to be set in several stages:

1. **Before** you start the container for the first time, you should define the following environment variables. If you make a mistake or change your mind later, the best course of action is to start over from a [clean slate](#cleanSlate):

	* `GITEA_DB_NAME` is the name of the database that Gitea (the service) will use to store its information in MariaDB. Example:

		``` console
		echo "GITEA_DB_NAME=gitea" >>~/IOTstack.env
		```

		If omitted, defaults to "gitea".

	* 	`GITEA_DB_USER` is the name of the user that Gitea (the service) will use to authenticate with MariaDB. Example:

		``` console
		echo "GITEA_DB_USER=gitea" >>~/IOTstack.env
		```

		If omitted, defaults to "gitea".

	* `GITEA_DB_PASSWORD` is the password associated with the above user. Example:

		``` console
		$ echo "GITEA_DB_PASSWORD=$(uuidgen)" >>~/IOTstack.env
		```

		If omitted, the container will not start.

	* `GITEA_DB_ROOT_PASSWORD` is the administative password for the MariaDB service. Keep in mind that the `gitea_db` service is dedicated to Gitea. You can run other MariaDB instances in parallel. They will not interfere with each other and neither will they share data or credentials. Example:

		``` console
		$ echo "GITEA_DB_ROOT_PASSWORD=$(uuidgen)" >>~/IOTstack.env
		```

		If omitted, the container will not start. See [note below](#rootpw).

	You (the human user) will **never** need to know the username and passwords set here. You will not need to use these values in practice.

2. **After** you have set the environment variables listed above, start the container:

	``` console
	$ cd ~/IOTstack
	$ docker compose up -d gitea
	```

	If this is the first time you have launched Gitea, docker compose will also build and run the `gitea_db` service.

	You can expect to see the following warning:

	```
	WARN[0000] The "GITEA_SECRET_KEY" variable is not set. Defaulting to a blank string.
	```

	This is actually a reminder to execute this command:

	``` console
	$ echo "GITEA_SECRET_KEY=$(docker exec gitea gitea generate secret SECRET_KEY)" >>~/IOTstack/.env
	```

	After that command has run, start the container again:

	``` console
	$ docker compose up -d gitea
	```

	The warning message will go away.

	See [Managing Deployments With Environment Variables](https://docs.gitea.com/installation/install-with-docker#managing-deployments-with-environment-variables) for more information.

3. The `GITEA_ROOT_URL` environment variable should be set to the URL that the **user** uses to reach the Gitea service. If you use a proxy host such as Nginx then this would be the URL you present to the proxy. For example:

	``` console
	$ echo "GITEA_ROOT_URL=https://gitea.my.domain.com >>~/IOTstack.env
	```

	Alternatively, if you connect directly to the host on which the service is running, the URL will be that of the host plus the external port of the Gitea container. For example:

	``` console
	$ echo "GITEA_ROOT_URL=http://host.my.domain.com:7920 >>~/IOTstack.env
	```

	If omitted, defaults to null in which case the container will make a best-efforts determination (which is unlikely to be correct). You will also see this warning:

	```
	WARN[0000] The "GITEA_ROOT_URL" variable is not set. Defaulting to a blank string.
	```

	You can change this variable whenever you like. Simply edit the value in `~/IOTstack/.env` and apply the change by running:

	``` console
	$ docker compose up -d gitea
	```

	See [Gitea Server](https://docs.gitea.com/next/administration/config-cheat-sheet#server-server) for more information.

### database root password  { #rootpw }

At the time of writing (April 2025), the MariaDB instance was not respecting the environment variable being used to pass the root password into the container.

> See [MariaDB issue 163](https://github.com/linuxserver/docker-mariadb/issues/163)

You can ensure that the root password is set by running the following command:

``` console
$ docker exec gitea_db bash -c 'mariadb-admin -u root password $MYSQL_ROOT_PASSWORD'
```

If this command returns an error, it means that the root password was already set (presumably because Issue 163 has been resolved).

If this command succeeds without error, it means that the root password was not set but is now set.

Also notice that you did not need to know or copy/paste the root password to run the above command. It was sufficient to know the name of the environment variable containing the database root password.

## default ports

The IOTstack implementation listens on the following ports:

* 7920 the Gitea graphical user interface 
* 2222 the SSH passthrough service

## getting started

Use your browser to connect to the Gitea service, either:

* directly:

	```
	http://«host»:7920
	```

	where `«host»` is:

	- an IP address (eg 192.168.1.10)
	- a hostname (eg `iot-hub`
	- a domain name (eg `iot-hub.my.domain.com`)
	- a multicast domain name (eg `iot-hub.local`)

* indirectly, via a reverse proxy:

	```
	https://gitea.my.domain.com
	```

	This assumes that the reverse proxy redirects the *indirect* form (using HTTPS) to one of the *direct* forms (using HTTP).

Click on the <kbd>Register</kbd> button to create an account for yourself.

After that, please rely on the [Gitea documentation](https://docs.gitea.com).

## launch times

When you start the `gitea` service, docker compose auto-starts the `gitea_db` service (a MariaDB aka MySQL implementation). The database service can take some time to start and that, in turn, affects the availability of the `gitea` service.

The time it takes for the `gitea` service to become fully available depends on your hardware (CPU speed, RAM, SD/HD/SSD). As an example, the `gitea` service takes about 30 seconds to become available on a 4GB Raspberry Pi 4 with SSD.

You may get strange error messages if you attempt to connect to `gitea` while it is still coming up.

The moral is: be patient!

## starting over from a clean slate  { #cleanSlate }

Proceed as follows:

``` console
$ cd ~/IOTstack
$ docker compose down gitea gitea_db
$ sudo rm -rf ./volumes/gitea
$ docker compose up -d
```

In this situation, you should also regenerate the secret key:

``` console
$ echo "$(docker exec gitea gitea generate secret SECRET_KEY)"
```

> The reason for wrapping the command in an `echo` is because the `generate` command does not terminate the line so the value of the key has a tendency to run into the next Linux prompt.

Copy the value that is returned to the clipboard, then edit `~/IOTstack.env` to replace the right hand side of `GITEA_SECRET_KEY` with whatever is on the clipboard. Save your work then start the container again:

``` console
$ docker compose up -d
```

## container maintenance

You can maintain the Gitea container with normal `pull` commands:

``` console
$ cd ~/IOTstack
$ docker compose pull gitea
$ docker compose up -d gitea
$ docker system prune -f
```

The Gitea_DB container needs special handling:

``` console
$ cd ~/IOTstack
$ docker-compose build --no-cache --pull gitea_db
$ docker compose up -d gitea_db
$ docker system prune -f
```

