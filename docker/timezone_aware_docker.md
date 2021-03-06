## Time Zone Aware Docker

If you are using functions such as `datetime.now()` in your python scripts and you want this to be consistent in your docker containers.
You can specify the timezone of your docker container.

Note `datetime.now()` provides an option to pass the timezone but if None is specified it uses the local timezone. In the case of a docker container
the local timezone is always UTC.

To change the timezone you can add the following to your dockerfile:

```docker
ENV TZ=Europe/London # specify timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone # update timezone from UTC to London time zone
```
