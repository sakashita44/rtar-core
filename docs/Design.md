# データ解析テンプレート設計書

## 概要と目的

本設計書は「データ加工履歴の追跡」「バージョン管理」「環境の汚染防止」などを効率的・再現性高く行うためのテンプレートを提供する.
本テンプレートはデーカル環境でのデータ解析を想定しており, データ解析の効率化と再現性向上を目指す.

* 研究データ解析の効率化・再現性向上
* データと解析スクリプトのバージョン対応の容易化
* 階層構造をもつデータの管理
* 処理経路の可視化と意思決定プロセスの追跡

## 設計思想

階層構造データを扱うデータ解析を前提とし，以下のツール，言語，ライブラリを使用することで，データ解析の効率化と再現性向上を図る．
これらのツールを使用しないユーザは対象外 (ただし本テンプレートを参考にする等を制限するものではない)．
なお，(クリーンアーキテクチャの概念を参考に，)本テンプレートはデータ解析のフレームワークを提供するものであるため，
異なるツールやライブラリを使用する場合は，本テンプレートを参考にして適切な設計を行うことが望ましい．
また，データ解析の手法等についても本テンプレートは提供しない．

* VS Code
    * 豊富な拡張機能を活用して他ツールとの連携を図る
* Jupyter Notebook
    * 探索的な解析を行う際に意思決定プロセスを同時に記録するためのツールとして使用
* Python
    * データ処理の汎用的な言語として柔軟なデータ処理を行うために採用
* PowerShell/ShellScript
    * 自動化スクリプト作成用にネイティブな環境で使用可能な言語として採用
* Git, GitHub
    * プロジェクトのバージョン管理と共有
* DVC
    * データのバージョン管理
* Docker
    * コンテナ化による環境分離と再現性向上
* DuckDB
    * リレーショナルデータベースを用いた階層構造データの管理

解析手順として以下のステップを想定する．
このステップを踏むことで，データ解析の効率化と再現性向上を図る．
また，各ステップでのデータの管理や処理経路の可視化を行うことで，意思決定プロセスを含めてデータ解析の透明性を高める．

1. 要求データ定義
    * 最終的な解析(発表等に必要なデータ)を定義するステップ
    * 要求データから逆算して，生データの加工方法を考える
1. 加工手順の策定とスクリプト化 (preprocessing)
    * 生データから要求データを生成するための加工手順を策定するステップ
    * Jupiter Notebookで試行錯誤を行い，手順が確定したらスクリプト化，DVCでパイプラインとして一括で処理できるよう管理
        * 試行錯誤の過程も記録 (nbstripoutで不要な出力を削除)
        * パイプライン処理の過程はDAGで可視化，`PROCESS_OVERVIEW.md`に記載することで処理経路を把握
        * 現在のデータがどのバージョンのデータから生成されたかをDVCで管理，`VERSION_MAPPING.md`で記録，把握
1. 解析 (analysis)
    * 要求データを基に統計解析等を行うステップ
        * 探索的な解析に対応するため，Jupyter Notebookを用いて意思決定プロセス含めて記録 (nbstripoutで不要な出力を削除)
        * 処理手順が確立したらスクリプト化してDVCで管理
            * 加工手順策定時と同様，処理経路の把握とバージョン管理を行う
1. 出力 (report)
    * 解析結果をグラフ化などして出力するステップ
        * レポート用Notebookを用いて出力
            * 繰り返し出力することは考慮せず，資料作成毎にNotebookを作成することを想定 (nbstripoutで不要な出力を削除)

### 用語

* 生データ: データ解析の対象となる生データ
    * 機器等から取得した生データを指す
* 加工データ: 生データを加工して得られるデータ
* 解析データ: 統計解析等の結果データ
* 前処理(preprocessing): 生データを加工する処理
* 解析(analysis): 加工データを基に統計解析などを行う処理
* 出力(report): 解析結果をグラフ化などして出力する処理

## 使用ツール等

### 想定開発環境

* VS Code
    * プログラミング，デバッグ，コード管理
    * profileを使って設定を共有
    * DevContainerで環境構築
    * 拡張機能でDVCやJupyter Notebookを使用
* Jupyter Notebook
    * 初期段階の加工の試行錯誤と記録
    * nbstripoutで不要メタデータやセル出力の除去

### 想定言語

* Python
    * データ解析
* PowerShell/ShellScript
    * 自動化

### バージョン管理と環境設定

* Git/GitHub
    * プロジェクトのバージョン管理
    * プロジェクトの共有
* DVC
    * データのバージョン管理
* Docker
    * コンテナ化による環境分離

## ディレクトリ構造

以下のような階層構造で開発・管理を進めます.

