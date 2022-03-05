.

Data_Engineering_TIL(20220305)

### [Overview]

Google Cloud Pub/Sub is a messaging service for exchanging event data among applications and services. A producer of data publishes messages to a Cloud Pub/Sub topic. A consumer creates a subscription to that topic. Subscribers either pull messages from a subscription or are configured as webhooks for push subscriptions. Every subscriber must acknowledge each message within a configurable window of time.

Dataflow is a fully-managed service for transforming and enriching data in stream (real-time) and batch modes with equal reliability and expressiveness. It provides a simplified pipeline development environment using the Apache Beam SDK, which has a rich set of windowing and session analysis primitives as well as an ecosystem of source and sink connectors.

Pub/Sub is a scalable, durable event ingestion and delivery system. Dataflow compliments Pub/Sub's scalable, at-least-once delivery model with message deduplication and exactly-once, in-order processing if you use windows and buffering.

### [What You'll Do]

Read messages published to a Pub/Sub topic

Window (or group) the messages by timestamp

Write the messages to Cloud Storage

### [실습 아키텍처]

```text
cloudshell --> pub/sub --> dataflow --> gcs

** cloudshell : 이번 실습에서는 자원생성하는 터미널이자 실시간 데이터를 생산하는 머신 역할을 하게됨
```

### [실습 요약]

STEP 1) 실습 리소스 생성

STEP 2) Start the Pipeline

STEP 3) Observe Job and Pipeline Progress

STEP 4) Cleanup

### [실습 상세내용]

GCP 클라우드 쉘을 열고 아래와 같이 명령어를 실행하면 위와 같은 실습아키텍처를 구현할 수 있다.

STEP 1) 실습 리소스 생성

```console
Welcome to Cloud Shell! Type "help" to get started.
To set your Cloud Platform project in this session use “gcloud config set project [PROJECT_ID]”
student_03_8b4d04139f65@cloudshell:~$ gcloud auth list
Credentialed Accounts

ACTIVE: *
ACCOUNT: student-03-8b4d04139f65@qwiklabs.net

To set the active account, run:
    $ gcloud config set account `ACCOUNT`

student_03_8b4d04139f65@cloudshell:~$ gcloud config set project qwiklabs-gcp-01-7a0357624d5f
Updated property [core/project].

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ PROJECT_ID=$(gcloud config get-value project)
Your active configuration is: [cloudshell-21702]

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ BUCKET_NAME=$PROJECT_ID

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ TOPIC_ID=student-03-8b4d04139f65

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ REGION=us-central1

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ AE_REGION=us-central

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ gsutil mb gs://$BUCKET_NAME
Creating gs://qwiklabs-gcp-01-7a0357624d5f/...

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ gcloud pubsub topics create $TOPIC_ID
Created topic [projects/qwiklabs-gcp-01-7a0357624d5f/topics/student-03-8b4d04139f65].

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ gcloud app create --region=$AE_REGION
You are creating an app for project [qwiklabs-gcp-01-7a0357624d5f].
WARNING: Creating an App Engine application for a project is irreversible and the region
cannot be changed. More information about regions is at
<https://cloud.google.com/appengine/docs/locations>.

Creating App Engine application in project [qwiklabs-gcp-01-7a0357624d5f] and region [us-central]....done.     
Success! The app is now created. Please use `gcloud app deploy` to deploy your first app.
student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ gcloud scheduler jobs create pubsub publisher-job --schedule="* * * * *" --topic=$TOPIC_ID --message-body="Hello!"
API [cloudscheduler.googleapis.com] not enabled on project [858729720438]. Would you like to enable and retry (this will take a few minutes)? (y/N)?  y

Enabling service [cloudscheduler.googleapis.com] on project [858729720438]...
Operation "operations/acf.p2-858729720438-cd1153ac-334e-49f8-8e9c-cbfb10f37679" finished successfully.
WARNING: We are using the App Engine app location (us-central1) as the default location. Please use the "--location" flag if you want to use a different location.
name: projects/qwiklabs-gcp-01-7a0357624d5f/locations/us-central1/jobs/publisher-job
pubsubTarget:
  data: SGVsbG8h
  topicName: projects/qwiklabs-gcp-01-7a0357624d5f/topics/student-03-8b4d04139f65
retryConfig:
  maxBackoffDuration: 3600s
  maxDoublings: 16
  maxRetryDuration: 0s
  minBackoffDuration: 5s
schedule: '* * * * *'
state: ENABLED
timeZone: Etc/UTC
userUpdateTime: '2022-03-05T07:22:27Z'

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5)$ gcloud scheduler jobs run publisher-job
WARNING: We are using the App Engine app location (us-central1) as the default location. Please use the "--location" flag if you want to use a different location.

# cloud shell 디폴트 python 버전이 3.9 버전대인데 이거를 그대로 사용하면 apache-beam 실행시 파이썬 버전 지원이 안된다고 에러가 발생한다. 그래서 파이썬 3.7이나 3.8을 설치해야 한다.
# Install requirements
student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$  student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$ sudo apt-get install -y build-essential checkinstall libreadline-gplv2-dev libncursesw5-dev libssl-dev libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev zlib1g-dev openssl libffi-dev python3-dev python3-setuptools wget 

# Prepare to build
student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$  mkdir /tmp/Python37

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$  cd /tmp/Python37

# Pull down Python 3.7, build, and install
student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$  wget https://www.python.org/ftp/python/3.7.0/Python-3.7.0.tar.xz

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$  tar xvf Python-3.7.0.tar.xz

student_03_8b4d04139f65@cloudshell:~ (qwiklabs-gcp-01-7a0357624d5f)$  cd /tmp/Python37/Python-3.7.0

student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0 (qwiklabs-gcp-01-7a0357624d5f)$ ./configure

student_03_8b4d04139f65@cloudshell/tmp/Python37/Python-3.7.0 (qwiklabs-gcp-01-7a0357624d5f)$  sudo make altinstall

student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0 (qwiklabs-gcp-01-7a0357624d5f)$ virtualenv env -p python3.7.0
created virtual environment CPython3.7.0.final.0-64 in 1354ms
  creator CPython3Posix(dest=/tmp/Python37/Python-3.7.0/env, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/student_03_8b4d04139f65/.local/share/virtualenv)
    added seed packages: pip==22.0.3, setuptools==60.6.0, wheel==0.37.1
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator

student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0$ source env/bin/activate

(env) student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0$ python --version
Python 3.7.0
```

