# 

* The Airflow platform is a tool for describing, executing, and monitoring workflows
* In Airflow, a `DAG` – or a Directed Acyclic Graph – is a collection of all the tasks you want to run, organized in a way that reflects their relationships and dependencies
* 数据管道是指通过一系列步骤来完成某个数据处理任务的流程
* Apache Airflow是一个用于以编程方式开发和监控批量处理数据管道的平台

```shell
# centos7创建新用户并授权
# 当前为root用户
adduser spark
passwd spark
whereis sudoers
ls -l /etc/sudoers
# 如果只读，则需赋予写权限
chmod -v u+w /etc/sudoers
# 追加新增的用户
vi /etc/sudoers
	root    ALL=(ALL)       ALL  
	spark  ALL=(ALL)       ALL  #这个是新增的用户
# wq保存退出，这时候要记得将写权限收回
chmod -v u-w /etc/sudoers
# 使用新用户登录，使用sudo
su spark
sudo cat /etc/profile
# 如果不想需要输入密码怎么办，将最后一个ALL修改成NOPASSWD: ALL
```

* python安装方式

```shell
# 使用非root用户
su woody
cd ~
mkdir -p data/airflow/airflow_demo && cd data/airflow/airflow_demo
sudo yum install python3 python3-devel 
sudo yum install gcc
# python虚拟环境  装在用户级别
# pip3 install pipenv --user
# 使用国内镜像 -i https://pypi.tuna.tsinghua.edu.cn/simple/
pip3 install -i https://pypi.douban.com/simple pipenv --user
#mkdir airflow-env-demo && cd airflow-env-demo

# Pipenv: Command Not Found
# 系统执行命令时会到 /usr/bin/ 目录下去找的，就像python3一样，需要软链接一下
# 找一下pipenv的位置
sudo find / -name "pipenv"
# 创建软连接
sudo ln -s /home/spark/.local/bin/pipenv /usr/bin/pipenv
pipenv --version

# 修改pipenv镜像源
vi Pipfile
	url = "https://pypi.tuna.tsinghua.edu.cn/simple/"

# 解决pipenv Locking... 慢


# 安装虚拟环境
pipenv --three
# 进入虚拟环境
pipenv shell
# 退出虚拟环境
exit

# 安装airflow
# pip3 install -i https://pypi.douban.com/simple apache-airflow==1.10.9
pipenv install apache-airflow==1.10.9 
# 修复SQLAlchemy的兼容性问题
pipenv uninstall SQLAlchemy
# pip3 install SQLAlchemy==1.3.15 -i https://pypi.douban.com/simple
pipenv install SQLAlchemy==1.3.15

# 创建.evn文件
# AIRFLOW_HOME=当前目录
AIRFLOW_HOME=/home/woody/data/airflow/py/airflow-env-demo

# bash: airflow: 未找到命令


# 创建airflow db
airflow initdb
# 启动web server
airflow webserver -p 8080 $
# airflow webserver -p 8080
# 启动sceduler
airflow scheduler

To activate this project's virtualenv, run pipenv shell.
Alternatively, run a command inside the virtualenv with pipenv run.
```

![image-20200615135923233](/Users/dingyuanjie/Documents/study/github/woodyprogram/img/image-20200615135923233.png)









```shell
# root用户
mkdir -p /data/airflow/demo
cd /data/airflow/demo
yum install python3 python3-devel 
yum install gcc
pip3 install -i https://pypi.douban.com/simple pipenv

# 安装虚拟环境
pipenv --three
# 修改pipenv镜像源
vi Pipfile
	url = "https://pypi.tuna.tsinghua.edu.cn/simple/"

# 进入虚拟环境
pipenv shell
# 退出虚拟环境
exit

pipenv install apache-airflow==1.10.9 
# 修复SQLAlchemy的兼容性问题
pipenv uninstall SQLAlchemy
# pip3 install SQLAlchemy==1.3.15 -i https://pypi.douban.com/simple
pipenv install SQLAlchemy==1.3.15

# 创建.evn文件
# AIRFLOW_HOME=当前目录
AIRFLOW_HOME=/home/woody/data/airflow/py/airflow-env-demo


# FileNotFoundError: [Errno 2] No such file or directory: 'gunicorn': 'gunicorn'
find / -name "gunicorn"
ln -s /root/.local/share/virtualenvs/demo-L96duEIF/bin/gunicorn /usr/bin/gunicorn

# 创建airflow db
airflow initdb
# 启动web server
airflow webserver -p 8080 &
# airflow webserver -p 8080
# 启动sceduler
airflow scheduler
# 访问
http://192.168.0.223:8080

# 默认的配置为~/airflow
```