```text
analysis-template/
├── data/
|   ├── dvc_repo/             # DVCリモートリポジトリ
│   ├── raw/                  # 生データ
│   ├── processed/            # 加工データ
│   └── analysis/             # 解析データ
├── params/                   # パラメータファイル
|   ├── preprocessing.yaml    # 前処理パラメータ
|   └── analysis.yaml         # 解析パラメータ
├── exploratory/              # 試行錯誤用Notebook
│   ├── preprocessing/        # データ前処理用Notebook
│   └── analysis/             # 解析(統計解析等)用Notebook
├── reports/                  # レポート用Notebookとスクリプト
│   ├── notebooks/            # レポート用Notebook
│   └── common/               # レポート用共通スクリプト
├── scripts/                  # スクリプト
│   ├── preprocessing/        # データ前処理スクリプト
│   ├── analysis/             # 解析スクリプト
│   ├── common/               # 共通処理ファイル
|   ├── interface/            # インターフェーススクリプト
|   └── misc/                 # その他のスクリプト
├── schema/                   # データの構造定義
|   ├── data_structure.yaml   # data/のデータ構造定義
|   └── entity_relation.yaml  # ER定義
├── env/                      # 仮想環境関連ファイル
│   ├── Dockerfile            # Docker設定ファイル
|   ├── requirements.txt      # Pythonパッケージ依存関係
│   └── analysis.code-profile # VS Codeの設定ファイル
├── docs/                     # ドキュメント
│   ├── Design.md             # デザインドキュメント
│   ├── Workflow.md           # ワークフロードキュメント
│   ├── Setup.md              # セットアップドキュメント
|   ├── DocumentationRules.md # ドキュメント記述ルール
|   └── LearningRoadmap.md    # 学習ロードマップ
├── info/                     # 確認の必要なファイル
|   ├── DATA_REQUIREMENTS.md  # データ要件定義文書
│   ├── PROCESS_OVERVIEW.md   # 処理経路の概要
│   ├── VERSION_MAPPING.md    # バージョン対応ドキュメント
│   └── dag_images/           # DAG画像
├── dvc.yaml                  # DVC設定ファイル
├── dvc_stages/               # DVCステージファイル
├── README.md                 # プロジェクトの概要
...                           # その他のディレクトリ
```

## ディレクトリ詳細

* `data/`
    * `dvc/` - DVCの実体を保管するディレクトリ (個人での使用を想定)
    * `raw/` - 各機器から取得した生データを保管する.
    * `processed/` - 生データに対する前処理や加工を施したデータを格納する.
    * `analysis/` - 統計解析等の結果データを格納する.
* `params/` - 前処理や解析に使用するパラメータファイルを管理するディレクトリ.
    * `preprocessing.yaml` - 前処理のパラメータを記載するファイル.
    * `analysis.yaml` - 解析のパラメータを記載するファイル.
* `exploratory/` - 試行錯誤用Notebook群を管理するディレクトリ.
    * `preprocessing/` - 前処理の試行錯誤用Notebookを配置する.
    * `analysis/` - 統計解析等を検証するNotebookを配置する.
* `reports/` - レポート作成に使用するNotebookとスクリプトを管理するディレクトリ.
    * `notebooks/` - 解析結果やデータ可視化のためのNotebookを格納する.
    * `scripts/` - Notebookと連携してレポートを自動生成するスクリプトを格納する.
* `scripts/` - 分析全体で利用する各種スクリプトを管理するディレクトリ.
    * `preprocessing/` - 生データの前処理スクリプトを配置する.
    * `analysis/` - 統計解析等の処理スクリプトを配置する.
    * `common/` - 複数の処理で再利用可能な共通関数を格納する.
    * `interface/` - データの読み込みや出力のラッパー関数を格納する.
    * `misc/` - gitやDVCの操作, セットアップスクリプトなど，その他のスクリプトを格納する.
* `schema/` - データの構造定義を管理するディレクトリ.
    * `data_structure.yaml` - `data/`のデータ構造定義を記載するファイル.
    * `entity_relation.yaml` - ERの定義を記載するファイル.
* `env/` - 仮想環境や依存ライブラリの設定ファイルを管理するディレクトリ.
    * `Dockerfile` - コンテナ環境構築用の設定ファイル.
    * `requirements.txt` - Pythonパッケージの依存関係を記載するファイル.
* `docs/` - プロジェクトに関するドキュメント類を管理するディレクトリ.
* `info/` - プロジェクトのリアルタイム情報を管理するディレクトリ.
    * `PROCESS_OVERVIEW.md` - 処理経路の概要をDAG画像とともに記載する.
    * `VERSION_MAPPING.md` - 各バージョンがどのデータやプロセスに対応しているか記録する.
    * `dag_images/` - DAG画像を保管する専用ディレクトリ.
* `dvc.yaml` - DVCの設定ファイル.
* `dvc_stages/` - DVCステージファイルを管理するディレクトリ. 実際のステージの階層構造を反映.
* `README.md` - プロジェクトの概要を記載したファイル.

## データ管理とバージョン管理

### データの加工履歴追跡

* DVCで生データ→加工データの変換を追跡
* どのスクリプトや変数を使用したか記録
* dvcで管理できない外部ツールによるデータについてもメタデータを記録してバージョン管理
* データ加工に使用された変数(しきい値など)もバージョン管理
    * DVCを使用することで特定データのみをロールバック等することが可能

