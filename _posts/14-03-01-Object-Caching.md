---
title: オブジェクトキャッシュ
isChild: true
anchor:  object_caching
---

## オブジェクトキャッシュ {#object_caching_title}

コード内の個々のオブジェクトをキャッシュできれば便利なこともある。
持ってくるのにコストがかかるデータや、結果がほとんど変わらないデータベースへの問い合わせなどだ。
オブジェクトキャッシュ用のソフトウェアを使えば、こういったデータをメモリ上に保持でき、
その後のアクセスの高速化につながる。
一度取得したデータをどこかに格納しておいて、それ以降のリクエストではそこから直接取り出す。
そうすればパフォーマンスは劇的に向上するし、データベースサーバーにかかる負荷も減らせる。

バイトコードキャッシュ用ソリューションの多くはカスタムデータも同様にキャッシュできるので、さらに活用できる。
APCu や WinCache は API を提供しており、PHP コードのデータをメモリキャッシュに格納できる。

オブジェクトキャッシュシステムとして最もよく使われているのは APCu と memcached だ。
APCu はオブジェクトキャッシュの選択肢として最適で、
シンプルな API を使ってデータをメモリキャッシュに格納できる。
セットアップも簡単で、すぐに使えるようになる。
APCu の制約のひとつは、インストールしたサーバーと密接に結合してしまうことだ。
一方、Memcached は個別のサービスとしてインストールするものであり、
ネットワーク越しにアクセスできる。つまり、
中央で管理している超高速なストレージにオブジェクトを格納して、
いろんなシステムからそれを取り出せるということだ。

PHPを(Fast-)CGIアプリケーションとしてウェブサーバーで実行するときには、
すべてのPHPプロセスが自身のキャッシュを持つことになる。つまり
APCuのデータもワーカープロセス間では共有されないということだ。
こんなときには、かわりにmemcachedを検討すればいい。
こっちはPHPのプロセスには結びついていない。

ネットワーク環境において、アクセス速度の面では APCu のほうが memcached より優れている。
しかし、memcached のほうが、より手軽にスケールアップできる。
複数のサーバーを使う予定がないとか memcached の追加機能が不要だという場合は、
オブジェクトキャッシュに APCu を選ぶのが最適だろう。

APCu を使うロジックの例を示す。

{% highlight php %}
<?php
// 'expensive_data' がキャッシュに保存されているかどうかを調べる
$data = apcu_fetch('expensive_data');
if ($data === false) {
    // データがキャッシュにないときは、コストのかかる操作をして取得する。
    // そして、その結果を保存してあとで使えるようにする。
    apcu_add('expensive_data', $data = get_expensive_data());
}

print_r($data);
{% endhighlight %}

PHP 5.5 より前のバージョンでは、APC がオブジェクトキャッシュとバイトコードキャッシュの両方の機能を提供していた。
新しい APCu は、APC のオブジェクトキャッシュ機能を PHP 5.5 以降で使えるようにするものだ。
というのも、PHP 5.5 以降ではバイトコードキャッシュ（OPcache）の機能が標準で組み込まれるようになったからだ。

### オブジェクトキャッシュシステムについての参考資料

* [APCu](https://github.com/krakjoe/apcu)
* [APCu Documentation](https://www.php.net/apcu)
* [Memcached](https://memcached.org/)
* [Redis](https://redis.io/)
* [WinCache 関数](https://www.php.net/ref.wincache)