* 安装mysql

  ```shell
  # 默认的配置为~/airflow
  # sql_alchemy_conn = sqlite:////root/airflow/airflow.db
  # $AIRFLOW_HOME/airflow.cfg配置文件，进行如下修改
  sql_alchemy_conn = mysql://root:root@localhost:3306/airflow
  # sql_alchemy_conn = mysql://woody:IamWoody0108!@localhost:3306/airflow
  executor = LocalExecutor
  
  # 数据库用户名与密码均为root，airflow使用的数据库为airflow．使用如下命令创建对应的数据库:
  mysql> create database airflow;
  # 重新初始化服务器数据库:
  xiaosi@yoona:~$ airflow initdb
  # ImportError: No module named MySQLdb
  # sudo apt-get install python-mysqldb
  # yum install MySQL-python
  yum install mysql-devel
  pipenv install mysqlclient
  
  # 再次初始化
  xiaosi@yoona:~$ airflow initdb
  # 查看一下airflow数据库中做了哪些操作:
  mysql> use airflow;
  mysql> show tables;
  
  
  # Exception: Global variable explicit_defaults_for_timestamp needs to be on (1) for mysql
  vim /etc/my.cnf
  [mysqld]
  explicit_defaults_for_timestamp = 1
  
  systemctl restart mysqld
  ```

  

* 编写PythonOperator

  ```shell
  find / -name "airflow.cfg"
  # 默认的配置为~/airflow
  
  # 配置时区
  local_tz = pendulum.timezone("Asia/Shanghai")
  
  # $AIRFLOW_HOME/dags 在该目录中创建一个名为hello_world.py
  # 运行测试脚本
  pipenv run python3 /root/airflow/dags/hello_world.py
  
  # 运行一些命令来进一步验证这个脚本
  # print the list of active DAGs
  airflow list_dags
  # prints the list of tasks the "tutorial" dag_id
  airflow list_tasks tutorial
  # prints the hierarchy of tasks in the tutorial DAG
  airflow list_tasks tutorial --tree
  
  # 通过在特定日期运行实际的任务实例进行测试
  # 在此上下文中指定的日期是一个execution_date，它模拟在特定日期+时间上运行任务或dag的调度程序
  # airflow test命令在本地运行任务实例，将其日志输出到stdout（屏幕上），不会影响依赖关系，并且不会将状态（运行，成功，失败，...）发送到数据库。 它只是允许简单的测试单个任务实例
  # command layout: command subcommand dag_id task_id date
  # testing print_date
  airflow test tutorial print_date 2015-06-01
  # testing sleep
  airflow test tutorial sleep 2015-06-01
  # testing templated
  airflow test tutorial templated 2015-06-01
  
  # Backfill
  # 将遵照依赖关系，并将日志发送到文件中，与数据库通信以记录状态
  # optional, start a web server in debug mode in the background
  # airflow webserver --debug &
  # 此上下文中的日期范围是start_date和可选的end_date，用于使用此dag中的任务实例填充运行计划
  # start your backfill on a date range
  airflow backfill tutorial -s 2015-06-01 -e 2015-06-07
  
  airflow list_tasks example_hello_world_dag
  airflow test example_hello_world_dag date_task 20200618
  airflow test example_hello_world_dag sleep_task 20200618
  ```

  ```python
  # -*- coding: utf-8 -*-
   
  import airflow
  from airflow import DAG
  from airflow.operators.bash_operator import BashOperator
  from airflow.operators.python_operator import PythonOperator
  from datetime import timedelta
   
  #-------------------------------------------------------------------------------
  # these args will get passed on to each operator
  # you can override them on a per-task basis during operator initialization
   
  default_args = {
      'owner': 'jifeng.si',
      'depends_on_past': False,
      'start_date': airflow.utils.dates.days_ago(2),
      'email': ['1203745031@qq.com'],
      'email_on_failure': False,
      'email_on_retry': False,
      'retries': 1,
      'retry_delay': timedelta(minutes=5),
      # 'queue': 'bash_queue',
      # 'pool': 'backfill',
      # 'priority_weight': 10,
      # 'end_date': datetime(2016, 1, 1),
      # 'wait_for_downstream': False,
      # 'dag': dag,
      # 'adhoc':False,
      # 'sla': timedelta(hours=2),
      # 'execution_timeout': timedelta(seconds=300),
      # 'on_failure_callback': some_function,
      # 'on_success_callback': some_other_function,
      # 'on_retry_callback': another_function,
      # 'trigger_rule': u'all_success'
  }
   
  #-------------------------------------------------------------------------------
  # dag
   
  dag = DAG(
      'example_hello_world_dag',
      default_args=default_args,
      description='my first DAG',
      schedule_interval=timedelta(days=1))
   
  #-------------------------------------------------------------------------------
  # first operator
   
  date_operator = BashOperator(
      task_id='date_task',
      bash_command='date',
      dag=dag)
   
  #-------------------------------------------------------------------------------
  # second operator
   
  sleep_operator = BashOperator(
      task_id='sleep_task',
      depends_on_past=False,
      bash_command='sleep 5',
      dag=dag)
   
  #-------------------------------------------------------------------------------
  # third operator
   
  def print_hello():
      return 'Hello world!'
   
  hello_operator = PythonOperator(
      task_id='hello_task',
      python_callable=print_hello,
      dag=dag)
   
  #-------------------------------------------------------------------------------
  # dependencies
   
  sleep_operator.set_upstream(date_operator)
  hello_operator.set_upstream(date_operator)
  ```

  

