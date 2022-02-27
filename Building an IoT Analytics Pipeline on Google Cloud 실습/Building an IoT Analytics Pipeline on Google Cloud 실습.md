.

Data_Engineering_TIL(20220227)

[실습소개]

- Overview

The term Internet of Things (IoT) refers to the interconnection of physical devices with the global Internet. These devices are equipped with sensors and networking hardware, and each is globally identifiable. Taken together, these capabilities afford rich data about items in the physical world.

Cloud IoT Core is a fully managed service that allows you to easily and securely connect, manage, and ingest data from millions of globally dispersed devices. The service connects IoT devices that use the standard Message Queue Telemetry Transport (MQTT) protocol to other Google Cloud data services.

Cloud IoT Core has two main components:

(1) A device manager for registering devices with the service, so you can then monitor and configure them.

(2) A protocol bridge that supports MQTT, which devices can use to connect to Google Cloud.

- What You'll Learn

(1) Connect and manage MQTT-based devices using Cloud IoT Core (using simulated devices)

(2) Ingest a stream of information from Cloud IoT Core using Cloud Pub/Sub.

(3) Process the IoT data using Cloud Dataflow.

(4) Analyze the IoT data using BigQuery.

[실습 아키텍처]

```text

데이터 흐름은 아래와 같음

compute engine VM compute --> IoT Core --> pub/sub --> Dataflow --> Bigquery
(iot-device-simulator) 

** 전반적인 자원생성 등 컨트롤은 GCP cloud shell 이라는 터미널에서 진행함
```

[실습요약]

STEP 1) cloud shell에서의 기본적인 권한설정

STEP 2) Enable APIs

STEP 3) Ensure that the Dataflow API is successfully enabled

STEP 4) Create a Cloud Pub/Sub topic

STEP 5) Create a BigQuery table

STEP 6) Create a cloud storage bucket

STEP 7) Set up a Cloud Dataflow Pipeline

STEP 8) Prepare your compute engine VM

STEP 9) Create a registry for IoT devices

STEP 10) Create a Cryptographic Keypair

STEP 11) Add simulated devices to the registry

STEP 12) Run simulated devices

STEP 13) Analyze the Sensor Data Using BigQuery

[실습내용]

#### STEP 1) cloud shell에서의 기본적인 권한설정

로컬 터미널 역할은 GCP 우측 상단에 cloud shell에서 진행하기 때문에 cloud shell에서 아래와 같이 기본적인 권한설정을 해준다.

GCP 웹콘솔에 최초접속해서 우측상단에 "Activate Cloud Shell" 버튼을 클릭해서 클라우드 쉘 터미널을 아래와 같이 활성화한다.

<img width="1439" alt="0" src="https://user-images.githubusercontent.com/41605276/155843747-c1a83ef9-f2b0-4e66-9b2d-57b35e639a7d.png">

Ensure that the Dataflow API is successfully enabled

```console
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
student_03_23376c19c633@cloudshell:~$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-03-23376c19c633@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_03_23376c19c633@cloudshell:~$ gcloud config set project qwiklabs-gcp-00-c1186d5d3f7c
Updated property [core/project].

student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$
```

#### STEP 2) Enable APIs

디폴트로는 enable 되어 있는데 체크는 한번 해보자

step 1) In the Cloud Console, click Navigation menu > APIs & Services.

![1](https://user-images.githubusercontent.com/41605276/155843587-d3bed58d-647f-4922-b5d8-a9978332a56e.png)

step 2) Scroll down in the list of enabled APIs, and confirm that these APIs are enabled:

(1) Cloud IoT API

(2) Cloud Pub/Sub API

(3) Dataflow API

step 3) If one or more API is not enabled, click the ENABLE APIS AND SERVICES button at the top. Search for the APIs by name and enable each API for your current project.

#### STEP 3) Ensure that the Dataflow API is successfully enabled

Dataflow API 도 기본적으로는 enable 되어 있는데 체크는 한번 해보자

