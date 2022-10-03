# Web App

```bash
cd web
# --- build image
docker build . -t zoidenberg/otus-2-web:latest
# --- run container and brows to http://localhost:8000/homework.html
docker run --rm -p 8000:8000 zoidenberg/otus-2-web:latest
# --- upload image
docker push zoidenberg/otus-2-web:latest
```