* 概念

  ```shell
  #  默认参数
  # 如果将default_args字典传递给DAG，DAG将会将字典应用于其内部的任何Operator上。这很容易的将常用参数应用于多个Operator，而无需多次键入
  
  # 上下文管理器
  # DAG可用作上下文管理器，以自动为DAG分配新的Operator
  
  # Operators
  # DAG描述了如何运行工作流
  # Operator描述了工作流中的单个任务。Operator通常（但不总是）原子性的，这意味着它们可以独立存在，不需要与其他Operator共享资源。DAG将确保Operator按正确的顺序运行; 除了这些依赖之外，Operator通常独立运行。实际上，他们可能会运行在两台完全不同的机器上
  # 一般来说，如果两个Operator需要共享信息，如文件名或少量数据，则应考虑将它们组合成一个Operator。如果绝对不可避免，Airflow确实有一个名为XCom的Operator可以交叉通信
  # Airflow为Operator提供常见任务：
  BashOperator - 执行bash命令
  PythonOperator - 调用任意的Python函数
  EmailOperator - 发送邮件
  HTTPOperator - 发送 HTTP 请求
  SqlOperator - 执行 SQL 命令
  Sensor - 等待一定时间，文件，数据库行，S3键等...
  还有更多的特定Operator:DockerOperator，HiveOperator，S3FileTransferOperator，PrestoToMysqlOperator，SlackOperator ...
  
  # DAG分配
  # Operators只有在分配给DAG时，才会被Airflow加载
  # Operator不需要立即分配给DAG（以前dag是必需的参数）。但是一旦operator分配给DAG, 它就不能transferred或者unassigned. 当一个Operator创建时，通过延迟分配或甚至从其他Operator推断，可以让DAG得到明确的分配
  dag = DAG('my_dag', start_date=datetime(2016, 1, 1))
  # 明确指定DAG
  explicit_op = DummyOperator(task_id='op1', dag=dag)
  # 延迟分配
  deferred_op = DummyOperator(task_id='op2')
  deferred_op.dag = dag
  # 从其他Operator推断 (linked operators must be in the same DAG)
  inferred_op = DummyOperator(task_id='op3')
  inferred_op.set_upstream(deferred_op)
  
  # 位移组合
  # op1 >> op2 表示op1先运行，op2然后运行。可以组合多个Operator - 记住从左到右执行的链，总是返回最右边的对象
  op1 >> op2   等价于   op1.set_downstream(op2)
  op2 << op1   等价于   op2.set_upstream(op1)
  # 位移操作符也可以与DAG一起使用
  dag >> op1 >> op2  
  #等价于   
  op1.dag = dag
  op1.set_downstream(op2)
  
  DAG：描述工作发生的顺序
  Operator：执行某些工作的模板类
  Task：Operator的参数化实例
  TaskInstances(任务实例)：1）已分配给DAG的任务，2）具有DAG特定运行相关的状态
  通过组合DAG和Operators来创建TaskInstances，可以构建复杂的工作流
  ```

  

