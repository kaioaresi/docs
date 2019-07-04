# Boas praticas ao escrever um `Dockerfile`
---

Criar um `Dockerfile` é um tarefa simples, porém você deve se atentar a alguns ponto importante que serão bem importantes para a saúde e manutenibilidade da imagem posteriormente. Primeiramente vamos falar sobre algumas boas praticas ao escrever o `Dockerfile` e posteriormente algumas dicas extras.


## 1 - Utilize `multistage` sempre que possível

```
FROM nginx:1.15.5
WORKDIR /usr/src/app
COPY package*.json ./
RUN apt-get update && \
    apt-get install node
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build:${NPM_ENV}
COPY /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```


```
# Stage para compilação
FROM node:8 as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.15.5
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```

## 2 - Utilize imagens oficiais sempre que possível e `-slim` | `alpine`

```
# Stage para compilação
FROM node:8 as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.15.5
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```

```
# Stage para compilação
FROM node:slim as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.17-alpine
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```

## 3 - Adcione metadados, informações e comentarios à image `LABEL` e `#`, seja breve nos comentarios

> Obervação: A tag `MAINTAINER` está deprecated, Utilize `LABEL`

```
# Stage para compilação
LABEL version="1.0.0" description="Aqui eu instalo diversos pacote de depencia do node e gero o pacote" maintainer="Queen Daenerys Stormborn of the House Targaryen, the First of Her Name, Queen of the Andals, the Rhoynar and the First Men, The rightful Queen of the Seven Kingdoms and Protector of the Realm, Queen of Dragonstone, Queen of Meereen, Khaleesi of the Great Grass Sea, the Unburnt, Breaker of Chains and Mother of Dragons,regent of the realm <dtargaryen@bionexo.com>"
FROM node:slim as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.17-alpine
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```

```
# Stage para compilação
LABEL version="1.0.0" description="Disponibilizando pacote node" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
FROM node:slim as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.17-alpine
LABEL version="1.0.0" description="Disponibilizando aplicação frontend" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```

## 4 - Utilize o `.dockerignore`

Coloque na image apenas os arquivos necessário, para evitar que a image fique cheia de arquivos desnecessários.

## 5 - Mapeie volumes, portas e diretorio de trabalho


```
# Stage para compilação
LABEL version="1.0.0" description="Disponibilizando pacote node" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
FROM node:slim as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.17-alpine
LABEL version="1.0.0" description="Disponibilizando aplicação frontend" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
# Copiando pacotes gerado no stage frontend
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
EXPOSE 80
```

```
# Stage para compilação
LABEL version="1.0.0" description="Disponibilizando pacote node" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
FROM node:slim as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.17-alpine
LABEL version="1.0.0" description="Disponibilizando aplicação frontend" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
# Copiando pacotes gerado no stage frontend
WORKDIR /usr/share/nginx/html
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
VOLUME /usr/src/app/dados
EXPOSE 80
```

## 6 - Construa o minimo de camadas possíveis

Organize o stage de sua imagem, para que seja feito o minimo de camadas possível, defina um processo

EX:

- Instalações
- Copias
- Compilação
- Alterações em arquivos
- Definiçoes

```
# Stage para compilação
LABEL version="1.0.0" description="Disponibilizando pacote node" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
FROM node:slim as frontend
WORKDIR /usr/src/app
COPY package*.json ./
RUN npm install
COPY src/ ./src/
COPY public/ ./public/
RUN npm run build

# Stage da aplicação
FROM nginx:1.17-alpine
LABEL version="1.0.0" description="Disponibilizando aplicação frontend" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
# Copiando pacotes gerado no stage frontend
WORKDIR /usr/share/nginx/html
COPY --from=frontend /usr/src/app/build/ /usr/share/nginx/html
VOLUME /usr/src/app/dados
EXPOSE 80
```

```
# Stage para compilação
FROM node:slim as frontend
LABEL version="1.0.0" description="Disponibilizando pacote node" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
WORKDIR /usr/src/app
RUN npm install && \
    apt update && \
    apt install python
COPY package*.json ./ && \
    src/ ./src/ && \
    public/ ./public/
RUN npm run build

FROM nginx:1.15.5
LABEL version="1.0.0" description="Disponibilizando aplicação frontend" maintainer="Daenerys Targaryen<dtargaryen@bionexo.com>"
WORKDIR /usr/share/nginx/html
# Copiando pacotes gerado no stage frontend
COPY --from=frontend /usr/src/app/build/ .
VOLUME /usr/src/app/dados
EXPOSE 8080
```

---
# Referencias

- [Melhores praticas dockerfile](https://www.mundodocker.com.br/melhores-praticas-dockerfile/)
- [Dockerfile best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
- [Dockerfile referencias](https://docs.docker.com/engine/reference/builder/)
- [Dicas de imagens](https://churrops.io/2017/09/08/docker-construindo-suas-imagens-com-dockerfile/)
- [Criando meu Dockerfile](https://blog.matheuscastiglioni.com.br/criando-minha-primeira-imagem-com-docker/)
- [O que e dockerfile](https://www.mundodocker.com.br/o-que-e-dockerfile/)
- [Maintainer](https://docs.docker.com/engine/reference/builder/#maintainer-deprecated)
