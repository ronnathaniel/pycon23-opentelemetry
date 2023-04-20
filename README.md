# How to Monitor and Troubleshoot Applications in Production using OpenTelemetry
    
The #1 most comprehensive OpenTelemetry tutorial from A to Z.

    Presented @

    PyCon US 2023  
    1:30pm-5pm. Thursday, April 20 2023. 

Authored by Ron Nathaniel

<img src="https://opentelemetry.io/img/social/logo-wordmark-001.png" height="400px">

## Observability:

> A measure of how well **internal states** of a system 
> 
> can be inferred from knowledge of its **external outputs**.
> <br> 
> <br> 
> <br> 
> A person aging:
> <br> 
> <img src="https://media.istockphoto.com/id/1305631710/photo/comparison-of-young-and-elderly-time-young-and-adult.jpg?s=612x612&w=0&k=20&c=d19b9IklmST4YXOPpd0qrDE7fLnbZwmEMFybdUBYSkM=" >
> <br>
> <br>
> A star burning:
> <br>
> <img src="https://scitechdaily.com/images/Burning-Star-Sun-Space-Illustration.jpg" height="400px">
> <br>
> <br>
> <br>
> Three Pillars of Observability:
> <br>
> <img src="https://marvel-b1-cdn.bc0a.com/f00000000236551/dt-cdn.net/wp-content/uploads/2019/08/three_pillars_observability.jpg" height="300px">
> <br>
> = MeLT. (a Telemetry Data stack).

<br>
<br>

## OpenTelemetry:

> A collection of **tools, APIs, and SDKs**
> 
> used to **instrument, generate, collect, and export** telemetry data.
> <br>
> <br>
> <br>
> At a high level:
> <br>
> <br>
> <img src="collector-demo/demo-arch.png" height="300px">
> <br>
> <br>
> In depth:
> <br>
> <img src="https://grafana.com/static/assets/img/blog/spring-boot-grafana-cloud-diagram-opentelemetry-architecture.png" height="300px">



## Table of Contents

> Preparing our Workstation: 