* docker安装方式

  ```shell
  # 安装docker-compose工具: 
  pip3 install docker-compose [--user]
  # 下载docker-compose.yml文件, 放在新建目录下
  # systemctl start docker
  # 进入目录执行docker-compose up -d创建容器并启动, 第一次执行需要下载镜像会需要一点时间
  # 容器启动成功后，检查用docker-compose ps;查看日志用docker-compose logs -f --tail 100 webserver
  
  # docker-compose.yml
  
  docker-compose exec webserver bash
  ```

  ```yaml
  version: '3.7'
  services:
      postgres:
          image: postgres:9.6
          environment:
              - POSTGRES_USER=airflow
              - POSTGRES_PASSWORD=airflow
              - POSTGRES_DB=airflow
          logging:
              options:
                  max-size: 10m
                  max-file: "3"
  
      webserver:
          image: puckel/docker-airflow:1.10.9
          restart: always
          depends_on:
              - postgres
          environment:
              - LOAD_EX=n
              - EXECUTOR=Local
          logging:
              options:
                  max-size: 10m
                  max-file: "3"
          volumes:
              - ./dags:/usr/local/airflow/dags
              # - ./plugins:/usr/local/airflow/plugins
          ports:
              - "8080:8080"
          command: webserver
          healthcheck:
              test: ["CMD-SHELL", "[ -f /usr/local/airflow/airflow-webserver.pid ]"]
              interval: 30s
              timeout: 30s
              retries: 3
  
  ```

## BashOperator

```python
# can use Jinja templates to parameterize the bash_command argument.
also_run_this = BashOperator(
    task_id='also_run_this',
    bash_command='echo "run_id={{ run_id }} | dag_run={{ dag_run }}"',
    dag=dag,
)

t2 = BashOperator(
    task_id='bash_example',
  	# 注意后面加空格
    bash_command="/home/batcher/test.sh ",
    dag=dag)
```

## PythonOperator

```python
def print_context(ds, **kwargs):
    pprint(kwargs)
    print(ds)
    return 'Whatever you return gets printed in the logs'

run_this = PythonOperator(
    task_id='print_the_context',
  	# When you set the provide_context argument to True, Airflow passes in an additional set of keyword arguments: one for each of the Jinja template variables and a templates_dict argument.
    provide_context=True,
    python_callable=print_context,
    dag=dag,
)


# Use the op_args and op_kwargs arguments to pass additional arguments to the Python callable
def my_sleeping_function(random_base):
    """This is a function that will run within the DAG execution"""
    time.sleep(random_base)

# Generate 5 sleeping tasks, sleeping from 0.0 to 0.4 seconds respectively
for i in range(5):
    task = PythonOperator(
        task_id='sleep_for_' + str(i),
        python_callable=my_sleeping_function,
        op_kwargs={'random_base': float(i) / 10},
        dag=dag,
    )

run_this >> task
```

## Jinja Templating

```python
# The execution date as YYYY-MM-DD
# Here, {{ ds }} is a macro, and because the env parameter of the BashOperator is templated with Jinja, the execution date will be available as an environment variable named EXECUTION_DATE in your Bash script
date = "{{ ds }}"
t = BashOperator(
    task_id='test_env',
    bash_command='/tmp/test.sh ',
    dag=dag,
    env={'EXECUTION_DATE': date})


# You can also use Jinja templating with nested fields, as long as these nested fields are marked as templated in the structure they belong to: fields registered in template_fields property will be submitted to template substitution, like the path field in the example below:
class MyDataReader:
  template_fields = ['path']

  def __init__(self, my_path):
    self.path = my_path

  # [additional code here...]

t = PythonOperator(
    task_id='transform_data',
    python_callable=transform_data
    op_args=[
      MyDataReader('/tmp/{{ ds }}/my_file')
    ],
    dag=dag)
```

