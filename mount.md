
Using a bind mount, you can map the configuration file on your host computer to a specific location within the container.

Create a new directory called `public_html` on your host system.

```
mkdir public_html
```

Change the directory to `public_html` and create a file called `index.html` with the following content. This is a basic HTML document that creates a simple webpage that welcomes you with a friendly whale.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>My Website with a Whale & Docker!</title>
  </head>
  <body>
    <h1>Whalecome!!</h1>
    <p>Look! There's a friendly whale greeting you!</p>
    <pre id="docker-art">
   ##         .
  ## ## ##        ==
 ## ## ## ## ##    ===
 /"""""""""""""""""\___/ ===
{                       /  ===-
\______ O           __/
\    \         __/
 \____\_______/

Hello from Docker!
</pre
    >
  </body>
</html>
```

It's time to run the container. The `--mount` and `-v` examples produce the same result.

`-v`:

```
docker run -d --name my_site -p 8080:80 -v .:/usr/local/apache2/htdocs/ httpd:2.4
```

`--mount`:

```
docker run -d --name my_site -p 8080:80 --mount type=bind,source=./,target=/usr/local/apache2/htdocs/ httpd:2.4
```

You should be able to access the site via http://localhost:8080.
