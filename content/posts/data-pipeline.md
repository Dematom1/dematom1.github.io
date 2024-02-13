+++
title = 'Data Pipeline'
date = 2024-01-16T17:50:54-07:00
draft = false
+++


# Project Overview
This project will simulate a end to end data pipeline for [Data First Jobs](www.datafirstjobs.com). It will consist of the following tools:
    * Kafka
    * Airflow
    * DBT
    * Metabase


I built [Data First Jobs](www.datafirstjobs.com) because not only was I tired of filtering through multiple job boards, but many of those job board did not consistenlty categorize data jobs. In other words, sometimes there were under "other", somethings they were under engineeers, but usually they were a mix and few and far between.

## Scenario
We have users, clicks, and applications. Let's pretend we get notified of a users successful application, that is they clicked through the job in the list view, to the detail view, where they went on and applied successfully on the company's website.

The issue is we need to attribute successful applications to users, the successful_application event will give us the email they used to apply, in this scenario this will be enough to attribute the application event to users, and their clicks.

Since the website doesn't have too much data at the moment, I will need to generate fake data to push through Kafka to simulate some real time stream processing, with flink thrown in there to handle the more "complex" transformation. We will feed this fake data through Kafka + Flink and save the results to a Postgres DB that will serve as a sink.

First I will generate fake data using Faker. I will put this in a seperate container, and run it on initialization with docker compose, so that when the postgres db is ready, it will create the tables required for the datasets as well as the final sink table.

Below is the click and application event that simulates a user clicking on the home page, as well as the search view where they may or may not have selected a category to filter through, before potentially successfully applying for a job. [Link](https://github.com/Dematom1/data_server/blob/Master/containers/fake_data/gen_fake_data.py)

These events are generated in a datastream, where they are subsequently pushed to Kafka. Who is allow for auto creating of topics.

```python
def gen_click_event(user_id: str, job_uuid: str = None) -> dict:
        user_agent: str = fake.user_agent()
        event_time: datetime = datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")[:-3]
        event_type: str = fake.job_event_type()
        template_name: str  = fake.job_template_name()
        element: str = fake.job_element()
        job_uuid: str = job_uuid or str(uuid4())
        ip_address: str = fake.ipv4()
        
        click_event = {
            'user_agent': user_agent,
            'event_time': event_time,
            'job_uuid': job_uuid,
            'event_type': event_type,
            'template_name': template_name,
            'tag': element,
            'job_uuid': job_uuid,
            'user_id': user_id,
            'ip_address': ip_address
        }
        return click_event


def gen_successful_job_application(event: dict) -> dict:
    event_time: datetime = datetime.now().strftime("%Y-%m-%d %H:%M:%S.%f")[:-3]
    job_title: str = fake.job_description()
    company_name: str = fake.company()
    application_id: str = str(uuid4())
    
    attr_job_applications = {
        'application_id': application_id,
        'event_time': event_time,
        'user_id': event['user_id'],
        'user_agent': event['user_agent'],
        'ip_address': event['ip_address'],
        'job_uuid': event['job_uuid'],
        'job_title': job_title,
        'company_name': company_name
    }
```

Now that we have our events, lets plan out the Flink Job.

First we need the following:
* Source DDL
* Sink DDL
* Process DML

For the Source DDL we need to map users for the users table, clicks, and applications for the kafka topics. You can find these [here](https://github.com/Dematom1/data_server/tree/Master/code/source)

Now we actually need to setup config for Flink and Kafka. [Link](https://github.com/Dematom1/data_server/blob/Master/code/attribute_successful_applications.py)

I saw this brillant way to store config variables using dataclasses:


```python
@dataclass(frozen=True)
class FlinkJobConfig:
    job_name: str = 'successful-job-applications'
    jars: List[str] = field(default_factory=lambda: REQUIRED_JARS)
    parallelism: int = 2


@dataclass(frozen=True)
class KafkaConfig:
    connector: str = 'kafka'
    bootstrap_servers: str = 'kafka:9092'
    scan_stratup_mode: str = 'earliest-offset'
    consumer_group_id: str = 'flink-consumer-group-1'

@dataclass(frozen=True)
class ClickEventTopicConfig(KafkaConfig):
    topic: str = 'clicks'
    format: str = 'json'

@dataclass(frozen=True)
class ApplicationTopicConfig(KafkaConfig):
    topic: str = 'applications'
    format: str = 'json'

```

This method prevents someone somehow changing any of the variables, if they do so, it will raise an error. Frozen in place.

Now that we configured Flink and Kafka, lets get to the crux of the code. The actual processing.

We need a way to dynamically process the events coming through the topics clicks and applications. From there we need to map the sources, create the sinks, and process the transformation, or in our case attribute the successful applications to our users!

The following will intialize our Stream Environment, in Flink terms, create a StreamExectutionEnvironment:
```python

def get_exec_env(config: FlinkJobConfig) -> tuple:
    s_env = StreamExecutionEnvironment.get_execution_environment()
    for jar in config.jars:
        s_env.add_jars(jar)
    
    execution_config = s_env.get_config()
    execution_config.set_parallelism(2)
    t_env = StreamTableEnvironment.create(s_env)
    return s_env, t_env
```

From there we will use the table_env, t_env here, and run the queries. 
Now I could have manually put the sql scripts in each of the execution statement, but thats a pain to maintain, plus its better to use well documented schemas, this is where our source, sinks and process DDL come in handy.

Using Jinja2 we can map each of our table and topic configs, to the Flink DDL scripts we created. After setting up the configs [Link](https://github.com/Dematom1/data_server/blob/Master/code/attribute_successful_applications.py)

Here is how we can map them:
```python

def map_sql_query(table: str, type: str = 'source', template_env: Environment = Environment(loader=FileSystemLoader('code/'))) -> str:
    config_map = {
        'clicks': ClickEventTopicConfig(),
        'applications': ApplicationTopicConfig(),
        'users' : PostgresUsersTableConfig(),
        'successful_applications' : PostgresConfig(),
        'attribute_successful_applications': PostgresSuccesfulApplicationsTableConfig()
    }

    return template_env.get_template(f'{type}/{table}.sql').render(
        asdict(config_map.get(table))
    )

```

I know what you are thinking, this is essientially how Django templating works. It basically is.

I loaded the code directory where the DDL scripts are located, and allowed us to map the intend events and/or data, to the right folders and its script.

Take the following:

```python
    table_env.execute_sql(map_sql_query('clicks'))
```

It will utilize the config at ClickEventTopicConfig(), where we defined the kafka topic we created, then it will execute code/souce/clicks.sql in the Flink table environement against the kafka topic.

After I run the subsequent applications and users sources, I then create the Sink DDL which we will insert to from the process script. This sink DDL is already configured to our table we create in postgres.

```python
table_env.execute_sql(map_sql_query('successful_applications','sink'))
```

Then we can run the statement set, this will allow us to run DML statements, in order words Flink will run the transformation as specified and insert it into the sink we defined.

Here you could execute a multi-sink job. But in my case it's just the one.

```python
process = table_env.create_statement_set()
process.add_insert_sql(map_sql_query('attribute_successful_applications', 'process'))

job  = process.execute()
```

Voila! 
Now I've process a few thousand click events and a couple hundred successful job applications. 


There is still some work to be done. From here I will schedule DAGS with Airflow runnning DBT scripts for the transformation layer, that will then connect to a GCP BigQuery instance, and from there the data can be queried into Metabase ready for you to view!

I will look to add pipeline reporting tools such as Grafana, and promethesus, for the pipeline. I will host this on a server so that you can view it. 

This is all still a work in progress!



