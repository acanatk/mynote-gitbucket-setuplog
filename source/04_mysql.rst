:orphan:

######################################################################
MySQL5.7 セットアップ
######################################################################

**********************************************************************
MariaDB関連パッケージのアンインストール
**********************************************************************

MariaDB関連パッケージをアンインストールします。

.. code-block:: bash

    $ yum remove mariadb-libs

**********************************************************************
MySQL オフィシャルリポジトリ追加
**********************************************************************

次のページで、Yum リポジトリ追加用 RPM パッケージを確認します。

.. code-block:: text

    https://dev.mysql.com/downloads/repo/yum/

なお、本ドキュメントでは、次の構成の RPM　パッケージを使用する場合を例に記述します。

* | Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package 
  | (mysql57-community-release-el7-11.noarch.rpm)

Yum リポジトリ を追加します。

.. code-block:: bash

    $ yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm

**********************************************************************
インストール
**********************************************************************

MySQLをインストールします。

.. code-block:: bash

    $ yum install mysql-community-server

**********************************************************************
設定
**********************************************************************

======================================================================
MySQL の root ユーザの初期パスワード確認
======================================================================

MySQLサービスを起動します。

.. code-block:: bash

    $ systemctl start mysqld

root ユーザの初期パスワードが自動生成されているので、MySQLログファイルで確認します。

.. code-block:: bash

    $ grep  "temporary password" /var/log/mysqld.log
    2017-06-21T14:13:09.684244Z 1 [Note] A temporary password is generated for root@localhost: hogehoge

| **"root@localhost:"** の後ろの **"hogehoge"** が自動生成された初期パスワードになります。
| ここに表示されるパスワードはインストール環境により異なります。
| あとでパスワードを変更するため、上記パスワードを書き留めておきます。

======================================================================
MySQL セキュリティー改善の設定
======================================================================

MySQL のセキュリティー改善の設定を行います。

**mysql_secure_installation** コマンドを実行します。

root のパスワードを聞かれるので、先のMySQLログファイルで確認したパスワードを入力します。

.. code-block:: text
    :emphasize-lines: 5

    $ mysql_secure_installation 

    Securing the MySQL server deployment.

    Enter password for user root: ********

新しく設定する root のパスワードを聞かれるので、任意のパスワードを入力します。

.. code-block:: text
    :emphasize-lines: 3,5

    The existing password for the user account root has expired. Please set a new password.

    New password: ********

    Re-enter new password: ********

**validate_password** プラグインが自動的にインストールされるので、root のパスワードを再入力します。

.. code-block:: text
    :emphasize-lines: 7,9,11,14

    The 'validate_password' plugin is installed on the server.
    The subsequent steps will run with the existing configuration
    of the plugin.
    Using existing password for root.

    Estimated strength of the password: 100 
    Change the password for root ? ((Press y|Y for Yes, any other key for No) : y

    New password: ********

    Re-enter new password: ********

    Estimated strength of the password: 100 
    Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y

匿名ユーザのアカウントを削除するか聞かれるので、 "y" を入力します。

.. code-block:: text
    :emphasize-lines: 8

    By default, a MySQL installation has an anonymous user,
    allowing anyone to log into MySQL without having to have
    a user account created for them. This is intended only for
    testing, and to make the installation go a bit smoother.
    You should remove them before moving into a production
    environment.

    Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
    Success.


リモートホストから root アカウントでのログインを禁止するか聞かれるので、 "y" を入力します。

.. code-block:: text
    :emphasize-lines: 5

    Normally, root should only be allowed to connect from
    'localhost'. This ensures that someone cannot guess at
    the root password from the network.

    Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
    Success.

    By default, MySQL comes with a database named 'test' that
    anyone can access. This is also intended only for testing,
    and should be removed before moving into a production
    environment.

テストデータベースを削除するか聞かれるので、 "y" を入力します。

.. code-block:: text
    :emphasize-lines: 1

    Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
    - Dropping test database...
    Success.

    - Removing privileges on test database...
    Success.

権限テーブルを再ロードするか聞かれるので、 "y" を入力します。

.. code-block:: text
    :emphasize-lines: 4

    Reloading the privilege tables will ensure that all changes
    made so far will take effect immediately.

    Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
    Success.

    All done! 

======================================================================
設定ファイルの変更
======================================================================

MySQLの次の設定を変更します。

* デフォルト文字コードを **UTF-8** に変更する。
* パスワードの有効期限を無効にする。

/etc/my.cnf ファイルの **mysqld** セクションに次の設定を追加します。

.. code-block:: text
    :emphasize-lines: 3-4

    [mysqld]

    character-set-server = utf8
    default_password_lifetime = 0

======================================================================
MySQLサービスの自動起動の設定
======================================================================

MySQLサービスの自動起動の設定をします。

.. code-block:: bash

    $ systemctl enable mysqld

