# Ansible

## AWS

### Unreal Engine ( EC2 )
AWS の Windows マシーンの設定を行います。
Windows で Ansible を使用するために事前準備が必要です。

1. RDPなどで接続し、 PowerShell で以下を実行する。
    ```
    $url = "https://raw.githubusercontent.com/ansible/ansible-documentation/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
    $file = "$env:temp\ConfigureRemotingForAnsible.ps1"
    (New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
    powershell.exe -ExecutionPolicy ByPass -File $file
    ```
2. Bastion VM で WinRM トンネルを貼る
    `ssh -L 5986:{Ansible実行対象のプライベートIP}:5986 ubuntu@{Bastion EC2のグルーバルIP} -i ~/.ssh/aries-aws-{環境名}.pem -N`

    ```
    [dev]

    $ ssh -L 5986:10.30.4.122:5986 ubuntu@13.231.67.139 -i ~/.ssh/aries-aws-dev.pem -N
    ```

実行時には以下のコマンドで行います。
※ パスワードはEC2マネジメントコンソール -> 接続 -> RDPライアント -> パスワードを取得 で取得できます。

```
ansible-playbook -i aws-development.yaml unreal_engine.yaml --extra-vars "aries_password=XXXX"
```

### Coturn
AWS 上の Linux VM に対して設定を行います。
Bastion VM を経由して接続するので事前準備が必要です。

1. ローカルに aries-aws-xxx.pem を置く
    1. 値は AWS Secrets Manager(aries-xxx-aries-pem-value)にあります
    2. 配置場所は、 ~/.ssh/aries-aws-xxx.pem
    3. xxx は環境名
2. Bastion VM に ssh トンネルを貼る
    `ssh -L 10022:{Ansible実行対象のプライベートIP}:22 ubuntu@{Bastion VMのグルーバルIP} -i ~/.ssh/aries-aws-{環境名}.pem -N`

    ```
    [dev]

    $ ssh -L 10022:10.2.4.5:22 ubuntu@4.216.27.220 -i ~/.ssh/aries-aws-dev.pem -N
    ```

実行は以下のコマンドで行います。
※ turn_credentialはAWS Secrets Manager(aries-dev-iceserver-credential)に格納されています。

```
ansible-playbook -i aws-development.yaml coturn.yaml --private-key ~/.ssh/aries-aws-xxx.pem --extra-vars "turn_credential=XXXX"
```

### GitHub Self Hosted Runner
AWS 上の Linux VM に対して設定を行います。
Bastion VM を経由して接続するので事前準備が必要です。

1. ローカルに aries-aws-xxx.pem を置く
    1. 値は AWS Secrets Manager(aries-xxx-aries-pem-value)にあります
    2. 配置場所は、 ~/.ssh/aries-aws-xxx.pem
    3. xxx は環境名
2. Bastion VM に ssh トンネルを貼る
    `ssh -L 10022:{Ansible実行対象のプライベートIP}:22 ubuntu@{Bastion VMのグルーバルIP} -i ~/.ssh/aries-aws-{環境名}.pem -N`

    ```
    [dev]

    $ ssh -L 10022:10.30.4.166:22 ubuntu@4.216.27.220 -i ~/.ssh/aries-aws-dev.pem -N
    ```

実行は以下のコマンドで行います。
※ ipsec_psk, ipsec_passwordはJFDE社提供のPDFに記載されています。
※ runner_tokenはGitHub SettingsのUIから取得してください。https://docs.github.com/ja/actions/how-tos/manage-runners/self-hosted-runners/add-runners

```
ansible-playbook -i aws-development.yaml github_runner.yaml --private-key ~/.ssh/aries-aws-dev.pem --extra-vars "ipsec_psk=XXXX ipsec_password=XXXX runner_token=XXXX"
```

## Co Location

### Unreal Engine ( 北浜コロケーション )
オンプレミスの Windows マシーンの設定を行います。
Windows で Ansible を使用するために事前準備が必要です。

1. aries ユーザーをローカルアカウント(管理者権限)で作成しておく
2. PowerShell で以下を実行する。

```
$url = "https://raw.githubusercontent.com/ansible/ansible-documentation/devel/examples/scripts/ConfigureRemotingForAnsible.ps1"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)
powershell.exe -ExecutionPolicy ByPass -File $file
```

実行は以下のコマンドで行います。
Windows ログインパスワードは実行時のコマンド引数　(--extra-vars)　で渡します。
値は AWS Secrets Manager aries-{env}-co-location-credential に格納されています。

実行時には コロケーション環境へ VPN で接続してください。

```
AWS_PROFILE={ your_credential_name } AWS_REGION=ap-northeast-1 ansible-playbook -i co-location-development.yaml unreal_engine.yaml --extra-vars "aries_password=XXXX"
```