## XComs

```shell
# XComs let tasks exchange messages, allowing more nuanced forms of control and shared state
# XComs are principally defined by a key, value, and timestamp, but also track attributes like the task/DAG that created the XCom and when it should become visible
# XComs can be “pushed” (sent) or “pulled” (received). When a task pushes an XCom, it makes it generally available to other tasks.
# Tasks can push XComs at any time by calling the xcom_push() method. In addition, if a task returns a value (either from its Operator’s execute() method, or from a PythonOperator’s python_callable function), then an XCom containing that value is automatically pushed
# Tasks call xcom_pull() to retrieve XComs, optionally applying filters based on criteria like key, source task_ids, and source dag_id
# By default, xcom_pull() filters for the keys that are automatically given to XComs when they are pushed by being returned from execute functions (as opposed to XComs that are pushed manually).
# If xcom_pull is passed a single string for task_ids, then the most recent XCom value from that task is returned; if a list of task_ids is passed, then a corresponding list of XCom values is returned
```



## Macros reference

```shell
# Variables and macros can be used in templates 
# Macros are a way to expose objects to your templates and live under the macros namespace in your templates.
```

## 官方Operator

### BashOperator

```python
from builtins import range
from datetime import timedelta

from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.dummy_operator import DummyOperator
from airflow.utils.dates import days_ago

args = {
    'owner': 'Airflow',
    'start_date': days_ago(2),
}

dag = DAG(
    dag_id='example_bash_operator',
    default_args=args,
    schedule_interval='0 0 * * *',
    dagrun_timeout=timedelta(minutes=60),
    tags=['example']
)

run_this_last = DummyOperator(
    task_id='run_this_last',
    dag=dag,
)

# [START howto_operator_bash]
run_this = BashOperator(
    task_id='run_after_loop',
    bash_command='echo 1',
    dag=dag,
)
# [END howto_operator_bash]

run_this >> run_this_last

for i in range(3):
    task = BashOperator(
        task_id='runme_' + str(i),
        bash_command='echo "{{ task_instance_key_str }}" && sleep 1',
        dag=dag,
    )
    task >> run_this

# [START howto_operator_bash_template]
also_run_this = BashOperator(
    task_id='also_run_this',
    bash_command='echo "run_id={{ run_id }} | dag_run={{ dag_run }}"',
    dag=dag,
)
# [END howto_operator_bash_template]
also_run_this >> run_this_last

if __name__ == "__main__":
    dag.cli()
```

### PythonOperator

```python
from __future__ import print_function

import time
from builtins import range
from pprint import pprint

from airflow.utils.dates import days_ago

from airflow.models import DAG
from airflow.operators.python_operator import PythonOperator

args = {
    'owner': 'Airflow',
    'start_date': days_ago(2),
}

dag = DAG(
    dag_id='example_python_operator',
    default_args=args,
    schedule_interval=None,
    tags=['example']
)

# [START howto_operator_python]
def print_context(ds, **kwargs):
    pprint(kwargs)
    print(ds)
    return 'Whatever you return gets printed in the logs'

run_this = PythonOperator(
    task_id='print_the_context',
    provide_context=True,
    python_callable=print_context,
    dag=dag,
)
# [END howto_operator_python]

# [START howto_operator_python_kwargs]
def my_sleeping_function(random_base):
    """This is a function that will run within the DAG execution"""
    time.sleep(random_base)

# Generate 5 sleeping tasks, sleeping from 0.0 to 0.4 seconds respectively
for i in range(5):
    task = PythonOperator(
        task_id='sleep_for_' + str(i),
        python_callable=my_sleeping_function,
        op_kwargs={'random_base': float(i) / 10},
        dag=dag,
    )
    run_this >> task
# [END howto_operator_python_kwargs]
```

## GP

