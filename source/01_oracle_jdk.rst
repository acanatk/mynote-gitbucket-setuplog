:orphan:

**********************************************************************
Oracle JDK セットアップ
**********************************************************************

======================================================================
インストール
======================================================================

Oracle JDKのダウンロードサイトから、インストーラのURLを確認する。

.. code-block:: text

    http://www.oracle.com/technetwork/java/javase/downloads/index.html

なお、本ドキュメントでは、次のインストーラを使用する場合を例に記述します。

* jdk-8u131-linux-x64.rpm

インストーラをダウンロードする。

.. code-block:: bash

    $ curl -LO -H "Cookie: oraclelicense=accept-securebackup-cookie" \
           "http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm"

インストールする。

.. code-block:: bash

    $ yum localinstall jdk-8u131-linux-x64.rpm


======================================================================
設定ファイル変更
======================================================================

/etc/profileファイルの最終行に、次の環境変数を追加します。

.. code-block:: text

    export JAVA_HOME=/usr/java/default
    export PATH=$PATH:$JAVA_HOME/bin
    export CLASSPATH=.:$JAVA_HOME/jre/lib:$JAVA_HOME/lib:$JAVA_HOME/lib/tools.jar

======================================================================
インストール確認
======================================================================

**java** コマンドを実行して、次のようにバージョンが表示されれば正しくインストールされています。

.. code-block:: bash

    $ java -version
    java version "1.8.0_131"
    Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
    Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)

**javac** コマンドを実行して、次のようにバージョンが表示されれば正しくインストールされています。

.. code-block:: bash

    $ javac -version
    javac 1.8.0_131
