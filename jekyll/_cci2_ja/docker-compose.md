---
layout: classic-docs
title: "Docker Compose のインストールと使用"
description: "プライマリコンテナで Docker Compose を有効化する方法"
version:
  - クラウド
  - Server v4.x
  - Server v3.x
  - Server v2.x
---

ここでは、 Docker Compose のインストール方法および使用方法を説明します。読者が `docker-compose` ユーティリティーを使用した経験があることを前提としています。

Docker Compose を初めて使う場合は、[公式の Docker Compose の概要](https://docs.docker.com/compose/)または[入門ガイド](https://docs.docker.com/compose/gettingstarted/)を参照してください。

* 目次
{:toc}

`docker-compose` ユーティリティーは、Machine Executor イメージや [CircleCI イメージにプリインストールされています]({{ site.baseurl }}/ja/circleci-images/#pre-installed-tools)。

Docker Executor を使用していて、CircleCI イメージを使用**していない**場合は、以下のコードを [`config.yml`]({{ site.baseurl }}/ja/configuration-reference/) ファイルに追加するとアクティブ化されるリモート Docker 環境を使用して、ジョブ実行時に Docker Compose を[プライマリコンテナ]({{ site.baseurl }}/ja/glossary/#primary-container)にインストールできます。

```yml
      - run:
          name: Install Docker Compose
          environment:
            COMPOSE_VERSION: '1.29.2'
          command: |
            curl -L "https://github.com/docker/compose/releases/download/${COMPOSE_VERSION}/docker-compose-$(uname -s)-$(uname -m)" -o ~/docker-compose
            chmod +x ~/docker-compose
            sudo mv ~/docker-compose /usr/local/bin/docker-compose

```

上記のコード例では、Executor で `curl` も 使用可能であることを想定しています。 独自の Docker イメージを構築する場合は、 [カスタム Docker イメージ]({{site.baseurl}}/ja/custom-images/)に関するドキュメントをお読みください。

次に、リモート Docker 環境をアクティブ化するために、`setup_remote_docker` ステップを追加します。

```yml
      setup_remote_docker
```

以下のステップにより、`docker-compose` コマンドをビルドイメージに追加できます。

```yml
      - run:
          name: Build images of services declared in docker-compose.yml
          command: docker-compose build
```

または、以下のステップで、システム全体を実行できます。

```yml
      - run:
          name: Start all services declared in docker-compose.yml
          command: docker-compose up -d
```

または、例えば以下により、サービスが実行されているかを確認できます。

```yml
      - run:
          name: Start docker-compose and verify service(s)
          command: |
            # Setting the Docker Compose project name to "circleci-demo-docker" means
            # the names of our services' containers would be prefixed with "circleci-demo-docker".
            docker-compose --project circleci-demo-docker up -d

            # In this example, we have a "contacts" service, and
            # we are trying to check, via `dockerize`, if the service is ready.
            docker container run --network container:circleci-demo-docker_contacts_1 \
              docker.io/jwilder/dockerize \
              -wait http://localhost:8080/healthcheck \
              -wait-retry-interval 2s \
              -timeout 20s
```

## サンプルプロジェクト
{: #example-project }

GitHub の [docker-compose サンプルプロジェクト](https://github.com/circleci/cci-demo-docker/tree/docker-compose)で、例を参照してください。また、[完全な設定ファイル](https://github.com/circleci/cci-demo-docker/blob/docker-compose/.circleci/config.yml)をお客様のプロジェクトのテンプレートとして利用できます。

**注:** プライマリコンテナは、リモート Docker とは独立した環境で動作し、両者は直接通信できません。 実行中のサービスとやり取りするためには、サービスのネットワーク内でコンテナを実行します。

## Docker Compose を Machine Executor で使用
{: #using-docker-compose-with-machine-executor }

Docker Compose を使用して Docker Compose ファイルを含むマルチコンテナのセットアップを管理するには、`.circleci/config.yml` ファイルで `machine` キーを使用し、`docker-compose` を通常どおりに使用します (詳細は[こちら]({{site.baseurl}}/ja/using-linuxvm)の Linux VM 実行環境に関するドキュメントを参照)。 つまり、Docker Compose ファイルがコンテナとローカル ディレクトリを共有する場合、予期したとおりに機能します。 詳細については、Docker ドキュメントの [最初の docker-compose.yml ファイル](https://docs.docker.com/get-started/part3/#your-first-docker-composeyml-file) を参照してください。


## Docker Compose を Docker Executor で使用
{: #using-docker-compose-with-docker-executor }

`docker` を `setup_remote_docker` と組み合わせて使用すると、docker-machine を使用して作成した場合と同様のリモートエンジンを作成できます。 ただし、このセットアップでは、ボリュームのマウントとポート転送は同じようには**機能しません**。 リモート Docker デーモンは、Docker CLI や Docker Compose とは異なるシステム上で動作するため、これを機能させるにはデータの移動が必要です。 マウントは通常、Docker ボリュームでコンテンツを利用可能にすることで解決できます。 `docker cp` を使用して、CLI ホストから Docker リモートホスト上で実行しているコンテナにデータを取得することで、Docker ボリュームにデータをロードできます。

デプロイ用の Docker イメージをビルドする場合は、この組み合わせが必要です。

## 制限事項
{: #limitations }

`docker-compose` の `macos` Executor での使用はサポートしていません。 詳細は[サポート記事](https://support.circleci.com/hc/en-us/articles/360045029591-Can-I-use-Docker-within-the-macOS-executor-)を参照してください。

## 関連項目
{: #see-also }
{:.no_toc}

例と詳細は、[Docker コマンドの実行についてのドキュメントのフォルダのマウントに関するセクション]({{ site.baseurl }}/ja/building-docker-images/#mounting-folders)を参照してください。
