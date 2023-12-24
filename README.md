# Misskey test environment with GitHub Actions

### 概要
GitHub Actions Runner上でMisskeyインスタンスをホストし、ngrok経由で公開します。

### 特徴
- push毎に自動でテスト用環境を生成
- idとパスワードによるアクセス制限

### 前提条件
- ngrokアカウント

### 使用方法
1. ngrokアカウントにログイン/作成し、Authtokenを取得
2. リポジトリのAction Secretに以下を追加
   NGROK_AUTH_TOKEN : ngrokアカウントのAuthtoken
   ACCESS_ID : テスト環境へのアクセスに使用するID
   ACCESS_PASS : テスト環境へのアクセスに使用するパスワード
3. workflowファイルをリポジトリにコピーし、必要に応じて編集
4. 開発でテスト環境を活用
