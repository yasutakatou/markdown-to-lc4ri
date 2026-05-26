---
name: markdown-to-lc4ri
description: Markdown形式の手順書を、LC4RI (Literate Computing for Reproducible Infrastructure) 形式へ変換します。`.md`ファイルの中にコードブロック (```bash 等) で記述された手順を、LC4RI実行エンジンが解釈・実行できる形へ書き換える際に使用してください。ユーザーが「LC4RI形式に変換」「LC4RIに直して」「手順をLC4RI化」「Literate Computingへ移行」と言及した場合では必ずこのスキルをトリガーしてください。
---

# Markdown → LC4RI 変換スキル

このスキルは、通常のMarkdownで書かれた手順書を、**LC4RI形式** (Literate Computing for Reproducible Infrastructure) のMarkdownへ機械的に変換するためのものです。LC4RIはNotebookセル相当の実行可能ブロックを `- ` プレフィクスで表現する記法で、手順書がそのまま自動実行・再現可能な記録となるのが特徴です。

## このスキルが目指すこと

通常のMarkdownでは、手順書の見出し・説明・コマンドが入り混じり、コマンドはコードブロック内に閉じ込められているため、人間が手で写経しなければ実行できません。LC4RIではコマンドを「文書のセル」として地の文に書き出すことで、文書を読みながらそのまま実行・記録できるようになります。

変換の本質は3つです。

第一に、**説明用の箇条書きと実行コマンドを記号レベルで分離する**こと。LC4RIでは行頭の `- ` や `1.` が「実行ブロック」の合図になるため、説明用に使われていた箇条書きはすべて中黒 `・` に置き換えます。

第二に、**コマンドをコードブロックから外に出し、`- ` を付けて1行ずつ記述する**こと。これによって各コマンドが独立した「セル」となり、選択して実行・結果記録ができるようになります。

第三に、**ファイル書き出しと環境変数管理をLC4RIのディレクティブに置き換える**こと。`cat <<EOF` のようなヒアドキュメントや、その場の値を埋め込んだコマンドは再現性が低いため、`- write:` ディレクティブと `export` による変数管理に書き換えます。

## 変換ルール一覧

以下のルールを順に適用してください。順序が重要です (特に、コマンド抽出を行う前にコードブロック内かどうかの判定が必要)。

### ルール1: 説明用の箇条書きは中黒 `・` に置き換える

LC4RI形式では、行頭の `- ` や `1.` (数字+ドット) が **実行ブロックの開始記号** として解釈されます。したがって、Markdownで単なる説明用リストとして書かれていた箇条書きは、すべて中黒 `・` に置換しなければなりません。

対象となるのは次のパターンです。

最も典型的なのは、ハイフン箇条書きです。

**変換前 (Markdown):**

```markdown
- 所要時間の目安: 約 90〜120 分
- 対象読者: Kubernetes の基本を一度は触ったことがある人
```

**変換後 (LC4RI):**

```markdown
・所要時間の目安: 約 90〜120 分
・対象読者: Kubernetes の基本を一度は触ったことがある人
```

数字付きの順序付きリスト (`1.`, `2.`, `3.` …) も同じく実行扱いされるため、説明文の場合は `・` に置換します。

**変換前 (Markdown):**

```markdown
1. [この作業で学ぶこと](#1-この作業で学ぶこと)
2. [前提条件](#2-前提条件)
3. [Part 0: 基礎知識](#3-part-0-基礎知識)
```

**変換後 (LC4RI):**

```markdown
・ [この作業で学ぶこと](#1-この作業で学ぶこと)
・ [前提条件](#2-前提条件)
・ [Part 0: 基礎知識](#3-part-0-基礎知識)
```

太字や強調が混じった項目でも同様です。Reconciliation Loop の説明など、本文内の手順説明を箇条書きで書いている部分にも適用します。

**変換前:**

```markdown
1. 自分が担当する CR の状態を読む（**現在の状態**）
2. CR の `spec` を見て、本来あるべき状態を計算する（**望ましい状態**）
3. 差分を埋めるための操作を Kubernetes API に対して実行する
```

**変換後:**

```markdown
・ 自分が担当する CR の状態を読む（**現在の状態**）
・ CR の `spec` を見て、本来あるべき状態を計算する（**望ましい状態**）
・ 差分を埋めるための操作を Kubernetes API に対して実行する
```

なお、**コードブロック内 (```` ```go ````、```` ```yaml ```` 等) の `-` はそのまま残します**。YAMLの配列記法やGoのコメントは構文上の意味があるため、置換してはいけません。引用ブロック (`> ` で始まる行) の中で `> - ` のように使われている場合も、文書として保持してかまいません。

### ルール2: 実行コマンドはコードブロックから外に出し、先頭に `- ` を付ける

通常のMarkdownでは、実行コマンドは ```` ```bash ```` や ```` ```sh ```` のコードブロックに閉じ込められています。LC4RIではこれを **コードブロックの外** に出し、各行の先頭に `- ` を付けます。これによって各コマンドが独立した実行セルになります。

**変換前 (Markdown):**

````markdown
```bash
kubectl create namespace crd-handson
kubectl config set-context --current --namespace=crd-handson
```
````

**変換後 (LC4RI):**

```markdown
- kubectl create namespace crd-handson
- kubectl config set-context --current --namespace=crd-handson
```

シェル風のコメント (`# ...`) で、コマンドではなく説明だったものは、コードブロックの外に出すときに **コードブロック外の地の文** にするか削除します。例えば `# PROJECT 内の項目に projectName: memo-operator が入っていればOK` のようなコメントは、`#` を外して通常のMarkdown行として地の文に置きます。

実行コマンドの直後には、結果を取り込むための **空の出力ブロック** (言語タグなしの ```` ``` ````) を置き、その後に `---` の区切り線を入れるのがLC4RIの慣例です (実行エンジンが実行結果をこのブロック内に追記する)。変換時はまず空ブロックを用意しておきます。

```markdown
- kubectl create namespace crd-handson
- kubectl config set-context --current --namespace=crd-handson

```

```
---
```

### ルール3: バックスラッシュで継続される複数行コマンドはそのまま保持する

`\` でラインを継続する複数行コマンドや、`cat <<EOF ... EOF` のようなヒアドキュメントは、**そのままLC4RI実行エンジンが解釈できる** ため、各行に `- ` を付けてはいけません。先頭行にだけ `- ` を付け、継続行はインデントを保ったまま `- ` なしで書きます。

**変換前 (Markdown):**

````markdown
```bash
gcloud container clusters get-credentials <CLUSTER_NAME> \
  --region <REGION> \
  --project <PROJECT_ID>
```
````

**変換後 (LC4RI):**

```markdown
- gcloud container clusters get-credentials ${CLUSTER_NAME} \
  --region ${REGION} \
  --project ${PROJECT_ID}
```

注意: 先頭行の `- ` は、これがLC4RIの「コマンドセルの開始」を示すために必要です。継続行に `- ` を付けると、それぞれが別セルになってしまい、シェルが `--region` だけを単独で実行しようとして壊れます。

### ルール4: 環境変数は `export` で定義し、以降は変数として参照する

ユーザーごとに異なる値 (プロジェクトID、クラスタ名など) をその場でコマンドに埋め込んでいます。LC4RIではこれらを **最初に `export` でまとめて環境変数化** し、後続のコマンドではすべて `${VAR}` の形で参照します。これによって、値が変わっても先頭の `export` 行を直すだけで全体に伝播します。

**変換前 (Markdown):**

````markdown
```bash
gcloud container clusters get-credentials my-cluster \
  --region asia-northeast1 \
  --project welcome-study-495602
```
````

**変換後 (LC4RI):**

```markdown
- export CLUSTER_NAME=my-cluster
- export REGION=asia-northeast1
- export PROJECT_ID=welcome-study-495602
- gcloud container clusters get-credentials ${CLUSTER_NAME} \
  --region ${REGION} \
  --project ${PROJECT_ID}
```

加えて、コマンドの出力結果を変数として捕捉したい場合は、LC4RIの **キャプチャ記法** (`→ {VAR_NAME}`) を使います。

```markdown
- find $HOME/.local/share/mise/installs -type f | grep kubebuilder$ → {kubebuilder}
- export KUBEBUILDER={kubebuilder}
- $KUBEBUILDER version
```

`{kubebuilder}` のように波括弧で囲んだ部分が、直前のコマンド出力で置換されるプレースホルダになります。これは「動的な値をユーザに手で書かせない」ためのパターンです。

### ルール5: ファイル書き出しは `- write:` ディレクティブに置き換える

通常のMarkdownでは、設定ファイルの内容をコードブロックで示し、別途 `kubectl apply -f` や `cat > file.yaml` で書き出させていました。LC4RIでは専用の **`- write: <ファイル名>`** ディレクティブがあり、これを使うと「指定したファイルへ、続くコードブロックの内容をそのまま書き出す」という意味になります。

**変換前 (Markdown):**

````markdown
`memo-crd.yaml`:

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: memos.handson.example.com
spec:
  group: handson.example.com
```
````

**変換後 (LC4RI):**

````markdown
- write: memo-crd.yaml
```
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: memos.handson.example.com
spec:
  group: handson.example.com
```
- ls -la memo-crd.yaml
````

重要なポイントが3つあります。

第一に、`- write:` の直後のコードブロックは **言語タグを付けない** (` ``` ` のみ)。LC4RI実行エンジンはこの記法を見て「ファイル内容ブロック」と解釈するため、`yaml` 等のフェンス言語タグは外します。

第二に、`- write:` の直後にファイル内容のコードブロックを置き、**書き出し後の検証コマンド** (`- ls -la <ファイル名>` など) を続けて入れておくと、実行ログ上で書き出しが成功したかが一目で分かります。これはLC4RIの常套手段です。

第三に、ヒアドキュメント (`cat <<EOF | kubectl apply -f -`) で実体を直接APIに渡している箇所も、**いったんファイルに書き出してから `kubectl apply -f` する形** に書き換えることを推奨します。再現性とログの追跡性が大きく向上します。

**変換前:**

````markdown
```bash
cat <<EOF | kubectl apply -f -
apiVersion: handson.example.com/v1
kind: Memo
metadata:
  name: my-first-memo
spec:
  title: "手順メモ"
EOF
```
````

**変換後:**

````markdown
- write: my-first-memo.yaml
```
apiVersion: handson.example.com/v1
kind: Memo
metadata:
  name: my-first-memo
spec:
  title: "手順メモ"
```
- kubectl apply -f my-first-memo.yaml
````

### ルール6: 対話操作・長時間実行コマンドは `- ! ` で別ターミナルへ

`make run` や `kubebuilder create api` のように、**対話入力を伴う** あるいは **プロセスがフォアグラウンドで長時間動き続ける** コマンドは、Notebookセル内でブロッキング実行すると後続が回らなくなります。そのため、行頭を `- ! ` (ハイフン・スペース・ビックリマーク・スペース) にして「別ターミナルへ送る」指示にします。

**変換前:**

````markdown
```bash
make run
```
````

**変換後:**

```markdown
- ! make run
```

`kubebuilder create api` のように `y/n` プロンプトが出るコマンドや、Operatorをフォアグラウンドで走らせる `make run`、そして `kubectl apply` 後に長時間ログを流すワークロード起動コマンドなどがこれに該当します。

### ルール7: 編集対象として「読ませる」コードブロックは保持する

Go言語の構造体定義、編集前後のYAMLスニペット、Markdownや設定ファイルのサンプル断片など、**ユーザがエディタで参照・編集するための内容** は、もとのコードブロックを (言語タグも含めて) そのまま残してかまいません。LC4RIで「実行」しなくてよい資料的コードは、Markdownのままが読みやすいからです。

たとえば次のようなGo構造体は、変換後も `go` タグ付きのコードブロックのまま保持します。

````markdown
```go
type MemoSpec struct {
    Title   string `json:"title"`
    Content string `json:"content"`
}
```
````

判定基準は「**この内容を直接シェルで実行するか？**」です。Yesなら `- ` 付きでブロック外へ。Noならコードブロックのまま保持してください。

### ルール8: 出力ログを記録する空ブロックと区切り線を入れる

LC4RIでは、各セクション (関連する一連のコマンド) の最後に **空の言語タグなしコードブロック** を1つ置き、そのあとに `---` の区切り線を入れるのが慣例です。実行エンジンはこの空ブロックに実行結果を追記し、`---` の手前までを「1つの作業単位」として扱います。

```markdown
- kubectl apply -f memo-crd.yaml
- kubectl get crd memos.handson.example.com
- kubectl api-resources | grep memo

```

```
---
```

変換時は、すでに記録されたログがあれば空ブロックの内側に保持し、なければ空のままにしておきます。

## 変換手順 (推奨ワークフロー)

実際にMarkdownファイルをLC4RIに変換するときは、以下の順序で進めると差分が安定します。

最初に、**入力ファイル全体をスキャンし、コードブロックとその外側を分離してマッピング** します。コードブロックは言語タグ (` ```bash `、` ```yaml `、` ```go ` 等) ごとに役割が違うので、種類別に処理を変えるためです。

次に、**コードブロック外** にあるすべての `- ` および `1.` `2.` `3.` 形式の箇条書きを `・` に置換します (ルール1)。コードブロック内には触りません。

続いて、**` ```bash ` / ` ```sh ` / ` ```shell ` のコードブロック** を1つずつ取り出し、内容を以下のように処理します。

第一に、ヒアドキュメント (`cat <<EOF ... EOF`) があれば、`- write: <ファイル名>` ディレクティブと言語タグなしのコードブロックに分離します (ルール5)。

第二に、各コマンド行を取り出してコードブロックの外に並べ、先頭に `- ` を付けます (ルール2)。バックスラッシュ継続行は1セルにまとめます (ルール3)。

第三に、`make run`、`kubebuilder create api`、`kubectl logs -f` のようなブロッキング実行コマンドには `- ! ` を付けます (ルール6)。

第四に、コマンド内のハードコード値があれば、セクション冒頭にまとめて `export VAR=value` を追加し、コマンド側を `${VAR}` 参照に書き換えます (ルール4)。

最後に、**各セクションの末尾に空の出力ブロックと `---` を挿入** します (ルール8)。

Goコード、編集対象のYAMLスキーマ、表形式 (Markdownテーブル)、引用ブロック (`> ...`) はそのまま保持します (ルール7)。

## 変換の総合例

簡単な before/after を1つ通して示します。

**変換前 (Markdown):**

````markdown
### 4-1. 専用の名前空間を作る

```bash
kubectl create namespace crd-handson
kubectl config set-context --current --namespace=crd-handson
```

ポイント:

- `kubectl config set-context` で以降の対象ネームスペースが固定される
- 別ターミナルでも有効になる
````

**変換後 (LC4RI):**

````markdown
### 4-1. 専用の名前空間を作る

- kubectl create namespace crd-handson
- kubectl config set-context --current --namespace=crd-handson

```

```
---

ポイント:

・ `kubectl config set-context` で以降の対象ネームスペースが固定される
・ 別ターミナルでも有効になる
````

## よくある落とし穴

第一に、**コードブロック内の `-` まで置換してしまうミス**。YAMLの配列要素 (`  - mm` など) やGoのコメント行 (`// - foo`) は構文の一部なので、必ず「コードブロックの外側でのみ」`・` への置換を行ってください。

第二に、**バックスラッシュ継続行に `- ` を付けてしまうミス**。継続行に `- ` を入れると別セルになり、`--region asia-northeast1` のような単独実行不可能なコマンドが生まれて壊れます。必ず先頭行にだけ付けます。

第三に、**`- write:` ディレクティブの後のコードブロックに言語タグを残してしまうミス**。LC4RI実行エンジンは ` ``` ` (言語タグなし) を「ファイル内容ブロック」と解釈するため、` ```yaml ` のままだと書き出しが行われない処理系があります。

第四に、**対話コマンドへの `- ! ` の付け忘れ**。`make run` や `kubebuilder create api` をそのまま `- ` で実行すると、Notebookがハングします。「実行が即時に終わるか？」を基準に判定してください。

これらに気を付ければ、LC4RIで読み・実行・記録が一気通貫できる手順書に整います。