```python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.python_operator import PythonOperator
from airflow.operators.mysql_operator import MySqlOperator
from airflow.operators.dummy_operator import DummyOperator
from gupao.operators.mysql2mysql_operators import MySQLToMySQLOperator
from datetime import datetime, timedelta
from airflow.models import Variable

import pendulum
import requests
from pprint import pprint
import os

# 配置时区
local_tz = pendulum.timezone("Asia/Shanghai")

default_args = {
    "owner": "magicwind",
    # 不依赖之前的任务是否成功
    "depends_on_past": False,
    # 开始时间
    "start_date": datetime(2020, 6, 8, tzinfo=local_tz),
    # 重试一次
    "retries": 1,
    # 重试间隔
    "retry_delay": timedelta(minutes=5)
}

app_id = Variable.get('exchange_rate_app_id')
if not app_id:
    print('exchange_rate_app_id is not configured')
    exit(0)

dag = DAG(
    "tutorial",
    default_args=default_args,
    # DAG RUN 超时时间
    dagrun_timeout=timedelta(minutes=10),
    # 定时调度配置
    schedule_interval="10 10 * * *",
    # 不要自动创建历史的DAG RUN
    catchup=False,
    # 每个DAG同时只能有一个DAG RUN在执行
    max_active_runs=1
)

t1 = BashOperator(task_id="print_date", bash_command="date", dag=dag)

t2 = BashOperator(task_id="sleep", bash_command="sleep 5", dag=dag)

templated_command = """
    {% for i in range(5) %}
        echo "{{ ds }}"
        echo "{{ macros.ds_add(ds, 7)}}"
        echo "{{ params.my_param }}"
    {% endfor %}
"""

t3 = BashOperator(
    task_id="templated",
    bash_command=templated_command,
    params={"my_param": "Parameter I passed in"},
    dag=dag,
)

const_filepath = '/tmp/exchange_rate.csv'


def get_exchange_rate(**context):
    print('********Start*********')
    tmp_file_path = context['templates_dict']['tmp_file_path']
    target_date = context['templates_dict']['target_date']
    in_app_id =  context['templates_dict']['app_id']
    print(f'tmp_file_path:{tmp_file_path}, target_date:{target_date}, in_app_id:{len(in_app_id)} chars')

    #5c2dba0f887544a4b196eeaa8a3052f4
    resp = requests.get(f'https://openexchangerates.org/api/historical/{target_date}.json?app_id={in_app_id}')
    result = resp.json()
    pprint(result)

    base_currency = result['base']
    data_timestamp = result['timestamp']
    date_date = datetime.utcfromtimestamp(data_timestamp).date().isoformat()
    rates = result['rates']

    rows = ['base,date,currency,rate']
    for key in rates:
        rate_in_dec = '{0:f}'.format(rates[key])
        rows.append(f'{base_currency},{date_date},{key},{rate_in_dec}\n')

    for r in rows:
        print(r[:-1])

    filepath = '/tmp/exchange_rate.csv'

    print(f'write result to file: {filepath}')
    if os.path.exists(filepath):
        os.remove(filepath)

    with open(filepath, 'w') as f:
        f.writelines(rows)

    print('********End*********')


get_exchange_rate = PythonOperator(
    task_id="get_exchange_rate",
    python_callable=get_exchange_rate,
    templates_dict={
        'tmp_file_path': const_filepath,
        'target_date': '{{ ds }}',
        'app_id': app_id
    },
    provide_context=True,
    dag=dag
)

# MySQL表的结构如下:
# CREATE TABLE `exchange_rate` (
#  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
#  `base` varchar(16) NOT NULL DEFAULT '',
#  `date` varchar(16) NOT NULL DEFAULT '',
#  `currency` varchar(16) NOT NULL DEFAULT '',
#  `rate` decimal(18,6) NOT NULL,
#  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
#  PRIMARY KEY (`id`)
# )

load_exchange_rate = MySqlOperator(
    task_id="load_exchange_rate",
    sql=f"load data local infile '{const_filepath}' into table exchange_rate FIELDS TERMINATED BY ',' IGNORE 1 lines (base,date,currency,rate)",
    mysql_conn_id='mysql_default',
    autocommit=True,
    database='airflow',
    dag=dag
)

# CREATE TABLE `exchange_rate_summary` (
#  `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
#  `stat_date` varchar(16) NOT NULL DEFAULT '',
#  `currency` varchar(16) NOT NULL DEFAULT '',
#  `min_rate` decimal(18,6) NOT NULL,
#  `max_rate` decimal(18,6) NOT NULL,
#  `created_at` timestamp NOT NULL DEFAULT current_timestamp(),
#  PRIMARY KEY (`id`)
# )

