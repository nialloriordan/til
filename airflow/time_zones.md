## Change DAG Timezones

This post is inspired by daylight savings which impacted by airflow orchestrator. Since Airflow uses UTC by default when daylight savings occured I noticed that my DAG was no longer running at the correct time but rather 1 hour later!

If you have the same issue here some info on how to make your DAGs time zone aware.

#### Time Zones

By default Airflow time zone is UTC and this can be changed in `airflow.cfg` or as an environment variable. The variable to change is called `default_timezone` and is within the `core` section. The impact of time zone means it will affect your scheduler and when you expect runs to start.

If like me you dont want to change the global settings for Airflow and just make one of your DAGs time zone aware you can find more info below.

#### Time zone aware DAGs

To create a time zone aware DAG we need to supply a time zone aware start date using pendulum.

Example:

```python
import pendulum

tz = "Europe/London"
local_tz = pendulum.timezone("Europe/Amsterdam")
default_args=dict(
    start_date=datetime(2021, 3, 28, tzinfo=local_tz),
    owner='airflow'
)
dag = DAG('my_tz_dag', default_args=default_args) # time zone aware dag
op = DummyOperator(task_id='dummy', dag=dag) # dummy op
print(dag.timezone) # <Timezone [Europe/London]>
```

Note that the DAG timezone takes precedence over the global timezone.

The only issue with the above is that Airflow does not automatically convert your macros from UTC to local time.

#### Time zone aware macros and templates

In order to convert important macros such as `execution_date` and `prev_execution_date` we need to make use of our DAGs timezone.

Example:

```python

# After converting your dag to be time zone aware
# convert your macros.
previous_execution_date_time = "{{ dag.timezone.convert(prev_execution_date_success) }}"
current_execution_date = "{{ dag.timezone.convert(execution_date) }}"

# If you also need to convert macros such as `ts_nodash` to
# local time zone this can be done by using `execution_date`
# and `strftime`

execution_date_as_datetime = (
        '{{ dag.timezone.convert(execution_date).strftime("%Y%m%dT%H%M%S") }}'
    )

```