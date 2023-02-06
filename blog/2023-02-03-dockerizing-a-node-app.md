# 容器化 nginx 與 React 項目

## 由來

要發布 React 項目之前需要 build 該項目，並透過網頁伺服器如 apache、nginx 來 serve 靜態網頁，可以透過 docker 把這些操作自動化並且產生 docker image，後續就可以用每次產生的 image 來部署服務，[範例項目放在 github](https://github.com/xralphack/dockerize-nginx-react "xralphack/dockerize-nginx-react") 可以搭配本文閱讀

## 生成一個 React app

```
npx create-react-app my-app
```

該樣板已經提供了指令(於 package.json 中的 scripts)，執行 yarn build 即可生成檔案

新增 .dockerignore 檔案

```
.git
node_modules
build
```

## 用來 build 項目的 Dockerfile

```
# 要 build React 項目需要有 node.js
FROM node:16

# 準備好項目的路徑
WORKDIR /usr/src/app

# 複製項目的所有內容
# 在項目中新增 .dockerignore 用來忽略 .git node_modules build ...
COPY . .

# 安裝 npm modules
RUN yarn

RUN npx browserslist@latest --update-db && yarn build

```

## 用 nginx 來 serve 這個項目

Dockerfile 可以用多個 `FROM`，最後一個 `FROM` 的 image 會作為此容器的基礎 image，因此我會把第一個 node 的容器作為 build 項目的環境，再把產生出的內容複製到第二個容器的指定路徑

```
# 取別名
FROM node:16 as builder

# nginx 的 docker image
FROM nginx:alpine

# 把 node 容器中 build 好的內容複製到 nginx 容器中
COPY --from=builder /usr/src/app/build /usr/share/nginx/html

# 不要用 daemon 模式執行 nginx，這樣可以讓 docker 蒐集 log
CMD ["nginx", "-g", "daemon off;"]
```

## 使用到的命令

docker build 需要指定一個 Dockerfile，會產出一個 docker image，用 -t 是給這個 image 命名(tag name)

```
docker build . -t my-app
```

透過 docker run 可以把 image 跑起來，用 -d 代表 daemon

```
docker run -d my-app
```

容器跑起來之後，如果沒事做會關掉，為了讓容器一直執行著，可以讓其執行一個命令

```
docker run -d my-app tail -f /dev/null
```

列出正在執行的容器(加上 -a 可以列出所有容器)

```
docker ps
```

可以進入到容器中看看，有些容器沒有 bash 可以用，可以試試看 sh

```
docker exec -it CONTAINER_ID bash
```

刪除容器(-f 可以把正在執行的容器也刪除)

```
docker rm -f CONTAINER_ID
```

nginx 容器預設會使用 80 port 作為 web server 的 port，透過 port mapping 可以把本地的 port 跟容器 port 做對應

```
docker run -p 8080:80 -d my-app
```

可以用瀏覽器看到 localhost:8080 會出現 React 的預設畫面

如果要看 nginx 的 log

```
docker logs CONTAINER_ID
```

做實驗可能會使用到很多系統空間，如果想要快速清理被使用的空間

```
docker system prune -a
```

## 參考資料

https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
