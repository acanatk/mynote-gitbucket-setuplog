:orphan:

######################################################################
Apache Tomcat セットアップ
######################################################################

**********************************************************************
Tomcat用ユーザ追加
**********************************************************************

Tomcat用のユーザを追加します。

.. code-block:: bash

    $ useradd -s /sbin/nologin tomcat

**********************************************************************
ダウンロード
**********************************************************************

Apache Tomcatのダウンロードサイトから、バイナリパッケージのURLを確認します。

.. code-block:: text

    http://archive.apache.org/dist/tomcat/tomcat-8/

なお、本ドキュメントでは、次のバイナリパッケージを使用する場合を例に記述します。

* apache-tomcat-8.5.33.tar.gz

バイナリパッケージをダウンロードする。

.. code-block:: bash

    $ curl -O http://archive.apache.org/dist/tomcat/tomcat-8/v8.5.33/bin/apache-tomcat-8.5.33.tar.gz

**********************************************************************
インストール
**********************************************************************

ダウンロードしたファイルを /opt ディレクトリに展開します。

.. code-block:: bash

    $ cd /opt
    $ tar xvzf apache-tomcat-8.5.33.tar.gz

展開したディレクトリおよびファイルの所有者を tomcat ユーザに変更します。

.. code-block:: bash

    $ chown -R tomcat.tomcat apache-tomcat-8.5.33

**********************************************************************
設定
**********************************************************************

======================================================================
Tomcat サービスの登録
======================================================================

ユニットファイル（/etc/systemd/system/tomcat.service）を作成し、次の設定を行います。

.. code-block:: text

    [Unit]
    Description=Apache Tomcat 8
    After=network.target
    
    [Service]
    User=tomcat
    Group=tomcat
    Type=oneshot
    PIDFile=/opt/apache-tomcat-8.5.33/tomcat.pid
    RemainAfterExit=yes
    
    ExecStart=/opt/apache-tomcat-8.5.33/bin/startup.sh
    ExecStop=/opt/apache-tomcat-8.5.33/bin/shutdown.sh
    ExecReStart=/opt/apache-tomcat-8.5.33/bin/shutdown.sh;/opt/apache-tomcat-8.5.33/bin/startup.sh
    
    [Install]
    WantedBy=multi-user.target

.. tip::

    Tomcat の起動が遅い場合、上記ファイルの **Service** セクションに次の設定を追加してください。

    .. code-block:: text
    
        Environment=CATALINA_OPTS="-Djava.security.egd=file:/dev/./urandom"


ユニットファイルのアクセス権を変更します。 

.. code-block:: bash

    $ chmod 755 tomcat.service

Tomcat サービスの自動起動の設定をします。

.. code-block:: bash

    $ systemctl enable tomcat

.. tip::

    手動で Tomcat サービスを起動する場合

    .. code-block:: bash

        $ systemctl start tomcat

    手動で Tomcat サービスを停止する場合

    .. code-block:: bash

        $ systemctl stop tomcat

======================================================================
Apache HTTP Server と連携するための設定
======================================================================

Tomcatへのアクセスを Apache HTTP Server（以下、HTTP）を経由して行うための設定を行います。

----------------------------------------------------------------------
HTTP の設定
----------------------------------------------------------------------

| HTTP設定ファイルで、AJPが有効になっていることを確認します。
| /etc/httpd/conf.modules.d/00-proxy.confファイルに次の設定があることを確認します。

.. code-block:: text

    LoadModule proxy_module modules/mod_proxy.so
    LoadModule proxy_ajp_module modules/mod_proxy_ajp.so

----------------------------------------------------------------------
Tomcat の設定
----------------------------------------------------------------------

| Tomcat設定ファイルで、AJPプロトコルが有効になっていることを確認します。
| $CATALINA_HOME/conf/server.xmlファイルに次の設定があることを確認します。

.. code-block:: xml

    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

| Tomcat設定ファイルのHTTP(8080ポート)の設定を無効にします。
| $CATALINA_HOME/conf/server.xmlファイルの次の設定をコメントアウトします。

.. code-block:: xml

    <!--
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    -->