STEP 2) Start the Pipeline

아래와 같은 샘플 코드를 실행하여 데이터 파이프라인을 구동할 것이다.

```text
This sample code uses Dataflow to:
Read Pub/Sub messages.
Window (or group) messages into fixed-size intervals by publish timestamps.
Write the messages in each window to files in Cloud Storage.
```

샘플코드 : PubSubToGCS.py

```python
import argparse
from datetime import datetime
import logging
import random
from apache_beam import DoFn, GroupByKey, io, ParDo, Pipeline, PTransform, WindowInto, WithKeys
from apache_beam.options.pipeline_options import PipelineOptions
from apache_beam.transforms.window import FixedWindows
class GroupMessagesByFixedWindows(PTransform):
    """A composite transform that groups Pub/Sub messages based on publish time
    and outputs a list of tuples, each containing a message and its publish time.
    """
    def __init__(self, window_size, num_shards=5):
        # Set window size to 60 seconds.
        self.window_size = int(window_size * 60)
        self.num_shards = num_shards
    def expand(self, pcoll):
        return (
            pcoll
            # Bind window info to each element using element timestamp (or publish time).
            | "Window into fixed intervals"
            >> WindowInto(FixedWindows(self.window_size))
            | "Add timestamp to windowed elements" >> ParDo(AddTimestamp())
            # Assign a random key to each windowed element based on the number of shards.
            | "Add key" >> WithKeys(lambda _: random.randint(0, self.num_shards - 1))
            # Group windowed elements by key. All the elements in the same window must fit
            # memory for this. If not, you need to use `beam.util.BatchElements`.
            | "Group by key" >> GroupByKey()
        )
class AddTimestamp(DoFn):
    def process(self, element, publish_time=DoFn.TimestampParam):
        """Processes each windowed element by extracting the message body and its
        publish time into a tuple.
        """
        yield (
            element.decode("utf-8"),
            datetime.utcfromtimestamp(float(publish_time)).strftime(
                "%Y-%m-%d %H:%M:%S.%f"
            ),
        )
class WriteToGCS(DoFn):
    def __init__(self, output_path):
        self.output_path = output_path
    def process(self, key_value, window=DoFn.WindowParam):
        """Write messages in a batch to Google Cloud Storage."""
        ts_format = "%H:%M"
        window_start = window.start.to_utc_datetime().strftime(ts_format)
        window_end = window.end.to_utc_datetime().strftime(ts_format)
        shard_id, batch = key_value
        filename = "-".join([self.output_path, window_start, window_end, str(shard_id)])
        with io.gcsio.GcsIO().open(filename=filename, mode="w") as f:
            for message_body, publish_time in batch:
                f.write(f"{message_body},{publish_time}\n".encode("utf-8"))
def run(input_topic, output_path, window_size=1.0, num_shards=5, pipeline_args=None):
    # Set `save_main_session` to True so DoFns can access globally imported modules.
    pipeline_options = PipelineOptions(
        pipeline_args, streaming=True, save_main_session=True
    )
    with Pipeline(options=pipeline_options) as pipeline:
        (
            pipeline
            # Because `timestamp_attribute` is unspecified in `ReadFromPubSub`, Beam
            # binds the publish time returned by the Pub/Sub server for each message
            # to the element's timestamp parameter, accessible via `DoFn.TimestampParam`.
            # https://beam.apache.org/releases/pydoc/current/apache_beam.io.gcp.pubsub.html#apache_beam.io.gcp.pubsub.ReadFromPubSub
            | "Read from Pub/Sub" >> io.ReadFromPubSub(topic=input_topic)
            | "Window into" >> GroupMessagesByFixedWindows(window_size, num_shards)
            | "Write to GCS" >> ParDo(WriteToGCS(output_path))
        )
if __name__ == "__main__":
    logging.getLogger().setLevel(logging.INFO)
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--input_topic",
        help="The Cloud Pub/Sub topic to read from."
        '"projects//topics/".',
    )
    parser.add_argument(
        "--window_size",
        type=float,
        default=1.0,
        help="Output file's window size in minutes.",
    )
    parser.add_argument(
        "--output_path",
        help="Path of the output GCS file including the prefix.",
    )
    parser.add_argument(
        "--num_shards",
        type=int,
        default=5,
        help="Number of shards to use when writing windowed elements to GCS.",
    )
    known_args, pipeline_args = parser.parse_known_args()
    run(
        known_args.input_topic,
        known_args.output_path,
        known_args.window_size,
        known_args.num_shards,
        pipeline_args,
    )
```

