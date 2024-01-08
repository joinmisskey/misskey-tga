# Misskey test environment with GitHub Actions

### 概要
GitHub ActionsでMisskeyのテスト用環境を構築します。  

### 特徴
- push毎、定期実行(cron)、webhookなど、柔軟にトリガー可能  
- 自動インストール(インストール中のユーザー操作不要)  
- デプロイ情報をDiscord(webhook)に送信  
- ~~idとパスワードによるアクセス制限~~ (issue https://github.com/Srgr0/misskey-test-ga/issues/1)    

### 前提条件
- ngrokアカウント
- webhookを受信可能なurl(Discord等)  

### 使用方法
1. ngrokアカウントにログイン/作成し、Authtokenを取得  
2. リポジトリのAction Secretに以下を追加  
   NGROK_AUTH_TOKEN : ngrokアカウントのAuthtoken  
   DISCORD_WEBHOOK_URL : デプロイ情報の送信先url(webhookを受信可能なurl, Discordを推奨)  
   ~~ACCESS_ID : テスト環境へのアクセスに使用するID~~ (issue https://github.com/Srgr0/misskey-test-ga/issues/1)  
   ~~ACCESS_PASS : テスト環境へのアクセスに使用するパスワード~~ (issue https://github.com/Srgr0/misskey-test-ga/issues/1)  
4. workflowファイルをリポジトリにコピーし、必要に応じて編集  
5. リポジトリのActionsタブ、Deploy test environmentからワークフローを実行  
6. デフォルトではインストール完了から600秒(10分)で終了します。この時間はworkflowファイルで変更可能です(最大で約6時間)。  