### データ／変数のバージョン管理

* データと解析パラメータをセットで追跡
* メタデータはJSON形式などで管理

### DVCの自動化

* DVCステージを活用し，加工からバージョン管理までを一元化
* `dvc dag`などで処理経路を可視化

## 環境設定と管理

### ローカル環境の汚染防止

* `Docker`を用いてPythonとpipでインストールするツールの環境を分離
* ホスト環境にはDockerとGit, VS Code以外のインストールを不要にする

### 初期セットアップの簡略化

* スクリプトでディレクトリ作成やパッケージインストールを自動化

## データ解析と可視化

### 共通処理の活用

* common.pyやcommon-for-processing.pyなどの共通処理を用意し，スクリプト内でimportして再利用

### 部分的なデータ加工

* 小規模データで最初に検証 → 問題なければ全データに適用

### Notebookによる試行錯誤

* Jupyter Notebookで手順を検討し，確定したらスクリプト化
* `nbstripout`で不要な出力セルを削除
* 前提条件や使用データを記載 (DAGのどのステージに対応するか等)

### 可視化

* Matplotlib, Seaborn, Plotly等で解析結果を可視化
* 論文や発表用など用途別にNotebookを作成

## データアクセスと管理

### 階層構造データの把握

* 被験者 > 試行 > データといった構造的情報をDuckDBで管理
    * リレーショナルデータベースを用いて階層構造を管理
        * データ加工はpandasで行い，DuckDBに格納等

### 前提条件とチェック

* データパスの存在確認，パラメータのチェックを自動化
* DVCを通じてメタ情報も管理

## 処理経路と意思決定プロセス

* スクリプトの先頭にNotebookの参照リンクなどを付与し，意思決定経緯を追跡
* DVCの処理経路可視化で，加工データがどのルートで生成されたかを確認

## データ生成プロセスとデータ管理

* 最終的にほしいデータが複数存在するため，中間のデータ処理プロセスを詳細に設計する
* 各生成データとそのプロセス，使用しているデータのバージョンがひと目で分かる仕組みを整備する
    * DVCのDAG可視化や`dvc repro`で再現性を担保
    * データ生成プロセスの概要を`PROCESS_OVERVIEW.md`に記載
    * データのバージョン管理を`VERSION_MAPPING.md`で行う
        * 現在のデータがどのバージョンのデータから生成されたかはCI/CDで自動チェックして適宜`VERSION_MAPPING.md`を更新

### DAG画像の管理

* DAGで生成した画像を保管するために，専用ディレクトリ `dag_images/` を用意する
* `PROCESS_OVERVIEW.md` により，生成データ，プロセスの流れ，使用データのバージョンが参照可能になる

### バージョン対応ドキュメント

* `VERSION_MAPPING.md` を作成し，各バージョンがどのプロセスやデータに対応するか記録する
* バージョン変更に伴う差分や使用ツールのバージョンを明記し，再現性を担保する

## CI/CDの導入

* Gitフックを用いて，コミット時に自動でDVCの処理やバージョン管理を行う

## 解析ワークフロー

本ワークフローは, 以下のステップで解析を進める

1. 要求データ定義
最終的に必要なデータを整理し, 各データの役割と使用目的を明確に定義する

* ドキュメント更新: 定義内容および目的を記載する

1. 中間データ定義
最終的に必要な各データに対して必要な中間データを定義し, 再利用可能な中間データを特定する

* ドキュメント更新: 中間データの仕様および関係を明記する

1. 前処理ステップ
各中間データに対する前処理を実施し, 生データを加工する

* ドキュメント更新: 前処理の方法や使用パラメータを記録する

1. 解析ステップ
加工データを基に統計解析を実施する

* ドキュメント更新: 解析手法や結果をドキュメントに反映する

1. 出力ステップ
解析結果を元にグラフ化やその他出力を実施する

* ドキュメント更新: 出力方法と結果の概要を記録する

## 拡張可能性

* 実験の設計とデータ取得の規格化
    * データ取得の際のメタデータを記録し，データ取得のプロセスを再現可能にする

## 結論

本テンプレートは，既存の解析設計書を統合し，効率的かつ再現性が高いデータ解析フローを実現します.DVCやNotebook，Docker等を組み合わせた環境により，加工履歴や解析変数の管理，環境汚染防止の仕組みを整え，研究データ解析の再現性や信頼性を向上させます.
本テンプレートはGitHub上で公開し，他の研究者との共同作業や，新規プロジェクトへの適用を容易にします.

## アイデア

* rtar listコマンドなどでデータ構造やバージョン情報を一覧表示
    * DVCでディレクトリ単位で管理しているデータにも対応
    * DB中での階層構造も表示したい
* DATA_REQUIREMENTS.mdをUMLで記述
    * DocumentationRules.mdを更新
* データのバリデーションステップを組み込めるように
    * 削除したデータ/特殊な処理を施したデータがあとから追えるようにしたい
