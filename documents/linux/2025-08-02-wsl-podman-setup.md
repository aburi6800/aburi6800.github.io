---
layout: page
title:  "【podman】WSL2環境へのセットアップ"
date:   2025-08-02 00:00:00 +0900
categories: wsl2 podman
---

初版 2025/08/02  
改訂 2025/08/09

-----

<br>

## インストール

リポジトリから入れた。  

```
hitoshi@infinity:~$ sudo apt install podman
hitoshi@infinity:~$ podman --version
podman version 3.4.4
```

イメージのリストを取ると、警告が出た。  

```
hitoshi@infinity:~$ podman images
WARN[0001] "/" is not a shared mount, this could cause issues or missing mounts with rootless containers
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE
```

WSL2環境ではルートディレクトリがprivateであることが原因。  
以下コマンドを実行することで出なくなる。  

```
sudo mount --make-rshared /
```

実行結果：  
```
hitoshi@infinity:~$ sudo mount --make-rshared /
[sudo] hitoshi のパスワード:
hitoshi@infinity:~$ podman images
REPOSITORY  TAG         IMAGE ID    CREATED     SIZE
hitoshi@infinity:~$ podman ps -a
CONTAINER ID  IMAGE       COMMAND     CREATED     STATUS      PORTS       NAMES
```

`docker`コマンドの代替えとなるよう、シンポリックリンクを作成しておく。  

```
hitoshi@infinity:~$ ln -s $(which podman) ~/.local/bin/docker
hitoshi@infinity:~$ ls -al ~/.local/bin/docker
lrwxrwxrwx 1 hitoshi hitoshi 15  8月  2 11:15 /home/hitoshi/.local/bin/docker -> /usr/bin/podman
```

<br>

## Dockerハブを構成する

```
hitoshi@infinity:~$ mkdir -p ~/.config/containers
hitoshi@infinity:~$ > ~/.config/containers/registries.conf cat <<EOF
> unqualified-search-registries=["docker.io"]
>
> [aliases]
> "library"="docker.io/library"
> EOF
```

`>`の部分は直接入力部分。  

<br>

## 動作確認

軽量Webサーバ`nginx`のコンテナでテストする。

```
hitoshi@infinity:~$ podman run -d --name nginxtest -p 8080:80 library/nginx:1.23.3-alpine
Resolving "library/nginx" using unqualified-search registries (/home/hitoshi/.config/containers/registries.conf)
Trying to pull docker.io/library/nginx:1.23.3-alpine...
Getting image source signatures
Copying blob dbe1551bd73f done
Copying blob 86c5246c96db done
Copying blob 63b65145d645 done
Copying blob 8c7e1fd96380 done
Copying blob 0d4f6b3f3de6 done
Copying blob b874033c43fb done
Copying blob 2a41f256c40f done
Copying config 2bc7edbc3c done
Writing manifest to image destination
Storing signatures
3ec489e6b3fdf71f8a9c50e4d5c4195e28542d1dae80d0cd1bc4722b4ef0eef6
hitoshi@infinity:~$ curl -I http://localhost:8080
HTTP/1.1 200 OK
Server: nginx/1.23.3
Date: Sat, 02 Aug 2025 02:10:26 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 13 Dec 2022 18:23:05 GMT
Connection: keep-alive
ETag: "6398c309-267"
Accept-Ranges: bytes
```

ブラウザで`localhost:8080`にアクセス。  

![2025-08-02-wsl-podman-setup-01.png](/resources/2025-08-02-wsl-podman-setup-01.png)

OKのようなので、削除。  

```
hitoshi@infinity:~$ podman rm -f nginxtest
3ec489e6b3fdf71f8a9c50e4d5c4195e28542d1dae80d0cd1bc4722b4ef0eef6
```

<br>

## コンテナイメージを作成する

作業用ディレクトリ作成。  

```
hitoshi@infinity:~$ mkdir podman_test
hitoshi@infinity:~$ cd podman_test
```

Dockerfile作成。  

```
hitoshi@infinity:~/podman_test$ >Dockerfile cat <<EOF
> FROM library/alpine:3.17.2
>
> RUN echo hello > /tmp/hello.txt
> EOF
```

ファイルを確認。  

```
hitoshi@infinity:~/podman_test$ ls -al ./Dockerfile
-rw-r--r-- 1 hitoshi hitoshi 60  8月  2 11:19 ./Dockerfile
hitoshi@infinity:~/podman_test$ cat ./Dockerfile
FROM library/alpine:3.17.2

RUN echo hello > /tmp/hello.txt
```

コンテナをビルド。  

```
hitoshi@infinity:~/podman_test$ podman build -t mytestimage .
STEP 1/2: FROM library/alpine:3.17.2
Resolving "library/alpine" using unqualified-search registries (/home/hitoshi/.config/containers/registries.conf)
Trying to pull docker.io/library/alpine:3.17.2...
Getting image source signatures
Copying blob 63b65145d645 skipped: already exists
Copying config b2aa39c304 done
Writing manifest to image destination
Storing signatures
STEP 2/2: RUN echo hello > /tmp/hello.txt
COMMIT mytestimage
--> 156ec018bf1
Successfully tagged localhost/mytestimage:latest
156ec018bf19f08be9a8fc9e5a7e85f3d5f56fdb5032e35253f38c5ee51fa930
```

コンテナを実行。  

```
hitoshi@infinity:~/podman_test$ podman run --rm mytestimage cat /tmp/hello.txt
hello
```

<br>

## 作成したイメージをすべて削除

```
hitoshi@infinity:~/podman_test$ podman images -q | xargs podman rmi
Untagged: localhost/mytestimage:latest
Untagged: docker.io/library/nginx:1.23.3-alpine
Untagged: docker.io/library/alpine:3.17.2
Deleted: 156ec018bf19f08be9a8fc9e5a7e85f3d5f56fdb5032e35253f38c5ee51fa930
Deleted: 2bc7edbc3cf2fce630a95d0586c48cd248e5df37df5b1244728a5c8c91becfe0
Deleted: b2aa39c304c27b96c1fef0c06bee651ac9241d49c4fe34381cab8453f9a89c7d
```
