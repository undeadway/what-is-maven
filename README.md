# 0. Mavenとその利点、使い方

[Maven](http://maven.apache.org/)はファイルの集まり（プロジェクト）をビルドして成果物を得るためのツールです。

ソースコードのコンパイルに加えて、コンパイルに必要なJARファイルを探したり、単体テストを実行したり、ビルドしたJARファイルを公開したりといった作業を簡単に行うことができます。
またMavenは「テストの前にコンパイル」「JARファイル作成の前にテスト」といった *作業の順番（ビルド・ライフサイクル）* や、「依存元の前に依存先をビルド」といった *依存関係* をきちんと考えてビルドしてくれます。

Mavenを使うことで、誰でも簡単にプロジェクトを同じ手順でビルドできるのです。

    ※本ドキュメントはJava開発にある程度なれた個人を想定して書かれており、
    コンパイル・PATH・CLASSPATH・JARファイル・単体テストについての説明は省略しています。
    Mavenのバージョンは`3.1.x`を想定しています。


# 1. 使ってみよう
## インストール

MavenはJDKを利用しますので、先に[JDKをインストール](http://www.oracle.com/technetwork/java/javase/downloads/)しておいてください。

次に[Maven公式サイトから圧縮ファイルをダウンロード](http://maven.apache.org/download.cgi)し、展開してください。binディレクトリに実行可能ファイルが含まれていますので、これをPATHに追加すればインストールは完了です。homebrewなどのパッケージ管理システムを使ってもインストールできます。

最後に`mvn --version`を実行して、Mavenが正しく実行されること・意図通りのJDKが認識されていることを確認してください。

## 使い方

コマンドラインから実行する場合、`pom.xml`というファイルがあるディレクトリに移動して`mvn package`コマンドを実行してください。
Mavenが依存ライブラリのダウンロード、ソースコードのコンパイル、テストの実行、JARの作成をすべて自動で行います。作成されたJARはtargetディレクトリとプライベートリポジトリに保存されています。

例えば以下のコマンドを実行すると、[findbugs plugin](https://github.com/eller86/findbugs-plugin)プロジェクトのビルドとインストールを行います。どんなプロジェクトでもインストールするために実行するコマンドは同じです。

```bash
$ git clone https://github.com/eller86/findbugs-plugin.git
$ cd findbugs-plugin
$ mvn install
```

    ※（makeやnpmに慣れている方へ）Mavenのインストールは`make install`や
    `npm install -g`とは違い、プロジェクトから実行可能ファイルを作ってPATHに追加することを
    意味しません。ライブラリ（JARなど）を作ってプライベートリポジトリに配置することです。
    プライベートリポジトリについては次章で解説します。

## pom.xmlについて

プロジェクトのトップディレクトリには通常`pom.xml`という名前の設定ファイルを配置します。このファイルにはプロジェクトの情報、依存関係、利用するプラグイン、ライセンス、SCMのURLなど様々な情報を記録できます。


# 2. リポジトリとは

JARなどの成果物やJavadocをライブラリを整理してまとめておく場所のことです。セントラルリポジトリ、ローカルリポジトリ、プライベートリポジトリの3種類があります。

## セントラルリポジトリとは

インターネットに公開されているリポジトリで、たくさんのライブラリが公開されています。

- [Maven central repository](http://search.maven.org/)

## プライベートリポジトリとは

何らかの理由でセントラルリポジトリにライブラリを公開したくない場合、自分でリポジトリを用意して利用することができます。このリポジトリのことをプライベートリポジトリと呼びます。WEBDAVが使えるサーバならなんでもプライベートリポジトリとして使えますが、Apache Archivaや[Nexus](http://www.sonatype.org/nexus/)などの管理機能を持つウェブアプリケーションを使うと便利です。

なおプライベートリポジトリ以外のリポジトリを表す用語としてリモートリポジトリ（remote repository）があります。

### プライベートリポジトリを使うには

プライベートリポジトリをセットアップしたら、[pom.xmlに使用するプライベートリポジトリのURLを明記する](http://maven.apache.org/guides/mini/guide-multiple-repositories.html)必要があります。

```xml
<repository>
  <id>my-repo1</id>
  <name>your custom repo</name>
  <url>http://jarsm2.dyndns.dk</url>
</repository>
```

## ローカルリポジトリとは

mvnコマンドを実行したマシンにあるディレクトリのことです。デフォルトでは`~/.m2/repository`が利用されます。他のリポジトリからダウンロードしたライブラリを保管したり、`install`ゴールでJARをインストールしたりするために使われます。

基本的にMavenは、ライブラリを取得ときにまずローカルリポジトリを確認し、そこになかった場合にセントラルリポジトリやプライベートリポジトリを見に行きます。
セントラルリポジトリにもプライベートリポジトリにも公開されていないライブラリを使う場合には、まず `mvn install` でそのライブラリをプライベートリポジトリにインストールしてやるときちんと使うことができます。

# 3. 依存関係とは

プロジェクトをビルドするときに、JDKだけでなくライブラリを必要とすることがあります。このことを「プロジェクトはライブラリに[依存している](http://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)」と表現します。
Mavenではプロジェクトがライブラリに依存していることを以下のように明記できます。

```xml
<dependency><!-- このプロジェクトはJUnit バージョン4.11に依存している -->
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.11</version>
  <scope>test</scope>
</dependency>
```

## ライブラリの指定方法

依存するライブラリを指定する場合、ライブラリを特定するために *groupId*, *artifactId*, *version*の3つを指定する必要があります。これらはそれぞれライブラリが所属するグループ（パッケージ名のようにドメインを逆にしたものを使っている場合が多い）、ライブラリ名、バージョンを意味します。

    ※まれに classifier を指定することもあります。

なお`1.0.0-SNAPSHOT`のように`-SNAPSHOT`で終わるバージョンは開発中を意味しています。開発中のバージョンは実装が変更される可能性があります。
対して`0.9.9`のように`-SNAPSHOT`がつかないバージョンは、実装が変更されることは基本的にありません。

バージョンの書式には`-SNAPSHOT`以外の決まりはありません。バージョンのつけ方に迷う場合は、[Semantic Versioning](http://semver.org/)を参考にすると良いでしょう。

## スコープの意味と使い分け

依存関係を明記する場合、スコープ（scope）を記述するときがあります。スコープとは「いつこのライブラリが必要になるか」を意味するものです。使える条件を明示するという意味で、Javaのスコープ（privateやpublicなど）に似ています。

スコープにはいくつか種類があります。
*compile* はプロジェクトをコンパイルするときと実行するときに必要なライブラリに使います。スコープのデフォルト値がこのcompileなので、`<scope>compile</scope>`は省略できます。
*test* はコンパイルにも実行にも必要ないがテストを実行する際に必要となるライブラリに使います。JUnitやMockito、Powermockなどに使うことが多いでしょう。
*provided* はコンパイル時には必要ないが、実行時には必要なライブラリに使います。ウェブアプリケーションを作成する際は、サーブレットコンテナが提供するライブラリのためにこのスコープを使うことになるでしょう。


# 4. ビルド・ライフサイクルとは

[ビルド・ライフサイクル](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)とは「コンパイル→テスト→JAR作成」などの作業の順番を定義したものです。標準でdefaultサイクルとcleanサイクル、siteサイクルが用意されています。

ライフサイクルに含まれている作業内容のことをフェーズ（phase）と呼びます。例えばdefaultライフサイクルにはcompile, test, verify, installといったフェーズが含まれています。
mvnコマンドを実行するときに「どのフェーズを実行するか」を指定できます。例えば今まで実行してきた`mvn install`は、defaultライフサイクルを初めからinstallフェーズまで実行するという意味になります。このとき指定したフェーズだけでなく、同じライフサイクルの指定フェーズ以前のフェーズも順次実行されることに注意してください。


# 5. プロジェクトをリポジトリに公開する

[こちらのドキュメント](./release.md)を参照してください。

# 9. この文書について

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/deed.en_US"><img alt="Creative Commons License" style="border-width:0" src="http://i.creativecommons.org/l/by/3.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/deed.en_US">Creative Commons Attribution 3.0 Unported License</a>.
