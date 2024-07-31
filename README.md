# Covid19

##全体の工程  
###1. AWS EC2インスタンスのセットアップ  
EC2インスタンスの作成:  

AWS Management Consoleにログインし、EC2インスタンスを作成します。  
インスタンスの種類は、t2.microなどの無料利用枠内のものを選択します。  
セキュリティグループを設定し、SSH接続を許可します。  
SSH接続:  

EC2インスタンスにSSHで接続します。
bash
コードをコピーする
ssh -i "your-key.pem" ec2-user@<your-ec2-ip>
###2. 必要なソフトウェアのインストール
Javaのインストール:

Embulkの依存関係としてJavaをインストールします。
bash
コードをコピーする
sudo yum update -y
sudo yum install -y java-1.8.0-openjdk
Embulkのインストール:

bash
コードをコピーする
curl --create-dirs -o ~/.embulk/bin/embulk -L "https://dl.embulk.org/embulk-latest.jar"
chmod +x ~/.embulk/bin/embulk
echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.bash_profile
source ~/.bash_profile
Embulkプラグインのインストール:

bash
コードをコピーする
embulk gem install embulk-output-postgresql
embulk gem install embulk-input-http
3. データベースのセットアップ
PostgreSQLのインストール:

Dockerを使用してPostgreSQLをセットアップします。
bash
コードをコピーする
sudo yum install -y docker
sudo service docker start
sudo usermod -a -G docker ec2-user
docker run --name postgres -e POSTGRES_PASSWORD=your_password -d -p 5432:5432 postgres
4. Airflowのセットアップ
Airflowのインストール:

AirflowをEC2インスタンスにインストールします。
bash
コードをコピーする
pip install apache-airflow
airflow db init
Airflowの設定:

airflow.cfgファイルを編集し、必要な設定を行います。
AirflowのWebサーバー起動:

bash
コードをコピーする
airflow webserver -p 8080
Airflowのスケジューラ起動:

bash
コードをコピーする
airflow scheduler
5. データパイプラインの構築
Embulk設定ファイルの作成:
データ取得とロードの設定ファイルを作成します。
config.yml

yaml
コードをコピーする
in:
  type: http
  url: "API_URL"
  parser:
    type: jsonl

out:
  type: postgresql
  host: localhost
  user: your_username
  password: your_password
  database: your_database
  table: covid_data
  mode: replace
Airflow DAGの作成:
データパイプラインのワークフローを設定します。
airflow_dag.py

python
コードをコピーする
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta
import requests
import json
import subprocess

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2024, 7, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

def fetch_covid_data():
    url = "API_URL"
    response = requests.get(url)
    data = response.json()
    with open('/path/to/save/data.json', 'w') as f:
        json.dump(data, f)

def run_embulk():
    subprocess.run(['embulk', 'run', 'path/to/config.yml'], check=True)

dag = DAG('covid_data_pipeline', default_args=default_args, schedule_interval='@weekly')

t1 = PythonOperator(
    task_id='fetch_covid_data',
    python_callable=fetch_covid_data,
    dag=dag,
)

t2 = PythonOperator(
    task_id='run_embulk',
    python_callable=run_embulk,
    dag=dag,
)

t1 >> t2
6. データの可視化
BIツールのセットアップ:
TableauやSupersetを使用してPostgreSQLに接続し、データを可視化します。
作業の進め方
EC2インスタンスのセットアップとSSH接続
必要なソフトウェアのインストール（Java、Embulk、Docker、PostgreSQL）
Airflowのインストールと設定
Embulk設定ファイルとAirflow DAGの作成
データパイプラインの実行と検証
BIツールを使用したデータの可視化
この手順を順番に実行することで、AWS上にEmbulkを使ったデータパイプラインを構築し、データの可視化を行うポートフォリオを作成することができます。何か不明点があれば、随時質問してください。