mysql2mysql_task = MySQLToMySQLOperator(
    task_id='mysql2mysql_task',
    destination_table='exchange_rate_summary',
    source_conn_id='mysql_default',
    destination_conn_id='mysql_default',
    sql="select current_date, currency, min(rate), max(rate) from exchange_rate where date >= date_sub(current_date, interval 7 day) group by currency",
    target_fields=['stat_date', 'currency', 'min_rate', 'max_rate'],
    dag=dag
)

t1 >> t2
t3.set_upstream(t1)

[t2, t3] >> get_exchange_rate

get_exchange_rate >> load_exchange_rate >> mysql2mysql_task

# 演示通过代码生产Task, 注意效率, 不能访问外部资源
tail_task = load_exchange_rate
for i in range(2):
    dummy_task = DummyOperator(
        task_id=f'dummy_task{i}',
        dag=dag
    )

    dummy_task.set_upstream(tail_task)
    tail_task = dummy_task

if __name__ == '__main__':
    dag.cli()
```







```python
from airflow import DAG
# ModuleNotFoundError: No module named 'airflow.operators.python'
# from airflow.operators.python import PythonOperator
from airflow.operators.python_operator import PythonOperator
from airflow.utils.dates import days_ago

args = {
    'owner': 'airflow',
    'start_date': days_ago(2),
}

# schedule_interval 是任务时间设定:与Linux cron 时间是不同的
# airflow cron 表达式: * * * * * * (分 时 月 年 周 秒)
dag = DAG('example_external_first', schedule_interval="*/1 * * * *", default_args=args, tags=['example'])

def external_first_m_1(**kwargs):
  print('external_first_m11111111111111111111')
    
def external_first_m_2(**kwargs):
  print('external_first_m22222222222222222222')

external1 = PythonOperator(
    task_id='external_first_op_first',
    dag=dag,
    python_callable=external_first_m_1,
)

external2 = PythonOperator(
    task_id='external_first_op_second',
    dag=dag,
    python_callable=external_first_m_2,
)

external2 << external1
```

```python
from airflow import DAG
# ModuleNotFoundError: No module named 'airflow.operators.python'
# from airflow.operators.python import PythonOperator
from airflow.operators.python_operator import PythonOperator
from airflow.utils.dates import days_ago
from datetime import datetime, timedelta
from airflow.operators.sensors import ExternalTaskSensor

args = {
    'owner': 'airflow',
    'start_date': days_ago(2),
}

dag = DAG('example_external_second', schedule_interval=timedelta(seconds=5), default_args=args, tags=['example'])

def external_secnod_m_1(**kwargs):
  print('external_secnod_m_1111111111111111')
    
def external_secnod_m_2(**kwargs): 
  print('external_secnod_m_22222222222222222')
  
wait_for_first = ExternalTaskSensor(
    task_id='wait_for_first',
    external_dag_id='example_external_first',
    external_task_id='external_first_op_second',
    execution_delta=timedelta(seconds=5),
    dag=dag)  

external_secnod_1 = PythonOperator(
    task_id='external_secnod_1',
    dag=dag,
    python_callable=external_secnod_m_1,
)

external_secnod_2 = PythonOperator(
    task_id='external_secnod_2',
    dag=dag,
    python_callable=external_secnod_m_2,
)

wait_for_first >> external_secnod_1 >> external_secnod_2
```





```python
from builtins import range
from datetime import timedelta

from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.dummy_operator import DummyOperator
from airflow.utils.dates import days_ago

# 配置时区
local_tz = pendulum.timezone("Asia/Shanghai")

args = {
    'owner': 'airflow',
    'start_date': days_ago(2),
}

dag = DAG('example_bash_sh', schedule_interval=timedelta(seconds=2), default_args=args, tags=['example'])

