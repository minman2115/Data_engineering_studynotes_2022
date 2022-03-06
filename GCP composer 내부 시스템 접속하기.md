.

Data_Engineering_TIL(20220306)

참고자료 : https://cloud.google.com/composer/docs/how-to/using/installing-python-dependencies#viewing_installed_python_packages

```console
$ gcloud composer environments list --project=minman-test --locations=asia-northeast3
┌──────────────────┬─────────────────┬─────────┬──────────────────────────┐
│       NAME       │     LOCATION    │  STATE  │       CREATE_TIME        │
├──────────────────┼─────────────────┼─────────┼──────────────────────────┤
│ minman-test      │ asia-northeast3 │ RUNNING │ 2021-12-10T01:23:45.549Z │
└──────────────────┴─────────────────┴─────────┴──────────────────────────┘


Updates are available for some Cloud SDK components.  To install them,
please run:
  $ gcloud components update


To take a quick anonymous survey, run:
  $ gcloud survey

$ gcloud composer environments describe minman-composer --project=minman-test --location asia-northeast3
config:
  airflowUri: https://a123456789b123456p-tp.appspot.com
  dagGcsPrefix: gs://asia-northeast3-minman-test-com-a12345678-bucket/dags
  databaseConfig:
    machineType: db-n1-standard-3
  encryptionConfig: {}
  gkeCluster: projects/minman-test/zones/asia-northeast3-a/clusters/asia-northeast3-minman-test-com-c123456-gke
  maintenanceWindow:
    endTime: '1970-01-01T00:00:00Z'
    startTime: '1970-01-01T00:00:00Z'

...

      value: ::0/0
  workloadsConfig: {}
createTime: '2021-08-10T01:23:45.549Z'
name: projects/minman-test/locations/asia-northeast3/environments/minman-composer
state: RUNNING
updateTime: '2022-02-28T07:14:58.332466Z'
uuid: 123ab123-b123-123a-a123-ab2c1c123a1c

$ gcloud container clusters get-credentials projects/minman-test/zones/asia-northeast3-a/clusters/asia-northeast3-minman-test-com-c123456-gke --zone projects/minman-test/zones/asia-northeast3-a
Fetching cluster endpoint and auth data.
kubeconfig entry generated for asia-northeast3-minman-test-com-c123456-gke.

# 123.123.12.123/32 : your IP address example
$ gcloud container clusters update projects/minman-test/zones/asia-northeast3-a/clusters/asia-northeast3-minman-test-com-c123456-gke --enable-master-authorized-networks --master-authorized-networks 123.123.12.123/32 --region asia-northeast3

$ kubectl get pods --all-namespaces | grep Running
composer-x-xx-xx-airflow-x-x-x-a1234abc   airflow-scheduler-12abc1d123-1ab12                               2/2     Running             0          117m
composer-x-xx-xx-airflow-x-x-x-a1234abc   airflow-worker-a12c123ab-abcd1                                   2/2     Running             4          22h

...

kube-system                               pdcsi-node-b1a12                                                 2/2     Running             0          71d

$ kubectl -n composer-x-xx-xx-airflow-x-x-x-a1234abc exec -it airflow-worker-a12c123ab-abcd1 -c airflow-worker -- /bin/bash
airflow@airflow-worker-a12c123ab-abcd1:~$ pip list
Package                                  Version
---------------------------------------- --------------
absl-py                                  1.0.0
alembic                                  1.7.5
amqp                                     5.0.7
anyio                                    3.4.0
apache-airflow                           2.2.3+composer
apache-airflow-providers-apache-beam     3.1.0
apache-airflow-providers-cncf-kubernetes 3.0.1
apache-airflow-providers-ftp             2.0.1
apache-airflow-providers-google          6.3.0
apache-airflow-providers-hashicorp       2.1.1
apache-airflow-providers-http            2.0.2
apache-airflow-providers-imap            2.1.0
apache-airflow-providers-mysql           2.1.1
apache-airflow-providers-oracle          2.2.0
apache-airflow-providers-postgres        2.4.0
apache-airflow-providers-sendgrid        2.0.1
apache-airflow-providers-slack           4.2.0
apache-airflow-providers-sqlite          2.0.1
apache-airflow-providers-ssh             2.3.0
apache-airflow-providers-tableau         2.1.4
apache-beam                              2.35.0
apispec                                  3.3.2
argcomplete                              1.12.3

...

zipp                                     3.6.0
```