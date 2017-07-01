######################################################################
GitBucket セットアップログ
######################################################################

**********************************************************************
事前準備
**********************************************************************

GitBucketで必要なソフトウェアをセットアップします。

* | Oracle JDKセットアップ
  | :doc:`./01_oracle_jdk` の手順に従ってセットアップします。

* | Apache HTTP Serverセットアップ
  | :doc:`./02_httpd` の手順に従ってセットアップします。

* | Apache Tomcatセットアップ
  | :doc:`./03_tomcat` の手順に従ってセットアップします。

* | MySQLセットアップ
  | :doc:`./04_mysql` の手順に従ってセットアップします。

**********************************************************************
インストール
**********************************************************************

GitBucketのダウンロードサイトから、warファイルのURLを確認する。

.. code-block:: text

    https://github.com/gitbucket/gitbucket/releases

なお、本ドキュメントでは、次のバージョンをインストール使用する場合を例に記述します。

* GitBucket 4.13

warファイルをダウンロードします。

.. code-block:: bash

    $ curl -LO https://github.com/gitbucket/gitbucket/releases/download/4.13/gitbucket.war

ダウンロードした war ファイルをTomcatの webapps ディレクトリにコピーします。

.. code-block:: bash

    $ cp gitbucket.war /opt/apache-tomcat-8.5.13/webapps/.

Tomcatを起動して、デプロイします。

.. code-block:: bash

    $ systemctl start tomcat


**********************************************************************
設定
**********************************************************************

======================================================================
HTTP の設定
======================================================================

Apache HTTP Server からGitBucketへアクセスするための設定をします。

/etc/httpd/conf.d/gitbucket.conf ファイルを作成し、次の設定を追加します。

.. code-block:: apacheconf

    <Location /gitbucket>
        ProxyPass ajp://localhost:8009/gitbucket
    </Location>

Apache HTTP Serverから接続できることを確認します。

HTTPサービスを再起動します。

.. code-block:: bash

    $ systemctl restart httpd

ブラウザから、次のURLで接続できることを確認します。

.. code-block:: text

    http://HOSTNAME/gitbucket


======================================================================
MySQL の設定
======================================================================

MySQLサービスを起動します。

.. code-block:: bash

    $ systemctl start mysqld

| **mysql** コマンドで、MySQLにrootアカウントで接続します。
| パスワードには、 :doc:`./04_mysql` で設定したものを入力してください。

.. code-block:: bash
    :emphasize-lines: 2

    $ mysql -u root -p
    Enter Password: ********
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 3
    Server version: 5.7.18 MySQL Community Server (GPL)

    Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql>

次の設定で、GitBucket用データベースを作成します。

.. list-table::
    :stub-columns: 1

    * - データベース名
      - gitbucket
    * - アカウント
      - gitbucket
    * - パスワード
      - *任意のパスワード*

.. code-block:: mysql
    :emphasize-lines: 1,4,7

    mysql> create database gitbucket;
    Query OK, 1 row affected (0.09 sec)

    mysql> grant all on gitbucket.* to gitbucket@localhost identified by '********';
    Query OK, 0 rows affected, 1 warning (0.50 sec)

    mysql> flush privileges;
    Query OK, 0 rows affected (0.04 sec)

    mysql> exit;
    Bye

======================================================================
GitBucket の設定
======================================================================

----------------------------------------------------------------------
Tomcatサービスを停止
----------------------------------------------------------------------

Tomcatサービスを停止します。

.. code-block:: bash

    $ systemctl stop tomcat

----------------------------------------------------------------------
H2用データベースファイルの削除
----------------------------------------------------------------------

tomcat ユーザのホームディレクトリに、次の **GitBucket** データディレクトリがあります。

.. code-block:: text

    ~tomcat/.gitbucket

このフォルダに次のファイルがあるので削除します。

* data.mv.db

----------------------------------------------------------------------
データベースをMySQLに変更
----------------------------------------------------------------------

**GitBucket** データディレクトリに、次のデータベース設定ファイルがあります。

.. code-block:: text

    ~tomcat/.gitbucket/database.conf

このファイルには、H2用に設定されてあるので、MySQL用に次のように変更します。

.. code-block:: text
    :emphasize-lines: 2-4

    db {
        url = "jdbc:mysql://localhost/gitbucket?useUnicode=true&characterEncoding=utf8"
        user = "gitbucket"
        password = "********"
    }

Tomcatサービスを起動します。

.. code-block:: bash

    $ systemctl start tomcat

ブラウザから、次のURLで接続します。

.. code-block:: text

    http://HOSTNAME/gitbucket

MySQL のGitBucket用データベースに接続します。

.. code-block:: bash

    $ mysql -u root -p gitbucket
    Enter password: 
    Reading table information for completion of table and column names
    You can turn off this feature to get a quicker startup with -A

    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 16
    Server version: 5.7.18 MySQL Community Server (GPL)

    Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

    mysql> 

**show tables** を実行して、GitBucket用の各種テーブルが作成されていることを確認します。

.. code-block:: mysql

    mysql> show tables;
    +----------------------------------+
    | Tables_in_gitbucket              |
    +----------------------------------+
    | ACCESS_TOKEN                     |
    | ACCOUNT                          |
    | ACTIVITY                         |
    | COLLABORATOR                     |
    | COMMIT_COMMENT                   |
    | COMMIT_STATUS                    |
    | DEPLOY_KEY                       |
    | GROUP_MEMBER                     |
    | ISSUE                            |
    | ISSUE_COMMENT                    |
    | ISSUE_ID                         |
    | ISSUE_LABEL                      |
    | ISSUE_OUTLINE_VIEW               |
    | LABEL                            |
    | MILESTONE                        |
    | PLUGIN                           |
    | PROTECTED_BRANCH                 |
    | PROTECTED_BRANCH_REQUIRE_CONTEXT |
    | PULL_REQUEST                     |
    | REPOSITORY                       |
    | SSH_KEY                          |
    | VERSIONS                         |
    | WEB_HOOK                         |
    | WEB_HOOK_EVENT                   |
    +----------------------------------+
    24 rows in set (0.00 sec)

    mysql> 

----------------------------------------------------------------------
GitBucket管理者(root)パスワードの変更
----------------------------------------------------------------------

| GitBucket管理者(root)のパスワードがデフォルトで「root」に設定されているので、
| 任意のパスワードに変更します。


以上で、GitBucketのセットアップは終了です。