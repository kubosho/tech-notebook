---
title: "Pull Requestの変更差分だけをCodeQLで検査する"
emoji: "🔐"
type: "tech"
topics: [codeql, github, githubactions]
published: false
---

[CodeQL](https://docs.github.com/ja/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql)使ってますか？

CodeQLをたっぷり使っているキミも、まだまだのキミも、CodeQLの高度な構成を知って素早くCodeQLによる検査を実行できるようにしましょう！

## はじめに

さっそくですが、今回設定するGitHub Actionsのworkflowファイルの全貌を見せます。

```yaml
name: "Advanced CodeQL (for Pull Request)"

on:
  pull_request:
    types: [opened, synchronize, reopened]
    branches: ["main"]

jobs:
  analyze:
    runs-on: "ubuntu-24.04"
    strategy:
      fail-fast: false
      matrix:
        include:
          - language: javascript-typescript
            build-mode: none
    name: Analyze (${{ matrix.language }})
    timeout-minutes: 10
    permissions:
      actions: read
      contents: read
      pull-requests: read
      security-events: write
    steps:
      - id: checkout
        name: Checkout repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 1
      - id: copy-changed-files
        name: Copy changed files
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # 一時ディレクトリを作成し、後続のジョブで使えるようにする
          temp_dir=$(mktemp -d)
          echo "temp_dir=${temp_dir}" >> $GITHUB_OUTPUT

          # PRで変更があったファイルの内、下記のURLに定義されている拡張子でリポジトリに必要なファイルのみを抽出する
          # https://codeql.github.com/docs/codeql-overview/supported-languages-and-frameworks/
          file_paths=$(\
            gh pr view "${{github.event.number}}" \
              --json files \
              --jq '.files | map(.path) | map(select(. | test("\\.(js|jsx|mjs|htm|html|ejs|json|yaml|yml|xml|ts|tsx|mts|cts)$")))[]'
          )

          # 解析対象のファイルが存在しない場合にでもスキャン結果をアップロード可能にするため、適当なTypeScriptのコードを追加する
          if [[ -z "$file_paths" ]]; then
            echo "Y29uc29sZS5sb2coJ0hlbGxvLCB3b3JsZCEnKTs=" | base64 -d | tee "$temp_dir/codeql_empty_result.ts"
          fi

          # ファイルパスに空白を含む場合でもコピーできるようにする
          IFS=$'\n'
          for file_path in $(echo -e "$file_paths"); do
            path="${{ github.workspace }}/$file_path"
            if [[ -e "$path" ]]; then
              cp "$path" "$temp_dir"
            else
              echo "Warning: $path does not exist. Skipping."
            fi
          done
          unset IFS
      - id: copy-temp-dir
        name: Copy temp dir to repository root directory
        # CodeQLで解析できるようにリポジトリルートにtemp_dirをコピーする
        run: |
          cp -r --parents ${{ steps.copy-changed-files.outputs.temp_dir }} "${{ github.workspace }}"
      - id: create-config
        name: Create CodeQL config file
        # temp_dirのみを解析する用のCodeQL config fileを作成する
        run: |
          echo "name: \"CodeQL config\"" > "${{ github.workspace }}/tmp_codeql_config.yaml"
          echo "paths:" >> "${{ github.workspace }}/tmp_codeql_config.yaml"
          echo "  - ${{ steps.copy-changed-files.outputs.temp_dir }}" >> "${{ github.workspace }}/tmp_codeql_config.yaml"
      - id: codeql-init
        name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          config-file: "${{ github.workspace }}/tmp_codeql_config.yaml"
          languages: ${{ matrix.language }}
          build-mode: ${{ matrix.build-mode }}
      - id: codeql-analyze
        name: Perform CodeQL Analysis
        uses: github/codeql-action/analyze@v3
        with:
          category: "/language:${{ matrix.language }}"
```

ゴリッゴリにGitHub Actionsのworkflowを書いて、シェルスクリプトも書いていますが、これからどんなことをしているかを書くので安心してください。

ただその前になぜPull Requestの変更差分だけCodeQLを実行するようにしたかったのか、その背景を書きます。

## 背景

## Pull Requestの変更差分だけをCodeQLで検査する

## まとめ

## 関連リンク

- [How to configure code security and quality scanning with CodeQL Advanced Setup](https://resources.github.com/learn/pathways/security/intermediate/codeql-advanced-setup/)
- [コード スキャンの高度なセットアップの構成](https://docs.github.com/ja/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/configuring-advanced-setup-for-code-scanning)
- [コード スキャン用の高度なセットアップのカスタマイズ](https://docs.github.com/ja/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning)