- [Pre-Reqs](#pre-requirements)
- [Step 1 - Getting the Collector](#step-1)
- [Step 2 - Inspecting the Source Code](#step-2)
- [Step 3 - Cleanup](#step-3)
- [Step 4 - Running the Collector](#step-4)
- [Step 5 - Monitoring with our Observability Tools](#step-5)

> Base Case: 

- [Step 6 - The **BASE** Case](#step-6)
- [Step 7 - Testing the Base Case](#step-7)

> Manual Instrumentation Case:

- [Step 8 - The **MANUAL** Instrumentation Case](#step-8)
- [Step 9 - Testing The Manual Instrumentation Case](#step-9)
- [Step 10 - Monitoring The Manual Instrumentation Case](#step-10)

> Contrib Instrumentation Case:

- [Step 11 - The **CONTRIB** Instrumentation Case](#step-11)
- [Step 12 - Testing The Contrib Instrumentation Case](#step-12)
- [Step 13 - Monitoring The Contrib Instrumentation Case](#step-13)

> No-Code Instrumentation Case:

- [Step 14 - The **NO CODE** Instrumentation Case](#step-14)
- [Step 15 - Testing The No Code Instrumentation Case](#step-15)
- [Step 16 - Monitoring The No Code Instrumentation Case](#step-16)

> Conclusion

- [Step 17 - What did we learn?](#step-17) 





## Pre-Requirements


First clone this repository:

```shell
git clone https://github.com/ronnathaniel/pycon23-opentelemetry
cd pycon23-opentelemetry
```


And then make sure the following dependencies are installed:




> All can be installed via
> [HomeBrew](https://formulae.brew.sh/formula/)
> / Yum
> / Rpm
> / Pacman
> / Apt-Get
> / Any OS Package Manager

- [Python 3](https://www.python.org/downloads/)
- [Docker Desktop](https://docs.docker.com/engine/install/)



<br>

### While we wait...
Are you experienced in:

- Python?
- Flask or Web servers?
- OpenTelemetry or Observabilitity?
- 1 year?
- 5 years?
- 10 years?


## Step 1
> #### Getting the Collector

Move into the OpenTelemetry Collector demo. In a Terminal, run:

```bash
cd collector-demo
```

## Step 2
> #### Inspecting the Source Code

At `collector-demo/`, read in order:

1) `docker-compose.yaml`
        
Brings up containers with the OpenTelemetry Collector 
and 3 observability tools: Jaeger, Zipkin, and Prometheus.

> **Side Note** \
> (Main difference between Jaeger and Zipkin: \
> Zipkin runs as 1 single process, Including Collector, Storage, Querying Service, and UI.  
> Jaeger introduced the concept of splitting into different processes, one being the Collector, another being the UI, etc. 
> 
> *Prompt:* Can you guess why?)



2) `otel-collector-config.yaml`

Defines how we want to receive Telemetry (as OTLP GRPC requests) \
How we want to process Telemetry (in Batches)  
And how we want to export Telemetry (to Prometheus, Zipkin, and Jaeger)

3) `prometheus.yaml`

Defines where we want to gather Metrics Telemetry from (the Otel Collector)


## Step 3
> #### Cleanup

Skip this section..... For now .....
<br>

<img src="https://www.memecreator.org/static/images/memes/4260641.jpg">


## Step 4
> #### Running the Collector 

Lets start the collector locally. In a Terminal, run:

```bash
docker compose up
```

And voila, our collector and observability tools are up and running.

## Step 5
> #### Monitoring with our Observability Tools

View the tools at:


- Jaeger = [localhost:16686](http://0.0.0.0:16686)

- Zipkin = [localhost:9411](http://0.0.0.0:9411)

- Prometheus = [localhost:9090](http://0.0.0.0:9090)


## Step 6
> #### The Base Case (not yet instrumented)

Create a Python file (`flask_base.py`) with the following:

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
def hello():
    return 'Hello, World! from our plain Flask App\n'


@app.route('/error')
def error():
    1 / 0


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

Let's install our dependencies with 

```shell
pip3 install flask
```

And open 2 empty terminals side by side (`CMD + D`, move between with `CMD + [` and `CMD + ]`).


## Step 7
> #### Testing the Base Case

In a main terminal, run with file with 

```python
python3 flask_base.py
```

And move to the other terminal.

 -> For a successful response, call the server with: 

```python
curl localhost:5000
```

-> For an unsuccessful response, call the server with: 

```python
curl localhost:5000/error
```

And I want you to tell me (as the client) 
- what happened? 
- why did the request fail?
- did the server crash?
- is the server still up and running?
- is my application still working or do I need to restart it?

OpenTelemetry to the rescue...

## Step 8
> #### The Manual Instrumentation Case

Manual Instrumentation is the hardest but safest of all instrumentations. 
The developer is in complete control over what gets instrumented, and how.

Create a Python file by duplicating `flask_base.py` with:

```bash
cp flask_base.py flask_instrumentation_manual.py
```

and let's install our dependencies:

```bash 
pip3 install \
   opentelemetry-api \
   opentelemetry-sdk  \
   opentelemetry-exporter-otlp
```

And in our `flask_instrumentation_manual.py` file add some boilerplate at the top:

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import (
    BatchSpanProcessor, ConsoleSpanExporter
)
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.resources import SERVICE_NAME, Resource


COLLECTOR_ENDPOINT = 'http://0.0.0.0:4317'
MY_SERVICE = 'MyFlaskAppInProductionManual'

resource = Resource(attributes={
    SERVICE_NAME: MY_SERVICE
})
provider = TracerProvider(resource=resource)

# exports to STDOUT
processor_console = BatchSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(processor_console)
# exports to Otel Collector
processor_grpc = BatchSpanProcessor(OTLPSpanExporter(endpoint=COLLECTOR_ENDPOINT))
provider.add_span_processor(processor_grpc)

trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

and replace our `hello()` function at `/` route with 

```python
@app.route('/')
def hello():
    with tracer.start_as_current_span('root') as root:
        root.add_event('root_event')
        return 'Hello, World! from our Manually Instrumented Flask App\n'
```



and our `error()` function at `/error` with

```python
@app.route('/error')
def error():
    with tracer.start_as_current_span('root') as root:
        root.add_event('root_event')
        1 / 0
```

## Step 9
> #### Testing The Manual Instrumentation Case

In the first terminal, run with file with 

```python
python3 flask_instrumentation_manual.py
```

And move to your second terminal.

 -> For a successful response, call the server with: 

```python
curl localhost:5000
```

-> For an unsuccessful response, call the server with: 

```python
curl localhost:5000/error
```



## Step 10
> #### Monitoring The Manual Instrumentation Case

View the tools at:


- Jaeger = [localhost:16686](http://0.0.0.0:16686)

- Zipkin = [localhost:9411](http://0.0.0.0:9411)

- Prometheus = [localhost:9090](http://0.0.0.0:9090)






## Step 11
> #### The Contrib Instrumentation Case

Contributed packages include support for third party tools. 

Restart our instrumentation with 

```python
cp flask_base.py flask_instrumentation_contrib.py
```

and Install the Flask contrib package with:

```bash
pip3 install opentelemetry-instrumentation-flask
```


Only 2 sections are needed to instrument. The top-level import

```python
import os
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import (
    BatchSpanProcessor, ConsoleSpanExporter
)
from opentelemetry.sdk.resources import SERVICE_NAME, Resource
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor

os.environ['OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_REQUEST'] = '.*'
os.environ['OTEL_INSTRUMENTATION_HTTP_CAPTURE_HEADERS_SERVER_RESPONSE'] = '.*'

COLLECTOR_ENDPOINT = 'http://0.0.0.0:4317'
MY_SERVICE = 'MyFlaskAppInProductionContrib'

resource = Resource(attributes={
    SERVICE_NAME: MY_SERVICE
})
provider = TracerProvider(resource=resource)

# exports to STDOUT
processor_console = BatchSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(processor_console)
# exports to Otel Collector
processor_grpc = BatchSpanProcessor(OTLPSpanExporter(endpoint=COLLECTOR_ENDPOINT))
provider.add_span_processor(processor_grpc)
```

and 

```python
FlaskInstrumentor().instrument_app(app, tracer_provider=provider)
```

given `app` is your flask app object.


## Step 12
> #### Testing The Contrib Instrumentation Case



In the first terminal, run with file with 

```python
python3 flask_instrumentation_contrib.py
```

And move to your second terminal.

 -> For a successful response, call the server with: 

```python
curl localhost:5000
```

-> For an unsuccessful response, call the server with: 

```python
curl localhost:5000/error
```

## Step 13
> #### Monitoring The Contrib Instrumentation Case

View the tools at:


- Jaeger = [localhost:16686](http://0.0.0.0:16686)

- Zipkin = [localhost:9411](http://0.0.0.0:9411)

- Prometheus = [localhost:9090](http://0.0.0.0:9090)


## Step 14
> #### The No Code Instrumentation Case 



Our only dependency to install is:

```python
pip3 install opentelemetry-distro
```

and we must bootstrap it with:

```python
opentelemetry-bootstrap -a install 
```

## Step 15
> #### Testing The No Code Instrumentation Case

Our method of running our application must change to

```bash
opentelemetry-instrument --traces_exporter otlp,console --service_name MyFlaskAppInProductionAuto --exporter_otlp_endpoint http://localhost:4317 python3 flask_base.py
```

Now lets break that command ^ down.

- `opentelemetry-instrument` = a command line tool which auto-instruments Python files.

- `--traces_exporter otlp,console` = flag to declare we will export our spans/traces via OTLP and our local Console (stdout).

- `--service_name MyFlaskAppInProductionAuto` = flag to declare our OpenTelemetry's Service name.

- `--exporter_otlp_endpoint http://localhost:4317` = a flag to declare exactly *where* we will send our OTLP spans.

- `python3 flask_base.py` = we tell `opentelemetry-instrument` how we usually run our application. 
This allows the CLI tool to run it for us.


Paste the command above into a file named `flask_instrumentation_nocode.sh`.

Be sure to `chmod +x flask_instrumentation_nocode.sh` to make it executable!


In the first terminal, run our new file with

    ./flask_instrumentation_nocode.sh

And move to your second terminal.

 -> For a successful response, call the server with: 

```python
curl localhost:5000
```

-> For an unsuccessful response, call the server with: 

```python
curl localhost:5000/error
```

## Step 16
> #### Monitoring The No Code Instrumentation Case


View the tools at:


- Jaeger = [localhost:16686](http://0.0.0.0:16686)

- Zipkin = [localhost:9411](http://0.0.0.0:9411)

- Prometheus = [localhost:9090](http://0.0.0.0:9090)


## Step 17
> #### What did we learn?


Can you answer these questions:


-     What is Observability?
-     What is OpenTelemetry?
-     What App Observability tools exist? 
-     With what methods can I instrument my app with OpenTelemetry?
-     When should I use each?
-     What is the OpenTelemetry Collector?

And you're free to go. Have a good day!

Thanks for participating.
