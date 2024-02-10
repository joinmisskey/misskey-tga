# Misskey Test environment with GitHub Actions (β)

### 概要
GitHub ActionsでMisskeyのテスト用環境を構築します。  

### 特徴
- push毎、定期実行(cron)、webhookなど、柔軟にトリガー可能  
- 自動インストール(インストール中のユーザー操作不要)
- 連合やファイル添付などにも対応(基本的に全機能が動作)
- 最大約6時間実行可能(デフォルトは約30分,変更可能)
- デプロイ情報をDiscord(webhook)に送信  

### 前提条件
- webhookを受信可能なurl(Discord等)
  <details><summary>送信されるデプロイ情報の例</summary><img width="409" alt="ScreenShot 2024-01-10 1 19 30" src="https://github.com/Srgr0/misskey-test-ga/assets/66754887/8022fd0b-2906-41eb-9b63-45b3aa18c932"></details>

### 使用方法
1. リポジトリのAction Secretに以下を追加  
   DISCORD_WEBHOOK_URL : デプロイ情報の送信先url(webhookを受信可能なurl, Discordを推奨)  
2. リポジトリで以下のようなワークフローを構成
```
name: deploy-test-environment

on:
  issue_comment:
    types: [created]
  workflow_dispatch:
    inputs:
      repository:
        description: 'Repository to deploy (optional, use the repository where this workflow is stored by default)'
        required: false
        default: ''
      branch_or_hash:
        description: 'Branch or Commit hash to deploy (optional, use the branch where this workflow is stored by default)'
        required: false
        default: ''
      wait_time:
        description: 'Time to wait in seconds (optional, 1800 seconds by default)'
        required: false
        default: ''

jobs:
  get-pr-ref:
    runs-on: ubuntu-latest
    if: github.event_name == 'issue_comment' && github.event.issue.pull_request && startsWith(github.event.comment.body, '/preview')
    outputs:
      pr-ref: ${{ steps.get-ref.outputs.pr-ref }}
      wait_time: ${{ steps.get-wait-time.outputs.wait_time }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Get PR ref
        id: get-ref
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          PR_NUMBER=$(jq --raw-output .issue.number $GITHUB_EVENT_PATH)
          PR_REF=$(gh pr view $PR_NUMBER --json headRefName -q '.headRefName')
          echo "pr-ref=$PR_REF" > $GITHUB_OUTPUT

      - name: Extract wait time
        id: get-wait-time
        run: |
          COMMENT_BODY="${{ github.event.comment.body }}"
          WAIT_TIME=$(echo "$COMMENT_BODY" | grep -oP '(?<=/preview\s)\d+' || echo "1800")
          echo "wait_time=$WAIT_TIME" > $GITHUB_OUTPUT

  deploy-test-environment-pr-comment:
    needs: get-pr-ref
    uses: joinmisskey/misskey-tga/.github/workflows/deploy-test-environment.yml@main
    with:
      repository: ${{ github.repository }}
      branch_or_hash: ${{ needs.get-pr-ref.outputs.pr-ref }}
      wait_time: ${{ needs.get-pr-ref.outputs.wait_time }}
    secrets:
      DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}

  deploy-test-environment-wd:
    if: github.event_name == 'workflow_dispatch'
    uses: joinmisskey/misskey-tga/.github/workflows/deploy-test-environment.yml@main
    with:
      repository: ${{ inputs.repository || github.repository }}
      branch_or_hash: ${{ inputs.branch_or_hash || github.ref }}
      wait_time: ${{ inputs.wait_time || '1800' }}
    secrets:
      DISCORD_WEBHOOK_URL: ${{ secrets.DISCORD_WEBHOOK_URL }}
```
> [!TIP]
> workflowファイルが保存されているリポジトリ・ブランチが、既定のMisskeyリポジトリ・ブランチとして使用されます。つまり、workflowファイルはMisskeyリポジトリ・ブランチ(複数のブランチがある場合には各ブランチ)に保存する必要があります。  
> Pull Request用のブランチなどにworkflowファイルを保存したくない場合、workflowファイルをデフォルトブランチに保存した上で、workflow_dispatchトリガーでinputsとして ``repository`` と ``branch_or_hash`` を指定することで、任意のリポジトリ・ブランチを使用することができます。  
> workflow_dispatchを含む一部のトリガーは、workflowがデフォルトブランチに保存されている場合のみ機能します。詳細については[ワークフローをトリガーするイベント - GitHub Docs](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows)を確認してください。  
4. workflowの編集  
   必要に応じて、トリガーや実行時間を変更してください。  
5. workflowの実行  
   設定されたトリガーに基づき、workflowが実行されます。自動でMisskeyの構築が行われ、インストールが完了するとDiscord(webhook)にデプロイ情報が送信されます。  
   公開先のurlについては、セキュリティ上の観点からログ上ではマスクされます。インスタンスのログは公開されますのでご注意ください(公開しないよう設定も可能ですが、インスタンスに直接接続できないためデバッグが困難になります)。  
6. Misskeyのテスト  
   Discord(webhook)に送信されたurlからMisskeyにアクセスし、テストを行ってください。外部インスタンスとの連合も可能ですが、テスト環境が一時的なものであることに留意し、テスト環境間でのみ連合をすることを強く推奨します。  
7. テストの終了  
   デフォルトでは、インストール完了から1800秒(30分)で終了します。終了後は全てのデータが消去されますのでご注意ください。インスタンスのログはGitHub Actionsの設定に基づき一定期間保存されます。  

### GitHub Actionsの規約への適合について
[joinmisskey/misskey-tga]()の[0e502cf](https://github.com/joinmisskey/misskey-tga/tree/0e502cf396ac85f6a4a6fe7a7111956a236b4579)時点でのワークフロー構成について、GitHub Supportに確認した際の記録を公開しております。リンクは[こちら](https://gist.github.com/Srgr0/53720be675c0fe902dc112497426ce7d)です。
記録内で確認している通り、この記録の[misskey-tgaのREADME](https://github.com/joinmisskey/misskey-tga/blob/main/README.md)での参照目的での使用(共有)については、サポート担当者より許可を得ております。  
記録内にある通り、GitHub Supportの見解は問い合わせ時点でのワークフロー構成([here](https://github.com/joinmisskey/misskey-tga/blob/0e502cf396ac85f6a4a6fe7a7111956a236b4579/.github/workflows/deploy-test-environment.yml))に対するものであり、この構成が変更された場合には見解が当てはまらない可能性があることに留意してください。ここでの変更には、[joinmisskey/misskey-tga](https://github.com/joinmisskey/misskey-tga)において行われた変更も含まれます。  
懸念点がある場合には、GitHub Supportに問い合わせることも可能です(有料アカウントが必要です)。  