bash_1 = BashOperator(
    task_id='bash_add',
  	# chmod +x add2.sh
    bash_command="/data/airflow/demo/bash_op/add.sh ",
    dag=dag)

bash_1
```

```python
from builtins import range
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
from datetime import datetime, timedelta
from airflow.operators.sensors import ExternalTaskSensor

# 配置时区
local_tz = pendulum.timezone("Asia/Shanghai")

args = {
    'owner': 'airflow',
    'start_date': days_ago(2),
}

dag = DAG('example_bash_sh_external', schedule_interval=timedelta(hours=1), default_args=args, tags=['example'])

wait_for_add1 = ExternalTaskSensor(
    task_id='wait_for_add_1',
    external_dag_id='example_bash_sh',
    external_task_id='bash_add',
    execution_delta=timedelta(hours=1),
    dag=dag) 

bash_2 = BashOperator(
    task_id='bash_add2',
  	# chmod +x add2.sh
    bash_command="/data/airflow/demo/bash_op/add2.sh ",
    dag=dag)

wait_for_add1 >> bash_2
```

```shell
# The scheduler does not appear to be running. Last heartbeat was received 1 minute ago.
# https://stackoverflow.com/questions/57668584/airflow-scheduler-does-not-appear-to-be-running-after-excute-a-task
I think it is expected for Sequential Executor. Sequential Executor runs one thing at a time so it cannot run heartbeat and task at the same time.
Why do you need to use Sequential Executor / Sqlite? The advice to switch to other DB/Executor make perfect sense.

# vim airflow.cfg
executor = LocalExecutor


# airflow.exceptions.AirflowConfigException: error: cannot use sqlite with the LocalExecutor
```



## triggerDagRunOperator

```python
# [root@woody dags]# cat triggerDagRun_1.py
from builtins import range
from datetime import timedelta

from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.operators.dummy_operator import DummyOperator
from airflow.operators.dagrun_operator import TriggerDagRunOperator
from airflow.utils.dates import days_ago
import pendulum

# 配置时区
local_tz = pendulum.timezone("Asia/Shanghai")

args = {
    'owner': 'airflow',
    'start_date': days_ago(2),
}

dag = DAG('example_trigger_run_1', schedule_interval=timedelta(seconds=2), default_args=args, tags=['example'])

print_1 = BashOperator(
    task_id='print_111',
    bash_command="/data/airflow/demo/oper/print_1.sh ",
    dag=dag)

# example_bash_sh_external_1
trigger = TriggerDagRunOperator(
    task_id='trigger_t',
    trigger_dag_id='example_trigger_run_2',
    python_callable=None,
    execution_date=None,
    dag=dag)

print_1 >> trigger
```

```python
# [root@woody dags]# cat triggerDagRun_2.py
from builtins import range
from airflow.models import DAG
from airflow.operators.bash_operator import BashOperator
from airflow.utils.dates import days_ago
from datetime import datetime, timedelta
from airflow.operators.sensors import ExternalTaskSensor
import pendulum

# 配置时区
local_tz = pendulum.timezone("Asia/Shanghai")

args = {
    'owner': 'airflow',
    'start_date': days_ago(2),
}

# dag = DAG('example_bash_sh_external_1', schedule_interval=timedelta(seconds=2), default_args=args, tags=['example'])
dag = DAG('example_trigger_run_2', schedule_interval=None, default_args=args, tags=['example'])

#wait_for_add1 = ExternalTaskSensor(
#    task_id='wait_for_add_1',
#    external_dag_id='example_bash_sh',
#    external_task_id='bash_add',
#    execution_delta=timedelta(seconds=2),
#    dag=dag)

print_2 = BashOperator(
    task_id='print_222',
    bash_command="/data/airflow/demo/oper/print_2.sh ",
    dag=dag)

#wait_for_add1 >> bash_2
print_2
```

```shell
# [root@woody bash_op]# cat add.sh
#!/bin/bash

echo '111' >> /data/airflow/demo/bash_op/t.txt

# [root@woody bash_op]# cat add2.sh
#!/bin/bash

echo "23333" >> /data/airflow/demo/bash_op/t.txt
```

