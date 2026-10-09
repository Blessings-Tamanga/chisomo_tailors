# Dockerizing — Chisomo Tailors

Static site served via `nginx:alpine` (minimal nginx image, ~20MB, no build step needed for plain HTML).

## Structure

```text
.
├── index.html
├── nginx.conf
└── Dockerfile
```

- `index.html` — the whole site (HTML + inline CSS/JS).
- `nginx.conf` — nginx server config (what folder to serve, which port).
- `Dockerfile` — recipe that packs the above into a runnable image.

## Dockerfile

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- `FROM nginx:alpine` — base image: lightweight, production-ready nginx.
- `COPY index.html .../html/` — puts your site where nginx looks for web content (`root`).
- `COPY nginx.conf .../default.conf` — replaces default server config with yours.
- `EXPOSE 80` — documents that the container listens on 80 (does not publish it alone).
- `CMD ["nginx", "-g", "daemon off;"]` — runs nginx in the foreground so the container stays alive.

`nginx.conf` serves `/usr/share/nginx/html` on port 80 with `try_files $uri $uri/ =404` (serve file or 404) and denies dotfiles.

## Build & Run

```bash
docker build -t chisomo-tailors .
docker run -d -p 8080:80 --name chisomo-site chisomo-tailors
```

- `docker build -t chisomo-tailors .` — builds the image; `-t` tags/names it, `.` is the build context (current dir, where the `Dockerfile` lives).
- `-d` — detached, runs in background so you keep your terminal.
- `-p 8080:80` — publish ports, `host:container` (`localhost:8080` → nginx on `80` inside the container).
- `--name chisomo-site` — container name for `stop`/`logs`/`rm`.
- `chisomo-tailors` — image to run (the one you just built).

## View

Open <http://localhost:8080>.

```bash
docker stop chisomo-site        # stop (keeps container, frees the port)
docker rm -f chisomo-site       # force-remove (needed to reuse --name on next run)
```
