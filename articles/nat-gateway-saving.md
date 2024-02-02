---
title: "NATGatewayやRDSProxyを節約したい！Lambdaで定期的に変更する"
emoji: "💰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["コスト削減","AWS","Lambda","NATGateway"]
published: true
---

# はじめに

せっかくサーバレスな構成を取っているのにNAT Gatewayが高い。
1時間あたり0.045ドル
1日1.08ドル

開発環境では常にリソースが起動している必要は無いので、料金削減したいと思いました。

# 問題

LambdaでSDK使って定期的にNAT Gatewayを削除、構築をしたいが関連するリソースが多く大変。

# 解決方法

NAT Gatewayが存在するCloudFormationテンプレートと、存在しないテンプレートを用意し、
定期的にLambdaを実行することで解決しました。

ついでに、RDS Proxyもそこそこ料金がかかっていたので、対応しています。
(こっちの方が大変だった)

```python
import boto3
import json
import time

# CloudFormationとRDSクライアントの作成
cf_client = boto3.client("cloudformation")
rds_client = boto3.client("rds")

templates = [
    # VPCやNAT Gatewayを定義したCloudFormation
    {
        "stack_name": "sample-vpc-dev",
        "saving": "sample-vpc.template-節約版.yml",
        "full": "sample-vpc.template.yml",
    },
    # RDSやProxyを定義したCloudFormation
    {
        "stack_name": "sample-rds-dev",
        "saving": "sample-rds.template-節約版.yml",
        "full": "sample-rds.template.yml",
        "db_cluster_identifier": "sample-dev-db-cluster",
    },
]


def start_db_cluster_if_stopped(db_cluster_identifier):
    # RDSクラスターの状態を確認
    response = rds_client.describe_db_clusters(
        DBClusterIdentifier=db_cluster_identifier
    )
    status = response["DBClusters"][0]["Status"]

    # クラスターが停止している場合、再開する
    if status == "stopped":
        print(f"Starting RDS Cluster: {db_cluster_identifier}")
        rds_client.start_db_cluster(DBClusterIdentifier=db_cluster_identifier)
        # 再開が完了するのを待つ（必要に応じて）
        waiter = rds_client.get_waiter("db_cluster_available")
        waiter.wait(DBClusterIdentifier=db_cluster_identifier)
        print(f"RDS Cluster {db_cluster_identifier} has been started.")
    else:
        print(f"RDS Cluster {db_cluster_identifier} is not in a stopped state.")
    print("RDS Cluster is running.")


def stop_db_cluster(db_cluster_identifier):
    print(f"Stopping RDS Cluster: {db_cluster_identifier}")
    rds_client.stop_db_cluster(DBClusterIdentifier=db_cluster_identifier)


def wait_for_stack_update(stack_name):
    while True:
        response = cf_client.describe_stacks(StackName=stack_name)
        stack_status = response["Stacks"][0]["StackStatus"]
        if stack_status in [
            "UPDATE_COMPLETE",
            "UPDATE_ROLLBACK_COMPLETE",
            "UPDATE_ROLLBACK_FAILED",
        ]:
            print(f"Stack update completed with status: {stack_status}")
            break
        elif stack_status in [
            "UPDATE_IN_PROGRESS",
            "UPDATE_COMPLETE_CLEANUP_IN_PROGRESS",
        ]:
            print(
                f"Stack update in progress, current status: {stack_status}. Waiting..."
            )
            time.sleep(10)  # 10秒ごとにステータスをチェック
        else:
            raise Exception(f"Unexpected stack status: {stack_status}")


def lambda_handler(event, context):
    mode = event["mode"]  # saving or full

    for template in templates:
        # RDSのテンプレートの場合、クラスターを再開する
        if "db_cluster_identifier" in template:
            start_db_cluster_if_stopped(template["db_cluster_identifier"])

        # テンプレートファイルの読み込み
        with open(template[mode], "r") as file:
            template_body = file.read()

        # スタックの更新パラメーター
        stack_name = template["stack_name"]

        try:
            # スタックの更新
            response = cf_client.update_stack(
                StackName=stack_name,
                TemplateBody=template_body,
                Capabilities=[
                    "CAPABILITY_IAM",
                    "CAPABILITY_NAMED_IAM",
                    "CAPABILITY_AUTO_EXPAND",
                ],
                # パラメーターやその他のオプションを必要に応じて追加
            )
            print(response)

            # スタックの更新後、特定の条件に基づいてRDSクラスタを停止
            if "db_cluster_identifier" in template:
                # スタックの更新が完了するまで待機
                wait_for_stack_update(stack_name)

                stop_db_cluster(template["db_cluster_identifier"])

        except cf_client.exceptions.ClientError as e:
            # スタックが更新不要な場合、エラーをキャッチ
            if "No updates are to be performed." in str(e):
                print(f"No updates are needed for {stack_name}.")
            else:
                raise

    return {
        "statusCode": 200,
        "body": json.dumps("CloudFormation stack update initiated."),
    }


if __name__ == "__main__":
    lambda_handler({"mode": "full"}, None)
```

# 解説

中身はシンプルでeventによって節約版とフル版を切り替えています。
ポイントとして、RDS Proxyの設定を変更する場合、
作りにもよりますが元のRDSインスタンスが起動していないとCloudFormationが実行できません。

なので、 `start_db_cluster_if_stopped`　を使って、事前に起動します。
また、起動しっぱなしになってはコストがかかるので、
RDSの場合は `wait_for_stack_update` でCloudFormation実行完了まで待って
`stop_db_cluster` でインスタンスを停止する指示を行っています。

このLambda自体はEventBridgeなどで起動してください。
その時のeventはmodeというキーでfullかsavingというバリューを渡してあげてください。

# おわりに

勤務時間に合わせて起動、停止してあげることで費用が50%以上削減できました。
(24時間起動していたのが10時間程度になったため。)

Lambdaの実行時間制限(15分)に引っかからずに終わりそうになかったら、
StepFunctionsやECSなど検討が必要になりそうです。

# 参考

<https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/rds/waiter/DBClusterAvailable.html>
