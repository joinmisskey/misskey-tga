# Misskey Test environment with GitHub Actions

### 概要
GitHub ActionsでMisskeyのテスト用環境を構築します。  

### 特徴
- push毎、定期実行(cron)、webhookなど、柔軟にトリガー可能  
- 自動インストール(インストール中のユーザー操作不要)
- 最大約6時間実行可能(デフォルトは約30分,変更可能)
- デプロイ情報をDiscord(webhook)に送信  

### 前提条件
- webhookを受信可能なurl(Discord等)
  <details><summary>送信されるデプロイ情報の例</summary><img width="409" alt="ScreenShot 2024-01-10 1 19 30" src="https://github.com/Srgr0/misskey-test-ga/assets/66754887/8022fd0b-2906-41eb-9b63-45b3aa18c932"></details>

### 使用方法
1. リポジトリのAction Secretに以下を追加  
   DISCORD_WEBHOOK_URL : デプロイ情報の送信先url(webhookを受信可能なurl, Discordを推奨)  
2. workflowをリポジトリに保存  
   [deploy-test-environment.yml](https://github.com/joinmisskey/misskey-tga/blob/main/.github/workflows/deploy-test-environment.yml)をコピーして、リポジトリの ``.github/workflows`` ディレクトリに保存してください。  
   一部のトリガーは、workflowがデフォルトブランチに保存されている場合のみ機能します。詳細については[ワークフローをトリガーするイベント - GitHub Docs](https://docs.github.com/ja/actions/using-workflows/events-that-trigger-workflows)を確認してください。  
3. workflowの編集  
   必要に応じて、トリガーや実行時間を変更してください。  
4. workflowの実行  
   設定されたトリガーに基づき、workflowが実行されます。自動でMisskeyの構築が行われ、インストールが完了するとDiscord(webhook)にデプロイ情報が送信されます。  
   既定では、workflowファイルの保存されているリポジトリ・ブランチが使用されます。workflow_dispatchトリガーでinputsとして ``repository`` と ``branch`` を指定することで、任意のリポジトリ・ブランチを使用することができます。  
   公開先のurlについては、セキュリティ上の観点からログ上ではマスクしています。インスタンスのログは公開されますのでご注意ください(公開しないよう設定も可能ですが、インスタンスに直接接続できないためデバッグが困難になります)。  
6. Misskeyのテスト  
   Discord(webhook)に送信されたurlからMisskeyにアクセスし、テストを行ってください。外部インスタンスとの連合も可能ですが、テスト環境が一時的なものであることに留意し、テスト環境間でのみ連合をすることを強く推奨します。  
7. テストの終了  
   デフォルトでは、インストール完了から1800秒(30分)で終了します。終了後は全てのデータが消去されますのでご注意ください。インスタンスのログはGitHub Actionsの設定に基づき一定期間保存されます。  
