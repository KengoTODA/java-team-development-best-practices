
# 開発プロセス
## コミットとプルリクエスト（PR）
* 小さい単位でコミットしプッシュする。
    * 「作業中」のような目的を達成しないコミットでも構わない。作業内容の透明性をチームに対して保つことは、[経験主義の三本柱](https://www.scrum.org/resources/blog/three-pillars-empiricism-scrum)を強め助け合う文化を醸成する。
* デフォルトブランチにマージする際は[Conventional Commits](https://www.conventionalcommits.org/ja/v1.0.0/)を適用する。
    * 上記の「目的を達成しないコミット」を使った場合、PRをマージする際にsquashする。
* 以下のような場合、こまめに[Draft PR](https://docs.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests#draft-pull-requests)を利用して途中経過を共有する：
    * 再現テストケースを作成した場合
    * 試している方法がうまく行かず、助言を求めたい場合
    * 言葉よりコードで説明したほうが伝わりやすい変更を提案する場合
* PR作成時にビルド（pre-merge build）を実行する。
    * ビルドに使用する手法は、開発者が手元で実行するものと同じにする。コマンドに血が通うこと、実行内容を全開発者が認識していることが重要。
        * PMDやSonarCloudなど、クリーンルーム環境でのみ実行するものはあっても良いが、実行コマンドを分けないこと。
        * `mvn verify -B`や`./gradlew build`のようなコマンドを実行し、MavenのprofileやGradleのonlyIfを使って自動的に実行するタスクを判別する。
    * [Gradle wrapper validation](https://github.com/gradle/wrapper-validation-action)や[semantic-pull-requests](https://github.com/zeke/semantic-pull-requests)のような処理を実行しても良い。

## チケット管理
* 作成時は背景や「何を持って解決とできるのか」を明示する。
    * 人数の多いプロジェクトでは[テンプレート](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests)の作成を検討する。
* ラベルの使い方について共通見解をつくり、`CONTRIBUTING.md`にまとめておく。
    * [Sane GitHub Labels](https://medium.com/@dave_lunny/sane-github-labels-c5d2e6004b63)のような外で定義・説明されているものを流用しても良い。
    * [actions/labeler](https://github.com/actions/labeler)や[github/issue-labeler](https://github.com/github/issue-labeler)を用いて自動的にラベルを決定・付与できるようにする。

## コードレビュー
* 潜在不具合やコードフォーマットの確認は自動化する。
    * 人間は設計や命名、意図が読み取れるか否かなどの生産性確保に注力する。
    * 潜在不具合の確認は[Google Errorprone](http://errorprone.info/)や[SonarCloud](https://sonarcloud.io/)で行う。
    * コードフォーマットは[google-java-format](https://github.com/google/google-java-format/)を利用すると、IntelliJ IDEAや[spotless](https://github.com/diffplug/spotless)による統合がやりやすい。

## リリース（開発）

ここで言う「リリース」は運用フェースにおける「リリース」とは異なるが、GitHub Releasesに合わせてリリースと呼称する。

* ライブラリの場合
    * [semantic-release](https://semantic-release.gitbook.io/)の利用を検討する。
        * ユーザが利用バージョンを選択可能なため、頻度高くリリースすることのデメリットが極めて小さい。
* アプリケーションの場合
    * [Release Drafter](https://github.com/release-drafter/release-drafter)あるいは[semantic-release](https://semantic-release.gitbook.io/)の利用を検討する。
    * Javaアプリケーションの場合、[Jib](https://github.com/GoogleContainerTools/jib)によるコンテナの作成を検討する。
* サポートバージョンは最小限に保つ。
    * 基本的には`main`ブランチのみを保守する。
    * ライブラリのバージョン移行期間は2つのブランチを保守しても良い。`semantic-release`は[branches](https://semantic-release.gitbook.io/semantic-release/usage/configuration#branches)設定によってリリースするブランチを複数設定できる。
## 依存管理
* GitHubの提供する機能を最大限活用する。
    * `.github/dependabot.yml`ファイルを作成し、dependabotを有効にする。
    * [bulldozer](https://github.com/palantir/bulldozer)や[probot-auto-merge](https://github.com/bobvanderlinden/probot-auto-merge)を用いて依存更新を自動化する。
        * 手動で確認・マージしていると、頻度が高く数も多いため追いつかない。
        * Branch protection機能で[Checkが通ることをマージ前に確認](https://docs.github.com/en/github/administering-a-repository/about-protected-branches#require-status-checks-before-merging)する設定をすれば、ビルドが破損した状態でのマージを防げる。

## アーキテクチャ
* `ARCHITECTURE.md`ファイルを作成し、俯瞰的な構成や基本方針を明文化する。
    * 人数の多いプロジェクトでは[ArchUnit](https://www.archunit.org/)を使って自動的に構成を検証しても良い。
* ミドルウェアをローカルで構築可能にするため、`docker-compose.yml`やKubernetesマニフェストを用意する。
    * 開発時も`docker-compose`や`minikube`などを使ってローカルで環境を構築する。
    * Kubernetesマニフェストは[別リポジトリでの管理が推奨される向きがある](https://argoproj.github.io/argo-cd/user-guide/best_practices/#separating-config-vs-source-code-repositories)。
* JavaバージョンはLTSよりも最新のものを使うことを検討する。
    * 特にJVMの実行時性能は新しいほうが高いことが多い。
    * JVM管理を第三者が行う場合にはLTSの採用を検討する。

## 自動テスト

* [Testing Pyramid](https://www.browserstack.com/guide/testing-pyramid-for-test-automation)ないし[testing honeycomb](https://engineering.atspotify.com/2018/01/11/testing-of-microservices/)を意識する。
    * コストの高い統合テストよりも単体テストで検証可能な構成を心がける。
    * 例えばDDDで開発する場合、ドメインモデルのみで機能要件をある程度検証可能であることが望ましい。


# 運用
## デプロイ
## リリース（運用）