아래의 명령어를 이용해서 파이프라인을 동작시켜보자.

```text
python PubSubToGCS.py \
    --project=$PROJECT_ID \
    --region=$REGION \
    --input_topic=projects/$PROJECT_ID/topics/$TOPIC_ID \
    --output_path=gs://$BUCKET_NAME/samples/output \
    --runner=DataflowRunner \
    --window_size=2 \
    --num_shards=2 \
    --temp_location=gs://$BUCKET_NAME/temp
```

```console
(env) student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0/python-docs-samples/pubsub/streaming-analytics (qwiklabs-gcp-01-7a0357624d5f)$ python PubSubToGCS.py --project=$PROJECT_ID --region=$REGION --input_topic=projects/$PROJECT_ID/topics/$TOPIC_ID --output_path=gs://$BUCKET_NAME/samples/output --runner=DataflowRunner --window_size=2 --num_shards=2 --temp_location=gs://$BUCKET_NAME/temp
INFO:apache_beam.runners.portability.stager:Downloading source distribution of the SDK from PyPi
INFO:apache_beam.runners.portability.stager:Executing command: ['/tmp/Python37/Python-3.7.0/env/bin/python', '-m', 'pip', 'download', '--dest', '/tmp/tmp19j5yx5_', 'apache-beam==2.33.0', '--no-deps', '--no-binary', ':all:']
INFO:apache_beam.runners.portability.stager:Staging SDK sources from PyPI: dataflow_python_sdk.tar
INFO:apache_beam.runners.portability.stager:Downloading binary distribution of the SDK from PyPi
INFO:apache_beam.runners.portability.stager:Executing command: ['/tmp/Python37/Python-3.7.0/env/bin/python', '-m', 'pip', 'download', '--dest', '/tmp/tmp19j5yx5_', 'apache-beam==2.33.0', '--no-deps', '--only-binary', ':all:', '--python-version', '37', '--implementation', 'cp', '--abi', 'cp37m', '--platform', 'manylinux1_x86_64']
INFO:apache_beam.runners.portability.stager:Staging binary distribution of the SDK from PyPI: apache_beam-2.33.0-cp37-cp37m-manylinux1_x86_64.whl
WARNING:root:Make sure that locally built Python SDK docker image has Python 3.7 interpreter.
INFO:root:Default Python SDK image for environment is apache/beam_python3.7_sdk:2.33.0
INFO:root:Using provided Python SDK container image: gcr.io/cloud-dataflow/v1beta3/python37-fnapi:2.33.0
INFO:root:Python SDK container image set to "gcr.io/cloud-dataflow/v1beta3/python37-fnapi:2.33.0" for Docker environment
INFO:apache_beam.runners.dataflow.internal.apiclient:Defaulting to the temp_location as staging_location: gs://qwiklabs-gcp-01-7a0357624d5f/temp
INFO:apache_beam.internal.gcp.auth:Setting socket default timeout to 60 seconds.
INFO:apache_beam.internal.gcp.auth:socket default timeout is 60.0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/pickled_main_session...
INFO:oauth2client.transport:Attempting refresh to obtain initial access_token
INFO:oauth2client.transport:Attempting refresh to obtain initial access_token
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/pickled_main_session in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/dataflow_python_sdk.tar...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/dataflow_python_sdk.tar in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/apache_beam-2.33.0-cp37-cp37m-manylinux1_x86_64.whl...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/apache_beam-2.33.0-cp37-cp37m-manylinux1_x86_64.whl in 2 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Starting GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/pipeline.pb...
INFO:apache_beam.runners.dataflow.internal.apiclient:Completed GCS upload to gs://qwiklabs-gcp-01-7a0357624d5f/temp/beamapp-student038b4d04139f65-0305073824-781086.1646465904.781712/pipeline.pb in 0 seconds.
INFO:apache_beam.runners.dataflow.internal.apiclient:Create job: <Job
 createTime: '2022-03-05T07:38:31.110117Z'
 currentStateTime: '1970-01-01T00:00:00Z'
 id: '2022-03-04_23_38_29-6012764980959823246'
 location: 'us-central1'
 name: 'beamapp-student038b4d04139f65-0305073824-781086'
 projectId: 'qwiklabs-gcp-01-7a0357624d5f'
 stageStates: []
 startTime: '2022-03-05T07:38:31.110117Z'
 steps: []
 tempFiles: []
 type: TypeValueValuesEnum(JOB_TYPE_STREAMING, 2)>
INFO:apache_beam.runners.dataflow.internal.apiclient:Created job with id: [2022-03-04_23_38_29-6012764980959823246]
INFO:apache_beam.runners.dataflow.internal.apiclient:Submitted job: 2022-03-04_23_38_29-6012764980959823246
INFO:apache_beam.runners.dataflow.internal.apiclient:To access the Dataflow monitoring console, please navigate to https://console.cloud.google.com/dataflow/jobs/us-central1/2022-03-04_23_38_29-6012764980959823246?project=qwiklabs-gcp-01-7a0357624d5f
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2022-03-04_23_38_29-6012764980959823246 is in state JOB_STATE_PENDING
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:29.926Z: JOB_MESSAGE_BASIC: Streaming Engine auto-enabled. Use --experiments=disable_streaming_engine to opt out.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:29.926Z: JOB_MESSAGE_BASIC: Dataflow Runner V2 auto-enabled. Use --experiments=disable_runner_v2 to opt out.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:31.885Z: JOB_MESSAGE_WARNING: Autoscaling is enabled for Dataflow Streaming Engine. Workers will scale between 1 and 100 unless maxNumWorkers is specified.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:32.122Z: JOB_MESSAGE_DETAILED: Autoscaling is enabled for job 2022-03-04_23_38_29-6012764980959823246. The number of workers will be between 1 and 100.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:32.146Z: JOB_MESSAGE_DETAILED: Autoscaling was automatically enabled for job 2022-03-04_23_38_29-6012764980959823246.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.091Z: JOB_MESSAGE_BASIC: Worker configuration: n1-standard-2 in us-central1-c.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.742Z: JOB_MESSAGE_DETAILED: Expanding SplittableParDo operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.780Z: JOB_MESSAGE_DETAILED: Expanding CollectionToSingleton operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.846Z: JOB_MESSAGE_DETAILED: Expanding CoGroupByKey operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.871Z: JOB_MESSAGE_DEBUG: Combiner lifting skipped for step Window into/Group by key: GroupByKey not followed by a combiner.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.895Z: JOB_MESSAGE_DETAILED: Expanding SplittableProcessKeyed operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.923Z: JOB_MESSAGE_DETAILED: Expanding GroupByKey operations into streaming Read/Write steps
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.990Z: JOB_MESSAGE_DETAILED: Lifting ValueCombiningMappingFns into MergeBucketsMappingFns
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.060Z: JOB_MESSAGE_DEBUG: Annotating graph with Autotuner information.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.112Z: JOB_MESSAGE_DETAILED: Fusing adjacent ParDo, Read, Write, and Flatten operations
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.143Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Window into fixed intervals into Read from Pub/Sub/Read
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.179Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Add timestamp to windowed elements into Window into/Window into fixed intervals
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.203Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Add key/Map(<lambda at util.py:780>) into Window into/Add timestamp to windowed elements
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.235Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Group by key/WriteStream into Window into/Add key/Map(<lambda at util.py:780>)
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.268Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Group by key/MergeBuckets into Window into/Group by key/ReadStream
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.302Z: JOB_MESSAGE_DETAILED: Fusing consumer Write to GCS into Window into/Group by key/MergeBuckets
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.339Z: JOB_MESSAGE_BASIC: Running job using Streaming Engine
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.370Z: JOB_MESSAGE_DEBUG: Workflow config is missing a default resource spec.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.399Z: JOB_MESSAGE_DEBUG: Adding StepResource setup and teardown to workflow graph.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.431Z: JOB_MESSAGE_DEBUG: Adding workflow start and stop steps.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.464Z: JOB_MESSAGE_DEBUG: Assigning stage ids.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.526Z: JOB_MESSAGE_DEBUG: Starting worker pool setup.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.555Z: JOB_MESSAGE_BASIC: Starting 1 workers in us-central1-c...
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2022-03-04_23_38_29-6012764980959823246 is in state JOB_STATE_RUNNING
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.620Z: JOB_MESSAGE_DEBUG: Starting worker pool setup.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:38.039Z: JOB_MESSAGE_DETAILED: Pub/Sub resources set up for topic 'projects/qwiklabs-gcp-01-7a0357624d5f/topics/student-03-8b4d04139f65'.
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2022-03-04_23_38_29-6012764980959823246 is in state JOB_STATE_PENDING
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:29.926Z: JOB_MESSAGE_BASIC: Streaming Engine auto-enabled. Use --experiments=disable_streaming_engine to opt out.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:29.926Z: JOB_MESSAGE_BASIC: Dataflow Runner V2 auto-enabled. Use --experiments=disable_runner_v2 to opt out.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:31.885Z: JOB_MESSAGE_WARNING: Autoscaling is enabled for Dataflow Streaming Engine. Workers will scale between 1 and 100 unless maxNumWorkers is specified.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:32.122Z: JOB_MESSAGE_DETAILED: Autoscaling is enabled for job 2022-03-04_23_38_29-6012764980959823246. The number of workers will be between 1 and 100.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:32.146Z: JOB_MESSAGE_DETAILED: Autoscaling was automatically enabled for job 2022-03-04_23_38_29-6012764980959823246.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.091Z: JOB_MESSAGE_BASIC: Worker configuration: n1-standard-2 in us-central1-c.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.742Z: JOB_MESSAGE_DETAILED: Expanding SplittableParDo operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.780Z: JOB_MESSAGE_DETAILED: Expanding CollectionToSingleton operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.846Z: JOB_MESSAGE_DETAILED: Expanding CoGroupByKey operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.871Z: JOB_MESSAGE_DEBUG: Combiner lifting skipped for step Window into/Group by key: GroupByKey not followed by a combiner.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.895Z: JOB_MESSAGE_DETAILED: Expanding SplittableProcessKeyed operations into optimizable parts.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.923Z: JOB_MESSAGE_DETAILED: Expanding GroupByKey operations into streaming Read/Write steps
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:35.990Z: JOB_MESSAGE_DETAILED: Lifting ValueCombiningMappingFns into MergeBucketsMappingFns
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.060Z: JOB_MESSAGE_DEBUG: Annotating graph with Autotuner information.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.112Z: JOB_MESSAGE_DETAILED: Fusing adjacent ParDo, Read, Write, and Flatten operations
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.143Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Window into fixed intervals into Read from Pub/Sub/Read
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.179Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Add timestamp to windowed elements into Window into/Window into fixed intervals
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.203Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Add key/Map(<lambda at util.py:780>) into Window into/Add timestamp to windowed elements
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.235Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Group by key/WriteStream into Window into/Add key/Map(<lambda at util.py:780>)
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.268Z: JOB_MESSAGE_DETAILED: Fusing consumer Window into/Group by key/MergeBuckets into Window into/Group by key/ReadStream
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.302Z: JOB_MESSAGE_DETAILED: Fusing consumer Write to GCS into Window into/Group by key/MergeBuckets
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.339Z: JOB_MESSAGE_BASIC: Running job using Streaming Engine
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.370Z: JOB_MESSAGE_DEBUG: Workflow config is missing a default resource spec.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.399Z: JOB_MESSAGE_DEBUG: Adding StepResource setup and teardown to workflow graph.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.431Z: JOB_MESSAGE_DEBUG: Adding workflow start and stop steps.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.464Z: JOB_MESSAGE_DEBUG: Assigning stage ids.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.526Z: JOB_MESSAGE_DEBUG: Starting worker pool setup.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.555Z: JOB_MESSAGE_BASIC: Starting 1 workers in us-central1-c...
INFO:apache_beam.runners.dataflow.dataflow_runner:Job 2022-03-04_23_38_29-6012764980959823246 is in state JOB_STATE_RUNNING
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:36.620Z: JOB_MESSAGE_DEBUG: Starting worker pool setup.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:38:38.039Z: JOB_MESSAGE_DETAILED: Pub/Sub resources set up for topic 'projects/qwiklabs-gcp-01-7a0357624d5f/topics/student-03-8b4d04139f65'.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:39:21.171Z: JOB_MESSAGE_DETAILED: Autoscaling: Raised the number of workers to 1 so that the pipeline can catch up with its backlog and keep up with its input rate.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:39:42.445Z: JOB_MESSAGE_DETAILED: Workers have started successfully.
INFO:apache_beam.runners.dataflow.dataflow_runner:2022-03-05T07:39:42.472Z: JOB_MESSAGE_DETAILED: Workers have started successfully.

^CTraceback (most recent call last):
  File "PubSubToGCS.py", line 133, in <module>
    pipeline_args,
  File "PubSubToGCS.py", line 97, in run
    | "Write to GCS" >> ParDo(WriteToGCS(output_path))
  File "/tmp/Python37/Python-3.7.0/env/lib/python3.7/site-packages/apache_beam/pipeline.py", line 587, in __exit__
    self.result.wait_until_finish()
  File "/tmp/Python37/Python-3.7.0/env/lib/python3.7/site-packages/apache_beam/runners/dataflow/dataflow_runner.py", line 1621, in wait_until_finish
    time.sleep(5.0)
KeyboardInterrupt


적당한 시간동안 실행시키다가 ctrl + c로 빠져나와본다.
```

