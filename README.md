# Docker Glassfish 2.1.1

[![Docker Stars](https://img.shields.io/docker/stars/jfloff/docker-glassfish-2.1.1.svg)][hub]
[![Docker Pulls](https://img.shields.io/docker/pulls/jfloff/docker-glassfish-2.1.1.svg)][hub]

[hub]: https://hub.docker.com/r/jfloff/docker-glassfish-2.1.1/

Docker image for projects that rely on **[Glassfish](https://javaee.github.io/glassfish/) 2.1.1**. Why such an old version? Well, don't ask me - [I just need it](https://github.com/jfloff/mrubis). :unamused:

First, finding this version was already hard! The oldest release on the [official repo](https://github.com/javaee/glassfish/releases) is 3.0. Thanks to @d10xa and his [repo](https://github.com/d10xa/docker-glassfish-2.1.1) I was able to find a download link. Second, due to some JDK version problems, I had to manually install a specific JDK version (`6u31`) from Oracle's binary.

The good news is that Glassfish comes with an old Apache Ant version, we avoid installing a newer version of Ant that would require Java 7 or 8, and most importantly we avoid any possible compatability problems with legacy code.

Binaries for JDK and Glassfish are downloaded from 3rd-party websites. This includes an external FTP that has the correct JDK version that Glassfish 2.1.1 needs. Even though its MD5-checked, if you prefer, download the binaries from Oracle's website (registration needed) and modify the Dockerfile accordingly.

There's two key environment variables set up:
- `$GLASSFISH_HOME` points to the Glassfish install directory
- `$GLASSFISH_CLASSPATH` points to all the Java libs that Glassfish comes with. You can use it to run your applications like so: `java -cp $GLASSFISH_CLASSPATH:app.jar foo.bar.App`

As usual in Glassfish you can access its admin panel at `localhost:4848` with username `admin` and password `adminadmin`. Other typical ports are also [exposed](DockerfileS#L91) in case you need them.

### Usage
If you are not familiar with Docker, or just forget the commands all the time like me, here is a resumé:
```
# build from this repo's Dockerfile
docker build --rm -t jfloff/glassfish-2.1.1 .

# run linking to the admin panel and main page
docker run --rm -p 4848:4848 -p 8080:800 -ti jfloff/glassfish-2.1.1

# Use in Dockerfile
FROM jfloff/glassfish-2.1.1
```

### Contribute
Feel free to contribute to this repo in any way you find fit.

### License
The code in this repository, unless otherwise noted, is MIT licensed. See the `LICENSE` file in this repository.
