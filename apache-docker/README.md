## Running Apache in a Docker container
The source code in this subdirectory is not directly part of the `scarlet` webapplication but can be used as a convenient replacement for part of the setup instructions for getting it to run.

More information on this subproject can be found in the blog post:
https://www.sentiatechblog.com/[FIXME TODO]


By following the steps in this readme you can create a Docker container that runs Apache webserver already configured for the scarlet webapplication. This is an alternative to installing and configuring Apache on your development machine.

However, you still must edit your `hosts` file and add the entries for the URL's that are used in the scarlet project.

This instruction assumes that you have docker installed on your development machine.

## An extremely short course in running Docker
Docker works by downloading or creating an `image`. The result is a file on your local file system, managed by Docker. After obtaining the image, you `docker run` it. This creates a `container` and starts it up, resulting in a running application. 

You can now interact with the application that runs in the Docker container. If your interactions result in a change in one of the files in the container, for instance if you add a record to a database, we say that the *state* of the container is *changed*. 

After the container stops running it still exists with its state.  However, if you now `run` the `image` again, you get a *new and fresh* instance of the container with an original state. If you want to run an existing `container` again with its preserved state, you must **`start` the container** instead of running the image.

## Building and running the Docker container
The `Dockerfile` in this apache-docker directory downloads a very stripped down `linux alpine` Docker image from the public Docker repository, installs the Apache webserver on to it and copies Apache configuration files to the image. The final `CMD` instruction in the Dockerfile tells Docker to start running apache in the continer when it is started.

The Docker image that we need to run Apache is made with this command:
```
   docker build -t dimario/apache:1.0 .
```
**Note** there is a dot at the end of this command.

You can create a container from this image and run it with this command (but fill in your own IP):
```
docker run -it --name rundfunk --hostname rundfunk -e DEVHOST=192.168.1.15 -p:80:80 -p:443:443 dimario/apache:1.0
```
* in the **-e DEVHOST** argument you must fill in the IP address of your own development machine. This environment variable is patched into the Apache configuration inside the container.

## Troubleshooting
The **--name rundfunk** argument in the `docker run` names the container. After terminating, that name is still in use because the terminated container still exists. If you try a second `docker run` using this same name you will get an "name already in use" error. Istead, you can either restart the terminated container:
```
   docker start -it rundfunk
```
or delete it:
```
   docker rm rundfunk
```
and then re-run the image (which creates a fresh container named "rundfunk").

You can log on to the running container using this command:
```
   docker exec --it rundfunk /bin/sh
```
The Apache configuration (including fake SSL certificate and key) is located at `/etc/apache2/`. Apache logfiles are in `/var/log/apache2`  and some more Apache related files can be found in `/var/www`.

