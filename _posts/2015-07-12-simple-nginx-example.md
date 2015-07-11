---
published: true
layout: post
categories: 
  - Tutorial
tags: 
  - nginx
  - docker
  - frontend
---


## Simple nginx example with docker
Prerequisites :
- you need to install [docker](https://github.com/tdeheurles/docs/blob/master/docker)


The nginx container is just 130mb, you can start pulling it with :

{% highlight bash %}
docker pull nginx:latest
{% endhighlight %}

Go/create the project directory and create a html index page :

{% highlight bash %}
echo "Hello from NginX" > index.html
{% endhighlight %}

Generate a nginx conf. We say that we want to serve on port 80 for the `/` location

{% highlight bash %}
cat <<EOF > site.conf
server {
  listen 80;
  location / {
    root /www;
  }
}
EOF
{% endhighlight %}

and the Dockerfile to run our website

{% highlight bash %}
cat <<EOF > Dockerfile
FROM        nginx:latest

RUN         rm /etc/nginx/conf.d/*.conf

WORKDIR     /usr/src

COPY        index.html   /www/
COPY        site.conf    /etc/nginx/conf.d/

ENTRYPOINT  nginx -g 'daemon off;'
EOF
{% endhighlight %}


Build the container

{% highlight bash %}
docker build -t my-website .
{% endhighlight %}


and run it

{% highlight bash %}
docker run -d -p 80:80 my-website > container_id
{% endhighlight %}


look if all is fine :

{% highlight bash %}
$ curl localhost
Hello from NginX
{% endhighlight %}

Then clean

{% highlight bash %}
$ cat container_id | xargs docker stop
9ddc387b297b
{% endhighlight %}

{% highlight bash %}
$ cat container_id | xargs docker rm
9ddc387b297b
{% endhighlight %}

{% highlight bash %}
$ docker rmi my-website
Untagged: my-website:latest
Deleted: 421067b514bf3f4b48cdb857b5115d1cb2170fd598a757f074e2ae069d0ea5bf
Deleted: 5570ce98d46297675aadbe5583a75b782e4dae45d332a20e0748df62dc86ad12
Deleted: 92f8b0503754cb577542d57d68c7be5988169ee73d5636e6cef36228c808f981
Deleted: 0acf6acd16f33ca34c8352606e668bf4cf0a340abf4502fab2a069075d27e389
Deleted: 190abb46cbc94826b6cdf587ea353bcc56851d8fca7392d2b254daa15b00d9dd
{% endhighlight %}
