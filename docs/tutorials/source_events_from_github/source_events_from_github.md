# Setup

## Deploy Gloo
        k apply -f https://raw.githubusercontent.com/solo-io/gloo-install/master/kube/install.yaml

## Deploy NATS and minio
        k apply -f kube-deploy.yaml

## Create a route for nats
        glooctl route create --sort \
            --path-exact /github-webhooks \
            --upstream default-nats-streaming-4222 \
            --function github-webhooks

## Create a route for mirroring with minio
        glooctl route create --sort \
            --path-prefix=/ \
            --header "User-Agent:Minio (linux; amd64) minio-go/5.0.1 mc/2018-03-25T01:22:22Z" \
            --upstream default-minio-service-9000

# Image Pusher Microservice

## Install minio client
        wget https://dl.minio.io/client/mc/release/linux-amd64/mc
        chmod +x mc
        ./mc config host add gloo http://ilackarms.aws.solo.io \
            gloo.solo.io \
            gloo.solo.io \
            --api s3v4 --lookup dns

## deploy the image-pusher service
        k apply -f  deploy-image-pusher.yaml

## start mirroring the "images" minio bucket
        mkdir images
        ./mc mirror gloo/images ./images -w # browse to ./images in your file browser 

**unstar/star the git repo to see images start to appear**


# Star-Tweeter Microservice