STEP 3) Observe Job and Pipeline Progress

아래 그림과 같이 dataflow 콘솔에 가보면 파이프라인이 실행되고 있음을 확인할 수 있고, gcs에도 가보면 데이터가 쌓이는 것을 확인할 수 있다.

![1](https://user-images.githubusercontent.com/41605276/156874783-1807f80d-c16c-47e6-be48-16e0dda2e524.png)

![2](https://user-images.githubusercontent.com/41605276/156874792-fa5af1a8-a9a0-4c91-887b-0bfcdcd255b6.png)

```console
(env) student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0/python-docs-samples/pubsub/streaming-analytics (qwiklabs-gcp-01-7a0357624d5f)$ gsutil ls gs://${BUCKET_NAME}/samples/
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:38-07:40-1
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:40-07:42-0
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:40-07:42-1

env) student_03_8b4d04139f65@cloudshell:/tmp/Python37/Python-3.7.0/python-docs-samples/pubsub/streaming-analytics (qwiklabs-gcp-01-7a0357624d5f)$ gsutil ls gs://${BUCKET_NAME}/samples/
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:38-07:40-1
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:40-07:42-0
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:40-07:42-1
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:42-07:44-0
gs://qwiklabs-gcp-01-7a0357624d5f/samples/output-07:42-07:44-1
```

STEP 4) Cleanup

```console
$ gcloud scheduler jobs delete publisher-job
# If prompted Do you want to continue press Y and enter.
# Press ctrl + c in your Cloud Shell if it's still busy printing output of your Dataflow job.
# In the Dataflow console, stop the job.
# With your job selected from the Dataflow Console, press the Stop button. Select the Cancel bubble to cancel the pipeline without draining.

$ gcloud pubsub topics delete $TOPIC_ID
 
$ gsutil -m rm -rf "gs://${BUCKET_NAME}/samples/output*"

$ gsutil -m rm -rf "gs://${BUCKET_NAME}/temp/*"

$ gsutil rb gs://${BUCKET_NAME}
```