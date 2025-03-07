# influxdb-odata-server
Odata provider (server) interfaces influxdb by exposing using 
REST based odata endpoints. Supports Influxdb 1.x and 2.x Connects InfluxDB with BI apps 
including Excel, PowerBI and Tableau.

## Installation and Setup
Current version has been tested on python 3.6. Few features might 
break in 2.x. 

Install all dependencies via pip

`pip install -r requirements.txt`

Configure Odata server host and port details in 
`settings.conf` Sample config file is well documented. Choose the 
`influxdb_version` in metadata section and accordingly provide
influxdb connection details. Also, enable authentication if required.
Refer to [Authentication](#Authentication) section 

Before spinning up the server, create an xml metadata defining the 
fields/tags of each measurement in an influx database (or bucket) using 

`python startServer.py -c production.conf -m `

`-m` or `--createmetadata` flag creates Odata schema metadata file
at the location defined by the `metadata_file` tag in the 
configuration. It doesn't spin up the server. Schema is generated 
for all measurements specified by the `databases` or `bucket` tags 
in the configuration. Multiple db/buckets can be configured by 
separating each by comma(,)

Once the metadata file is generated, spin up the Odata server by 
`python startServer.py -c settings.conf`

### Optional Systemd Service

You may wish to install the influx-odata.service file and run this on
system start. Note that the file currently is configured for use with
NGINX, python3.8, and Centos8, so modify accordingly.

If you use SELinux, you will need to enable the `httpd_can_network_connect`
and `nis_enabled` booleans. You may also need to adjust the labels and
generate some policy modules. In my case, I needed policies for
`transition`, `search`, `nnp_transition`, and `entrypoint` before Python
was able to start under Systemd. A similar hurdle likely exists for
AppArmor.


## Supported features
* Odata 2 provider (server) implemetation.
* Supports both Influxdb 1.x and Influx 2.0 with Flux support. 
Based on the request, it internally converts into 
influxql/flux query (based on influx version in use).
* Works with any BI tool supporting Odata 2 as data source. 
Tested for Excel 2016, 2019, Office 365, Tableau Desktop 2019.
* Supports only one database/bucket per service. Multiple 
instances on separate ports can be initiated to handle multiple 
databas/bucket.

## Supported Influx Functions and Usage
* Database/Bucket and Measuremnt
* Select
* Filter (time based and tag-value based)
* Aggregate Function
* Group by time 
* Limit

For example, the endpoint 

`http://localhost:8080/data/monitoring__cpu?$filter=timestamp ge datetime'2020-04-03T00:00:00' and timestamp lt datetime'2020-04-06T00:00:00' and host eq '192.168.1.12'&$top=100&groupByTime=1h&aggregate=mean&$select=usage_guest,usage_guest_nice,usage_idle,usage_iowait, usage_steal,usage_system,usage_user`

queries measurement `cpu` in`monitoring` database for `timestamp` 
greater equal to `2020-04-03T00:00:00` and less than `2020-04-06T00:00:00`
and tag `host` equal to string value `192.168.1.12`, group by 
every `1 hour` and returns aggregated `mean` on fields `usage_guest,
usage_guest_nice,usage_idle,usage_iowait, usage_steal,usage_system,
usage_user`
  

## Authentication
Current version supports only basic HTTP authentication via Simple
Login and AWS Cognito. Simple Login method stores username and password
in configuration file (in plain text). 

Authentication validator module is called dynamically based on the 
configuration. Additional modules can be added in the `authentication` 
directory of the project. 

## Deployment
* `startServer.py` spins up a werkzeug server using `run_simple` module.
 This is not suitable for production and doesn't perform well on 
 concurrent requests.
 
 * The code can also be hosted AWS Lambda and served with  AWS API 
 Gateway for serveless implementation. Code has tested with Zappa
 framework. Detailed instructions to be released soon. 
 
 * Docker image to be released soon.