To ensure access to the necessary API, restart the connection to the Dataflow API.

(1) In the Cloud Console, enter Dataflow API in the top search bar. Click on the result for Dataflow API.

(2) Click Manage.

(3) Click Disable API.

If asked to confirm, click Disable.

(4) Click Enable.

When the API has been enabled again, the page will show the option to disable.

![2](https://user-images.githubusercontent.com/41605276/155843985-1288f89a-0b92-4e18-b08f-2ee59d3c19a1.png)

#### STEP 4) Create a Cloud Pub/Sub topic

Cloud Pub/Sub is an asynchronous global messaging service. By decoupling senders and receivers, it allows for secure and highly available communication between independently written applications. Cloud Pub/Sub delivers low-latency, durable messaging.

In Cloud Pub/Sub, publisher applications and subscriber applications connect with one another through the use of a shared string called a topic. A publisher application creates and sends messages to a topic. Subscriber applications create a subscription to a topic to receive messages from it.

In an IoT solution built with Cloud IoT Core, device telemetry data is forwarded to a Cloud Pub/Sub topic.

To define a new Cloud Pub/Sub topic:

(1) In the Cloud Console, go to Navigation menu > Pub/Sub > Topics.

(2) Click + CREATE TOPIC. The Create a topic dialog shows you a partial URL path.

(3) Add this string as your Topic ID: iotlab

(4) Click CREATE TOPIC.

위에 안내문 같이 웹콘솔에서 만들어도 되고 아래와 같이 cloud shell에서 생성할 수도 있다.

```console
student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ gcloud pubsub topics create iotlab
Created topic [projects/qwiklabs-gcp-00-c1186d5d3f7c/topics/iotlab].

student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ gcloud pubsub topics list
---
name: projects/qwiklabs-gcp-00-c1186d5d3f7c/topics/iotlab
```

(5) In the list of topics, you will see a new topic whose partial URL ends in iotlab. Click the three-dot icon at the right edge of its row to open the context menu. Choose View permissions.

<img width="1425" alt="3" src="https://user-images.githubusercontent.com/41605276/155844477-1254fa6e-f66d-4a17-bb9f-69dc0e5d0c1e.png">

(6) In the Permissions dialogue, click ADD PRINCIPAL and copy the below principal as New principals: cloud-iot@system.gserviceaccount.com

(7) From the Select a role menu, give the new member the Pub/Sub > Pub/Sub Publisher role.

(8) Click Save.

위에 안내문 같이 웹콘솔에서 권한을 부여해도 되고 아래와 같이 cloud shell에서 명령어로 부여할 수도 있다.

```console
student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ gcloud alpha pubsub topics add-iam-policy-binding iotlab --member="serviceAccount:cloud-iot@system.gserviceaccount.com" --role='roles/pubsub.publisher'
Updated IAM policy for topic [iotlab].
bindings:
- members:
  - serviceAccount:cloud-iot@system.gserviceaccount.com
  role: roles/pubsub.publisher
etag: BwXY-CxAXH4=
version: 1
```

#### STEP 5) Create a BigQuery table

아래와 같이 cloud shell에서 iotlabdataset라는 이름으로 빅쿼리 데이터셋을 만들고, sensordata라는 이름으로 테이블을 만들어보자. 스키마는 아래와 같다.

```console
student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ bq mk --dataset qwiklabs-gcp-00-c1186d5d3f7c:iotlabdataset
Dataset 'qwiklabs-gcp-00-c1186d5d3f7c:iotlabdataset' successfully created.

student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ bq mk --table qwiklabs-gcp-00-c1186d5d3f7c:iotlabdataset.sensordata timestamp:TIMESTAMP,device:STRING,temperature:FLOAT
Table 'qwiklabs-gcp-00-c1186d5d3f7c:iotlabdataset.sensordata' successfully created.
```

#### STEP 6) Create a cloud storage bucket

```console
student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ gsutil mb -l US gs://qwiklabs-gcp-00-c1186d5d3f7c-bucket
Creating gs://qwiklabs-gcp-00-c1186d5d3f7c-bucket/...
```

#### STEP 7) Set up a Cloud Dataflow Pipeline

Cloud Dataflow is a serverless way to carry out data analysis. In this lab, you will set up a streaming data pipeline to read sensor data from Pub/Sub, compute the maximum temperature within a time window, and write this out to BigQuery.

(1) In the Cloud Console, go to Navigation menu > Dataflow.

(2) In the top menu bar, click + CREATE JOB FROM TEMPLATE.

(3) In the job-creation dialog, for Job name, enter `iotlabflow`.

(4) For Regional Endpoint, choose the region as us-central1.

(5) For Dataflow template, choose Pub/Sub Topic to BigQuery. When you choose this template, the form updates to review new fields below.

(6) For Input Pub/Sub topic, enter `projects/` followed by your Project ID then add /topics/iotlab. The resulting string will look like this: `projects/qwiklabs-gcp-d2e509fed105b3ed/topics/iotlab`

(7) The BigQuery output table takes the form of Project ID:dataset.table (`:iotlabdataset.sensordata`). The resulting string will look like this: `qwiklabs-gcp-d2e509fed105b3ed:iotlabdataset.sensordata`

(8) For Temporary location, enter `gs://` followed by your Cloud Storage bucket name (should be your Project ID if you followed the instructions) then `/tmp/`. The resulting string will look like this: `gs://qwiklabs-gcp-d2e509fed105b3ed-bucket/tmp/`

(9) Click SHOW OPTIONAL PARAMETERS.

(10) For Max workers, enter 2.

(11) For Machine type, enter n1-standard-1.

(12) Click RUN JOB.

A new streaming job is started. You can now see a visual representation of the data pipeline.

위에 안내문 같이 웹콘솔에서 생성해도 되고 아래와 같이 cloud shell에서 명령어로 생성할 수도 있다.

```console
student_03_23376c19c633@cloudshell:~ (qwiklabs-gcp-00-c1186d5d3f7c)$ gcloud dataflow jobs run iotlabflow --gcs-location gs://dataflow-templates/latest/PubSub_to_BigQuery --region us-central1 --staging-location gs://qwiklabs-gcp-00-c1186d5d3f7c-bucket/tmp/ --max-workers 2 --parameters inputTopic=projects/qwiklabs-gcp-00-c1186d5d3f7c/topics/iotlab,outputTableSpec=qwiklabs-gcp-00-c1186d5d3f7c:iotlabdataset.sensordata
createTime: '2022-02-27T04:21:02.534725Z'
currentStateTime: '1970-01-01T00:00:00Z'
id: 2022-02-26_20_21_01-3942008062108993312
location: us-central1
name: iotlabflow
projectId: qwiklabs-gcp-00-c1186d5d3f7c
startTime: '2022-02-27T04:21:02.534725Z'
type: JOB_TYPE_STREAMING
```

** 참고사항

```text

위에 명령어는 원래는 위에 안내문에서 가이드 했듯이 --worker-machine-type n1-standard-1 옵션을 주고 생성을 시도하였는데 ERROR: (gcloud.dataflow.jobs.run) NOT_FOUND: (c7b7b3578273e897): The workflow could not be created. 에러가 계속 발생하여서 워커 타입옵션을 빼고 실행한거임

gcloud dataflow jobs run iotlabflow --gcs-location gs://dataflow-templates/latest/PubSub_to_BigQuery --region us-central1 --staging-location gs://qwiklabs-gcp-00-c1186d5d3f7c-bucket/tmp/ --worker-machine-type n1-standard-1 --max-workers 2 --parameters inputTopic=projects/qwiklabs-gcp-00-c1186d5d3f7c/topics/iotlab,outputTableSpec=qwiklabs-gcp-00-c1186d5d3f7c:iotlabdataset.sensordata
```

#### STEP 8) Prepare your compute engine VM

In your project, a pre-provisioned VM instance named `iot-device-simulator` will let you run instances of a Python script that emulate an MQTT-connected IoT device. Before you emulate the devices, you will also use this VM instance to populate your Cloud IoT Core device registry.

To connect to the `iot-device-simulator` VM instance:

(1) In the Cloud Console, go to Navigation menu > Compute Engine > VM Instances. You'll see your VM instance listed as `iot-device-simulator`.

(2) Click the SSH drop-down arrow and select `Open in browser window`.

<img width="1383" alt="4" src="https://user-images.githubusercontent.com/41605276/155868259-add4a8db-3033-4103-abf1-5c0410e7e108.png">

위에 그림과 같이 `Open in browser window` 를 클릭하면 별도의 웹브라우저 창으로 해당 머신의 터미널이 뜰 것이다.

<img width="1423" alt="5" src="https://user-images.githubusercontent.com/41605276/155868320-fa41569b-68bf-40a4-a8f2-8def17de6d63.png">

(3) In your SSH session, enter following commands to create a virtual environment.

해당 머신 터미널에서 아래와 같이 명령어를 실행해준다.

```console
Linux iot-device-simulator 4.19.0-18-cloud-amd64 #1 SMP Debian 4.19.208-1 (2021-09-29) x86_64
The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the individual files in /usr/share/doc/*/copyright.
Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent permitted by applicable law.
Creating directory '/home/student-03-23376c19c633'.
student-03-23376c19c633@iot-device-simulator:~$ sudo pip3 install virtualenv

student-03-23376c19c633@iot-device-simulator:~$ virtualenv -p python3 venv

student-03-23376c19c633@iot-device-simulator:~$ source venv/bin/activate

(venv) student-03-23376c19c633@iot-device-simulator:~$ gcloud init
Welcome! This command will take you through the configuration of gcloud.

Settings from your current configuration [default] are:
core:
  account: 1078489835382-compute@developer.gserviceaccount.com
  disable_usage_reporting: 'True'
  project: qwiklabs-gcp-00-c1186d5d3f7c

Pick configuration to use:
 [1] Re-initialize this configuration [default] with new settings 
 [2] Create a new configuration
Please enter your numeric choice:  1

...

Choose the account you would like to use to perform operations for this configuration:
 [1] 1078489835382-compute@developer.gserviceaccount.com
 [2] Log in with a new account
Please enter your numeric choice:  2

You are running on a Google Compute Engine virtual machine.
It is recommended that you use service accounts for authentication.

You can run:
  $ gcloud config set account `ACCOUNT`
to switch accounts if necessary.
Your credentials may be visible to others with access to this
virtual machine. Are you sure you want to authenticate with your personal account?

Do you want to continue (Y/n)?  y
Go to the following link in your browser:

 https://accounts.google.com/o/oauth2/auth?response_type=code&client_id=32555940559.apps.googleusercontent.com&redirect_uri=urn%3Aietf%3Awg%3Aoauth%3A2.0%3Aoob&scope=openid+httpsxxxxxxxxxxx

Enter verification code: 위에 웹브라우저에 접속해서 계정인증 후 발급받은 토큰값을 넣고 엔터

You are logged in as: [student-03-23376c19c633@qwiklabs.net].

Pick cloud project to use: 
 [1] anthos-demo-source
 [2] cloudshell-images
 [3] esoteric-quanta-324122
 [4] pso-vmaas-1
 [5] qwiklabs-gcp-00-c1186d5d3f7c
 [6] qwiklabs-resources
 [7] Create a new project
Please enter numeric choice or text value (must exactly match list item):  5

Your current project has been set to: [qwiklabs-gcp-00-c1186d5d3f7c].

Your project default Compute Engine zone has been set to [us-central1-a].
You can change it by running [gcloud config set compute/zone NAME].

Your project default Compute Engine region has been set to [us-central1].
You can change it by running [gcloud config set compute/region NAME].

Created a default .boto configuration file at [/home/student-03-23376c19c633/.boto]. See this file and
[https://cloud.google.com/storage/docs/gsutil/commands/config] for more

(venv) student-03-23376c19c633@iot-device-simulator:~$ sudo apt-get update

(venv) student-03-23376c19c633@iot-device-simulator:~$ sudo apt-get install python-pip openssl git -y

(venv) student-03-23376c19c633@iot-device-simulator:~$ pip install pyjwt paho-mqtt cryptography

(venv) student-03-23376c19c633@iot-device-simulator:~$ git clone http://github.com/GoogleCloudPlatform/training-data-analyst
```

#### STEP 9) Create a registry for IoT devices

To register devices, you must create a registry for the devices. The registry is a point of control for devices.

`iot-device-simulator` VM 터미널에서 아래와 같이 실행해준다.

```console
(venv) student-03-23376c19c633@iot-device-simulator:~$ export PROJECT_ID=qwiklabs-gcp-00-c1186d5d3f7c

(venv) student-03-23376c19c633@iot-device-simulator:~$ export MY_REGION=us-central1

(venv) student-03-23376c19c633@iot-device-simulator:~$ gcloud iot registries create iotlab-registry --project=$PROJECT_ID --region=$MY_REGION --event-notification-config=topic=projects/$PROJECT_ID/topics/iotlab
Created registry [iotlab-registry].
```

#### STEP 10) Create a Cryptographic Keypair

`iot-device-simulator` VM 터미널에서 아래와 같이 실행해준다.

To allow IoT devices to connect securely to Cloud IoT Core, you must create a cryptographic keypair.

In your SSH session on the `iot-device-simulator` VM instance, enter these commands to create the keypair in the appropriate directory:

```console
(venv) student-03-23376c19c633@iot-device-simulator:~$ cd $HOME/training-data-analyst/quests/iotlab/

(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes -out rsa_cert.pem -subj "/C
N=unused"
Generating a RSA private key
...+++++
..............................................+++++
writing new private key to 'rsa_private.pem'
-----
```

This openssl command creates an RSA cryptographic keypair and writes it to a file called `rsa_private.pem`.

```console
(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ ls
cloudiot_mqtt_example.py  cloudiot_mqtt_example_json.py  labinfra  pom.xml  rsa_cert.pem  rsa_private.pem  run_oncloud.sh  src

(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ cat rsa_private.pem
-----BEGIN PRIVATE KEY-----
xxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxx

...

xxxxxxxxxxxxxxxxx
xxxxxxxxxxxxxxxxx
-----END PRIVATE KEY-----
```

#### STEP 11) Add simulated devices to the registry

For a device to be able to connect to Cloud IoT Core, it must first be added to the registry.

In your SSH session on the `iot-device-simulator` VM instance, enter this command to create a device called `temp-sensor-buenos-aires`:

```console
(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ gcloud iot devices create temp-sensor-buenos-aires --project=$PROJECT_ID --region=$MY_REGION --registry=iotlab-registry --public-key path=rsa_cert.pem,type=rs256
Created device [temp-sensor-buenos-aires].
```

Enter this command to create a device called `temp-sensor-istanbul`:

```console
(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ gcloud iot devices create temp-sensor-istanbul --project=$PROJECT_ID --region=$MY_REGION --registry=iotlab-registry --public-key path=rsa_cert.pem,type=rs256
Created device [temp-sensor-istanbul].
```

#### STEP 12) Run simulated devices

In your SSH session on the iot-device-simulator VM instance, enter these commands to download the CA root certificates from pki.google.com to the appropriate directory:

```console
(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ cd $HOME/training-data-analyst/quests/iotlab/

(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ curl -o roots.pem -s -m 10 --retry 0 "https://pki.goog/roots.pem"

(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ python cloudiot_mqtt_example_json.py --project_id=$PROJECT_ID --cloud_region=$MY_REGION --registry_id=iotlab-registry --device_id=temp-sensor-buenos-aires --private_key_file=rsa_private.pem --message_type=event --algorithm=RS256 > buenos-aires-log.txt 2>&1 &
[1] 14310

(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ python cloudiot_mqtt_example_json.py --project_id=$PROJECT_ID --cloud_region=$MY_REGION --registry_id=iotlab-registry --device_id=temp-sensor-istanbul --private_key_file=rsa_private.pem --message_type=event --algorithm=RS256
Creating JWT using RS256 from private key file rsa_private.pem
Publishing message 1 of 100: '{'timestamp': 1645938060, 'device': 'temp-sensor-istanbul', 'temperature': 17.95215062408679}'
on_connect 0: No error. 
on_publish
Publishing message 2 of 100: '{'timestamp': 1645938061, 'device': 'temp-sensor-istanbul', 'temperature': 17.93978667617052}'
on_publish
Publishing message 3 of 100: '{'timestamp': 1645938062, 'device': 'temp-sensor-istanbul', 'temperature': 17.930482207001894}'
on_publish
Publishing message 4 of 100: '{'timestamp': 1645938063, 'device': 'temp-sensor-istanbul', 'temperature': 17.92328879519045}'
on_publish
Publishing message 5 of 100: '{'timestamp': 1645938064, 'device': 'temp-sensor-istanbul', 'temperature': 17.90621449536501}'
on_publish
Publishing message 6 of 100: '{'timestamp': 1645938065, 'device': 'temp-sensor-istanbul', 'temperature': 17.888048946494294}'
on_publish
Publishing message 7 of 100: '{'timestamp': 1645938066, 'device': 'temp-sensor-istanbul', 'temperature': 17.88218096323425}'
on_publish
Publishing message 8 of 100: '{'timestamp': 1645938067, 'device': 'temp-sensor-istanbul', 'temperature': 17.86960374439278}'
on_publish

...

Publishing message 95 of 100: '{'timestamp': 1645938154, 'device': 'temp-sensor-istanbul', 'temperature': 17.03200724041793}'
on_publish
Publishing message 96 of 100: '{'timestamp': 1645938155, 'device': 'temp-sensor-istanbul', 'temperature': 17.025247907218034}'
on_publish
Publishing message 97 of 100: '{'timestamp': 1645938156, 'device': 'temp-sensor-istanbul', 'temperature': 17.01341910151476}'
on_publish
Publishing message 98 of 100: '{'timestamp': 1645938157, 'device': 'temp-sensor-istanbul', 'temperature': 17.009774941659}'
on_publish
Publishing message 99 of 100: '{'timestamp': 1645938158, 'device': 'temp-sensor-istanbul', 'temperature': 16.998693455331075}'
on_publish
Publishing message 100 of 100: '{'timestamp': 1645938159, 'device': 'temp-sensor-istanbul', 'temperature': 16.989718969595128}'
on_publish
Finished.
[1]+  Done                    python cloudiot_mqtt_example_json.py --project_id=$PROJECT_ID --cloud_region=$MY_REGION --registry_id=iotlab-registry --device_id=temp-sensor-buenos-aires
 --private_key_file=rsa_private.pem --message_type=event --algorithm=RS256 > buenos-aires-log.txt 2>&1

(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ cat cloudiot_mqtt_example_json.py
```

```python
#!/usr/bin/env python
# Copyright 2017 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#         http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
"""Python sample for connecting to Google Cloud IoT Core via MQTT, using JWT.
This example connects to Google Cloud IoT Core via MQTT, using a JWT for device
authentication. After connecting, by default the device publishes 100 messages
to the device's MQTT topic at a rate of one per second, and then exits.
Before you run the sample, you must follow the instructions in the README
for this sample.
"""
import argparse
import datetime
import os
import time
import json
import jwt
import paho.mqtt.client as mqtt
import random
def create_jwt(project_id, private_key_file, algorithm):
    """Creates a JWT (https://jwt.io) to establish an MQTT connection.
        Args:
         project_id: The cloud project ID this device belongs to
         private_key_file: A path to a file containing either an RSA256 or
                 ES256 private key.
         algorithm: The encryption algorithm to use. Either 'RS256' or 'ES256'
        Returns:
            An MQTT generated from the given project_id and private key, which
            expires in 20 minutes. After 20 minutes, your client will be
            disconnected, and a new JWT will have to be generated.
        Raises:
            ValueError: If the private_key_file does not contain a known key.
        """
    token = {
            # The time that the token was issued at
            'iat': datetime.datetime.utcnow(),
            # The time the token expires.
            'exp': datetime.datetime.utcnow() + datetime.timedelta(minutes=60),
            # The audience field should always be set to the GCP project id.
            'aud': project_id
    }
    # Read the private key file.
    with open(private_key_file, 'r') as f:
        private_key = f.read()
    print('Creating JWT using {} from private key file {}'.format(
            algorithm, private_key_file))

    return jwt.encode(token, private_key, algorithm=algorithm)

def error_str(rc):
    """Convert a Paho error to a human readable string."""
    return '{}: {}'.format(rc, mqtt.error_string(rc))

def on_connect(unused_client, unused_userdata, unused_flags, rc):
    """Callback for when a device connects."""
    print('on_connect', error_str(rc))

def on_disconnect(unused_client, unused_userdata, rc):
    """Paho callback for when a device disconnects."""
    print('on_disconnect', error_str(rc))

def on_publish(unused_client, unused_userdata, unused_mid):
    """Paho callback when a message is sent to the broker."""
    print('on_publish')

def parse_command_line_args():
    """Parse command line arguments."""
    parser = argparse.ArgumentParser(description=(
            'Example Google Cloud IoT Core MQTT device connection code.'))
    parser.add_argument(
            '--project_id',
            default=os.environ.get('GOOGLE_CLOUD_PROJECT'),
            help='GCP cloud project name')
    parser.add_argument(
            '--registry_id', required=True, help='Cloud IoT Core registry id')
    parser.add_argument(
            '--device_id', required=True, help='Cloud IoT Core device id')
    parser.add_argument(
            '--private_key_file',
            required=True, help='Path to private key file.')
    parser.add_argument(
            '--algorithm',
            choices=('RS256', 'ES256'),
            required=True,
            help='Which encryption algorithm to use to generate the JWT.')
    parser.add_argument(
            '--cloud_region', default='us-central1', help='GCP cloud region')
    parser.add_argument(
            '--ca_certs',
            default='roots.pem',
            help=('CA root from https://pki.google.com/roots.pem'))
    parser.add_argument(
            '--num_messages',
            type=int,
            default=100,
            help='Number of messages to publish.')
    parser.add_argument(
            '--message_type',
            choices=('event', 'state'),
            default='event',
            required=True,
            help=('Indicates whether the message to be published is a '
                  'telemetry event or a device state message.'))
    parser.add_argument(
            '--mqtt_bridge_hostname',
            default='mqtt.googleapis.com',
            help='MQTT bridge hostname.')
    parser.add_argument(
            '--mqtt_bridge_port',
            default=8883,
            type=int,
            help='MQTT bridge port.')

    return parser.parse_args()

def main():
    args = parse_command_line_args()
    # Create our MQTT client. The client_id is a unique string that identifies
    # this device. For Google Cloud IoT Core, it must be in the format below.
    client = mqtt.Client(
            client_id=('projects/{}/locations/{}/registries/{}/devices/{}'
                       .format(
                               args.project_id,
                               args.cloud_region,
                               args.registry_id,
                               args.device_id)))
    # With Google Cloud IoT Core, the username field is ignored, and the
    # password field is used to transmit a JWT to authorize the device.
    client.username_pw_set(
            username='unused',
            password=create_jwt(
                    args.project_id, args.private_key_file, args.algorithm))
    # Enable SSL/TLS support.
    client.tls_set(ca_certs=args.ca_certs)

    # Register message callbacks. https://eclipse.org/paho/clients/python/docs/
    # describes additional callbacks that Paho supports. In this example, the
    # callbacks just print to standard out.
    client.on_connect = on_connect
    client.on_publish = on_publish
    client.on_disconnect = on_disconnect
    # Connect to the Google MQTT bridge.
    client.connect(args.mqtt_bridge_hostname, args.mqtt_bridge_port)
    # Start the network loop.
    client.loop_start()
    # Publish to the events or state topic based on the flag.
    sub_topic = 'events' if args.message_type == 'event' else 'state'
    mqtt_topic = '/devices/{}/{}'.format(args.device_id, sub_topic)
    random.seed(args.device_id)  # A given device ID will always generate
                                 # the same random data
    simulated_temp = 10 + random.random() * 20
    if random.random() > 0.5:
        temperature_trend = +1     # temps will slowly rise
    else:
        temperature_trend = -1     # temps will slowly fall

    # Publish num_messages mesages to the MQTT bridge once per second.
    for i in range(1, args.num_messages + 1):
        simulated_temp = simulated_temp + temperature_trend * random.normalvariate(0.01,0.005)
        payload = {"timestamp": int(time.time()), "device": args.device_id, "temperature": simulated_temp}
        print('Publishing message {} of {}: \'{}\''.format(
                i, args.num_messages, payload))
        jsonpayload =  json.dumps(payload,indent=4)
        # Publish "jsonpayload" to the MQTT topic. qos=1 means at least once
        # delivery. Cloud IoT Core also supports qos=0 for at most once
        # delivery.
        client.publish(mqtt_topic, jsonpayload, qos=1)
        # Send events every second. State should not be updated as often
        time.sleep(1 if args.message_type == 'event' else 5)
    # End the network loop and finish.
    client.loop_stop()
    print('Finished.')

if __name__ == '__main__':
    main()       
```

```console
(venv) student-03-23376c19c633@iot-device-simulator:~/training-data-analyst/quests/iotlab$ ls
buenos-aires-log.txt  cloudiot_mqtt_example.py  cloudiot_mqtt_example_json.py  labinfra  pom.xml  roots.pem  rsa_cert.pem  rsa_private.pem  run_oncloud.sh  src
```

Telemetry data will flow from the simulated devices through Cloud IoT Core to your Cloud Pub/Sub topic. In turn, your Dataflow job will read messages from your Pub/Sub topic and write their contents to your BigQuery table.

#### STEP 13) Analyze the Sensor Data Using BigQuery

위에 스탭 12까지 했는데 빅쿼리에 데이터가 쌓이지 않는 경우가 있다. 그런경우 dataflow 콘솔로 가보면 job failed 메세지가 떠 있는 것을 확인할 수 있다. 이런경우 compute engine의 서비스 어카운트에 대한 권한체크가 필요하다.

```text
Workflow failed. Causes: There was a problem refreshing your credentials.

Please check: 
1. Dataflow API is enabled for your project. 
2. Make sure both the Dataflow service account and the controller service account have sufficient permissions. If you are not specifying a controller service account, ensure the default Compute Engine service account [PROJECT_NUMBER]-compute@developer.gserviceaccount.com exists and has sufficient permissions. 

If you have deleted the default Compute Engine service account, you must specify a controller service account. 
For more information, see: https://cloud.google.com/dataflow/docs/concepts/security-and-permissions#security_and_permissions_for_pipelines_on_google_cloud_platform. , There is no cloudservices robot account for your project. 

Please ensure that the Dataflow API is enabled for your project.
```

여기까지 문제가 없다면 아까 생성했던 빅쿼리 테이블로 이동해서 아래와 같이 쿼리를 날려보자.

To analyze the data as it is streaming:

In the Cloud Console, open the Navigation menu and select BigQuery.

Enter the following query in the Query editor and click RUN:

```SQL
SELECT timestamp, device, temperature from iotlabdataset.sensordata
ORDER BY timestamp DESC
LIMIT 100
```

쿼리결과 예시

<img width="556" alt="6" src="https://user-images.githubusercontent.com/41605276/155869358-a4aad48c-8e5c-42be-b338-f0841c7dc8da.png">