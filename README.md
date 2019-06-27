# gauge-docker-issue2
## Context
- I'm trying to incorporate a docker container with gauge installed into Google Cloud Build as a build step.
- In the attempts so far, I found that when trying to run gauge (with python) on Docker without setting the `runner_connection_timeout` to about 240 seconds or more, gauge fails with the error message, 'Failed to start gauge API: Timed out connecting to 127.0.0.1:~'.
- So I wrote the Dockerfile so that `docker build` copies 'gauge.properties' in which `runner_connection_timeout` is set to enough duration, 600 sec, into '/root/.gauge/config/' in container image.

## Expected behavior
Even when the container is incorporated in Cloud Build, the `runner_connection_timeout` value is recognized as 600 seconds.

## Actual Behavior
When executed as a build step from Cloud Build, the `runner_connection_timeout value` is recognized as the default of 30 seconds. So if you write specs/tests and try to execute it from Cloud Build, then get the error message "Failed to start gauge API: Timed out connecting to ~".

## How to replicate the issue
This is a minimal project to reproduce and observe the issue.

You can reproduce it according to the following procedure, but please replace the docker registry name 'gcr.io/gauge-gcb-trial/' in the commands and cloudbuild.yaml with an appropriate one you have access.

```
~$ cd ~/tmp
tmp $ git clone https://github.com/yasuabe/gauge-docker-issue2
tmp $ cd gauge-docker-issue2
gauge-docker-issue2 $ cd docker
docker $ docker build -t gcr.io/gauge-gcb-trial/gauge-docker-issue2:latest .
docker $ docker run -it --entrypoint '' gcr.io/gauge-gcb-trial/gauge-docker-issue2:latest /bin/ash
/workspace # gauge version
Gauge version: 1.0.5
Commit Hash: 562f036

Plugins
-------
html-report (4.0.8)
python (0.3.5)
screenshot (0.0.1)
/workspace # exit
docker $ docker push gcr.io/gauge-gcb-trial/gauge-docker-issue2
docker $ cd ..
gauge-docker-issue2 $ docker run gcr.io/gauge-gcb-trial/gauge-docker-issue2:latest
Key                           	Value
-------------------------------------------------------------------
check_updates                 	false
gauge_telemetry_enabled       	false
gauge_telemetry_action_recorded	false
gauge_repository_url          	https://downloads.gauge.org/plugin
plugin_connection_timeout     	10000
plugin_kill_timeout           	4000
ide_request_timeout           	30000
gauge_telemetry_log_enabled   	false
gauge_templates_url           	https://templates.gauge.org
runner_connection_timeout     	600000 ★★★ This is the value I intended
runner_request_timeout        	30000

Telemetry
---------

This installation of Gauge collects usage data in order to help us improve your experience.
The data is anonymous and doesn't include command-line arguments.
To turn this message off opt in or out by running 'gauge telemetry on' or 'gauge telemetry off'.

Read more about Gauge telemetry at https://gauge.org/telemetry

gauge-docker-issue2 $ gcloud builds submit --config cloudbuild.yaml .
Creating temporary tarball archive of 4 file(s) totalling 1.7 KiB before compression.
Uploading tarball of [.] to [gs://gauge-gcb-trial_cloudbuild/source/1561610257.16-e5ffadf140b544f3b7b86e7baa359229.tgz]
Created [https://cloudbuild.googleapis.com/v1/projects/gauge-gcb-trial/builds/33ab2549-4ac9-4d95-9d67-51e6ca6b5ab3].
Logs are available at [https://console.cloud.google.com/gcr/builds/33ab2549-4ac9-4d95-9d67-51e6ca6b5ab3?project=173910909337].
---------------------------------------------------------------------- REMOTE BUILD OUTPUT ----------------------------------------------------------------------
starting build "33ab2549-4ac9-4d95-9d67-51e6ca6b5ab3"

FETCHSOURCE
Fetching storage object: gs://gauge-gcb-trial_cloudbuild/source/1561610257.16-e5ffadf140b544f3b7b86e7baa359229.tgz#1561610289871946
Copying gs://gauge-gcb-trial_cloudbuild/source/1561610257.16-e5ffadf140b544f3b7b86e7baa359229.tgz#1561610289871946...
/ [1 files][  1.0 KiB/  1.0 KiB]
Operation completed over 1 objects/1.0 KiB.
BUILD
Pulling image: gcr.io/gauge-gcb-trial/gauge-docker-issue2:latest
latest: Pulling from gauge-gcb-trial/gauge-docker-issue2
e7c96db7181b: Already exists
799a5534f213: Already exists
913b50bbe755: Already exists
11154abc6081: Already exists
c805e63f69fe: Already exists
850211d935f2: Pulling fs layer
d19848b9fc50: Pulling fs layer
cd3a457b2014: Pulling fs layer
d19848b9fc50: Verifying Checksum
d19848b9fc50: Download complete
cd3a457b2014: Verifying Checksum
cd3a457b2014: Download complete
850211d935f2: Verifying Checksum
850211d935f2: Download complete
850211d935f2: Pull complete
d19848b9fc50: Pull complete
cd3a457b2014: Pull complete
Digest: sha256:bd1aafe1782a8b4aaa88094a0fe3ecbfdb4d1d5a00fade097c4901aec2faee94
Status: Downloaded newer image for gcr.io/gauge-gcb-trial/gauge-docker-issue2:latest
Key                           	Value
-------------------------------------------------------------------
gauge_telemetry_enabled       	true
gauge_repository_url          	https://downloads.gauge.org/plugin
gauge_templates_url           	https://templates.gauge.org
runner_request_timeout        	30000
ide_request_timeout           	30000
check_updates                 	true
gauge_telemetry_log_enabled   	false
gauge_telemetry_action_recorded	false
runner_connection_timeout     	30000 ★★★ This is the default value, not I intended
plugin_connection_timeout     	10000
plugin_kill_timeout           	4000

Telemetry
---------

This installation of Gauge collects usage data in order to help us improve your experience.
The data is anonymous and doesn't include command-line arguments.
To turn this message off opt in or out by running 'gauge telemetry on' or 'gauge telemetry off'.

Read more about Gauge telemetry at https://gauge.org/telemetry

PUSH
DONE
-----------------------------------------------------------------------------------------------------------------------------------------------------------------

ID                                    CREATE_TIME                DURATION  SOURCE                                                                                     IMAGES  STATUS
33ab2549-4ac9-4d95-9d67-51e6ca6b5ab3  2019-06-27T04:38:11+00:00  17S       gs://gauge-gcb-trial_cloudbuild/source/1561610257.16-e5ffadf140b544f3b7b86e7baa359229.tgz  -       SUCCESS
```
