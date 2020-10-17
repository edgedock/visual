## IBM Edge Application Manager (IEAM)
### Edge deployment of container workload and ML model

TensorFlow Lite and OpenVINO based Machine Inferencing and Object Detection

IEAM + MMS + OpenCV + Tensorflow Lite + OpenVINO + Python 

### Introduction

A python based implementation using three containers with detection time below < 0.4 sec without any GPU accelerator on Intel NUC using coco_ssd_mobilenet_v1_1.0 model

- Image capture and classification
- Http and kafka message bridge
- MJPEG based streaming available on http:<edge-device-ip-address:5000> 
- A simple  Web UI to interactivly upload config using MMS to edge nodes.
- Frameworks - TensorFlow Lite and OpenVINO
- Intel Neural Compute Stick 2 + Movidius MyriadX VPU

#### Build, Publish and Use in a shared IEAM environment

Using a shared single tenant instance creates several challenges when services, patterns and policies need to be published in a common exchange under one org structure without multiple developers clobbering over each other.

#### Issues

- Developers need to identify their service, pattern and policies among many similar assets (concept of owner)
- Developers may have project for demo, dev, test and more ( group the assets)
- Developers need to use their own docker repository (specify own docker account)

The tooling outlined below addresses these concerns and builds on top of existing infrastructure

### Automated Steps

Start with reviewing Makefile for targets.

Build and publish images to docker and services to exchange. Can be executed again and again.

    make

### ENVIRONMENT variables

Must define following ENVIRONMENT variables to build application, add policies and register edge node. Preferebly save in the script directory and use when invoking device registration. 

Enviornment variables EDGE_OWNER, EDGE_DEPLOY provide flexiblity for different developers to use the same exchange without clobering over each other.

    export EDGE_OWNER=<a-two-or-three-letter-distinctive-initial-of-your-name>  # sg.dev  
    export EDGE_DEPLOY=<deploy-target> # e.g: example.tfvisual

    export DOCKER_BASE=<docker-base> # e.g. Change this this your docker base 

#### IEAM specific

    export HZN_ORG_ID=mycluster
    export HZN_EXCHANGE_USER_AUTH=iamapikey:<iam-api-key>
    export HZN_EXCHANGE_NODE_AUTH="<UNIQUE-NODE-ANME>:<node-token>"
    export HZN_DEVICE_ID=`grep HZN_DEVICE_ID /etc/default/horizon | cut -d'=' -f2`

#### Application specific
    export APP_NODE_NAME=<UNIQUE-NODE-ANME>
    export DEVICE_ID=<unique-device-id>
    export DEVICE_NAME="<short device name"
    export DEVICE_IP_ADDRESS=`hostname -i`
    
    export APP_IEAM_API_CSS_OBJECTS=https://`grep -viE '^$|^#' /etc/default/horizon |  grep HZN_EXCHANGE_URL | cut -d'=' -f2 | cut -d'/' -f3`/edge-css/api/v1/objects
    export APP_APIKEY_PASSWORD=<api-key-provided-to-you>
    export APP_MMS_OBJECT_ID_CONFIG="config.json"
    export APP_MMS_OBJECT_TYPE_CONFIG="tflite-mmsconfig"
    export APP_MMS_OBJECT_SERVICE_NAME_CONFIG="$EDGE_OWNER.$EDGE_DEPLOY.mms"
    
    export APP_BIND_HORIZON_DIR=/var/tmp/horizon
    
    export APP_MMS_OBJECT_ID_MODEL="<object-model>"
    export APP_MMS_OBJECT_TYPE_MODEL="tflite-mmsmodel"
    export APP_MMS_OBJECT_SERVICE_NAME_MODEL="$EDGE_OWNER.$EDGE_DEPLOY.mms"
    
    # face detection disabled for now as haarscascade based detection is not reliable
    export SHOW_OVERLAY=true # false to hide OVERLAY
    export DETECT_FACE=false
    export BLUR_FACE=false
    export PUBLISH_KAFKA=false # To send kafka stream
    export PUBLISH_STREAM=true # to send local mjpeg stream and view in browser

#### RTSP Streams

    export RTSP_STREAMS=rtsp://<ip-address>:<port>/rtsp,rtsp://<ip-address>:<port>/rtsp

#### Event Streams

    export EVENTSTREAMS_BASIC_TOPIC=<your-event-stream-topic>
    export EVENTSTREAMS_ENHANCED_TOPIC=<your-event-stream-topic>
    export EVENTSTREAMS_API_KEY=<your-event-stream-api-key>
    export EVENTSTREAMS_BROKER_URLS="your-event-stream-brokers"

### Create node

    hzn exchange node create -n $HZN_EXCHANGE_NODE_AUTH
    
### Publish Machine Inference model (object id format is my own nomenclature to pass required info) 

    hzn mms object publish -t tflite-mmsmodel -i <net>-<framework>-<x.y.z>-mms -f <model-file-with-labelmap>

### Register node using framework TensorFlow lite NUC (amd64), RPI (arm32)
    
    hzn register --policy=node_policy_tflite.json --input-file user_input_app_tflite.json
    hzn register --policy=node_policy_vino.json --input-file user_input_app_vino.json

### Register node  
If using script to register node

    ./node_register_app.sh -e ENV_TFLITE -k tflite -r -l
    
    ./node_register_app.sh -e ENV_VINO -k vino -r -l

### View result 

- View streaming output in a browser 
    http://<local-ip-address>:5000/stream
    
- Take spanpshot of annotated frame
    http://<local-ip-address>:5000/test
    
- Get a file output locally (to test on the local machine)
    
        wget http://<local-ip-address>:5000/wget
    
        cat wget | base64 -d > wget.jpg

### Research, reference and acknowledgement

  https://opencv.org
  
  https://www.tensorflow.org/lite
  
  https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi
  
  https://stackoverflow.com/questions/tagged/tensorflow
  
  OpenVINO
    
    

    
