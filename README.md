# AWS-Docker-Apache_and_Flask

![](https://private-user-images.githubusercontent.com/108736814/288289725-829bc2b6-95a7-479e-84a6-cd648847dfd1.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTEiLCJleHAiOjE3MDE4NDA1MDksIm5iZiI6MTcwMTg0MDIwOSwicGF0aCI6Ii8xMDg3MzY4MTQvMjg4Mjg5NzI1LTgyOWJjMmI2LTk1YTctNDc5ZS04NGE2LWNkNjQ4ODQ3ZGZkMS5wbmc_WC1BbXotQWxnb3JpdGhtPUFXUzQtSE1BQy1TSEEyNTYmWC1BbXotQ3JlZGVudGlhbD1BS0lBSVdOSllBWDRDU1ZFSDUzQSUyRjIwMjMxMjA2JTJGdXMtZWFzdC0xJTJGczMlMkZhd3M0X3JlcXVlc3QmWC1BbXotRGF0ZT0yMDIzMTIwNlQwNTIzMjlaJlgtQW16LUV4cGlyZXM9MzAwJlgtQW16LVNpZ25hdHVyZT03MDE1NTI4MmQwZTZmN2EyZDNjMmYwOTI1MzI2Njc2MGYwZTVkYjcwNjZjOTgwOWU1Zjg0ZGI3Y2MzYjljMWIyJlgtQW16LVNpZ25lZEhlYWRlcnM9aG9zdCZhY3Rvcl9pZD0wJmtleV9pZD0wJnJlcG9faWQ9MCJ9.kCmuLxeG-14_V1p_2eu5BM9qriOIye-vDMIwAGUCUFM)



## 必要なコンポーネント
- Docker: コンテナ化技術を用いてアプリケーションの環境を構築します。
- Flask: Pythonで書かれた軽量なウェブフレームワークです。
- Apache: 信頼性の高いウェブサーバーで、ここではFlaskアプリケーションのリバースプロキシとして使用します。

## 1. Flaskアプリケーションの準備

まずは、シンプルなFlaskアプリケーションを作成します。以下のコードを `app.py` というファイル名で保存してください。

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```


## 2. Dockerfileの作成

FlaskアプリケーションをDockerコンテナ内で実行するためのDockerfileを作成します。以下の内容を `Dockerfile` として保存します。

```Dockerfile
FROM python:3.8
WORKDIR /usr/src/app
COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 5000
CMD ["flask", "run"]
```


## 3. Docker Composeの設定

Docker Composeを使用して、FlaskアプリケーションとApacheサーバーを連携させます。以下の内容を `docker-compose.yml` ファイルに記述します。

```yaml
version: '3'

services:
  flask-app:
    build:
      context: ./flask
      dockerfile: Dockerfile
    ports:
      - "5000:5000"
    volumes:
      - ./flask:/usr/src/app
    environment:
      - FLASK_RUN_HOST=0.0.0.0
      - FLASK_APP=app.py

  apache:
    image: httpd
    stdin_open: true
    tty: true
    privileged: true
    ports:
      - "80:80"
    depends_on:
      - flask-app
    volumes:
      - ./apache/my-httpd.conf:/usr/local/apache2/conf/extra/httpd-vhosts.conf
      - ./apache/httpd.conf:/usr/local/apache2/conf/httpd.conf
```


## 4. Apacheの設定ファイル

Apacheの設定ファイル `httpd.conf` を用意します。主な内容は以下の通りです。

```apacheconf
...

# VirtualHost設定
<VirtualHost *:80>
    ProxyRequests Off
    ProxyPass / http://flask-app:5000/
    ProxyPassReverse / http://flask-app:5000/
</VirtualHost>
```


## 5. 環境のビルドと実行

すべての設定が完了したら、以下のコマンドを実行して環境をビルドし、コンテナを起動します。

```bash
sudo docker compose up --build
```


## 6. 動作確認

ブラウザまたはcurlコマンドを使用して、アプリケーションが正しく動作しているか確認します。

```bash
curl http://localhost
```



正常に動作すれば、「Hello, World!」と表示されます。---

以上が、Dockerを用いてFlaskアプリケーションとApacheサーバーを構築する基本的な手順です。このガイドが皆さんの開発作業に役立つことを願っています。
