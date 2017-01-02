+++
tags = ["hugo","gcp"]
date = "2017-01-02T22:41:23+09:00"
title = "Deploy Hugo on Google Cloud Storage"

+++

Deploy part of the Hugo blog site created in [last post](/en/2017/01/01/hello-hugo./).


### Deploy target：Google Cloud Storage

About Hugo, I saw a lot of articles that deploy to GitHub Pages.

It's easy and cost free, but I did't want to set this blog as my GitHub user page, nor create dummy GitHub account.

Therefore, I deployed on Google Cloud Storage, in Google Cloud Platform.

No particular reason why not AWS S3.

Google Cloud Storage it self supports [Hosting a Static Website](https://cloud.google.com/storage/docs/hosting-static-website),
but it doesn't support SSL.

So, in order to achieve https access, I used alpha release function, [Using a Cloud Storage bucket as a load balancer backend](https://cloud.google.com/compute/docs/load-balancing/http/using-http-lb-with-cloud-storage).

This is my personal blog site, so not care about alpha's stability.


### Request to be whitelisted to use alpha feature

Need to request from here.

https://cloud.google.com/compute/docs/load-balancing/http/using-http-lb-with-cloud-storage


### Deploy to Google Cloud Storage

Firstly, deploy normaly by following [document](https://cloud.google.com/storage/docs/hosting-static-website).


Create Bucket

```
gsutil mb -l asia gs://yokomotod.io
```

Set default ACL

```
gsutil defacl ch -u allUsers:READER gs://yokomotod.io
```

Upload contents

```
gsutil -m rsync -R public gs://yokomotod.io
```

Set Index page and Not Found page

```
gsutil web set -m index.html -e 404.html gs://yokomotod.io
```

### Get SSL certificates

Use Let's Encrypt.

I used compute instance temporary because certbot doesn't support for mac currently.

```
sudo apt-get update
sudo apt-get install certbot -t jessie-backports
sudo certbot certonly --manual --agree-tos --manual-public-ip-logging-ok --email yokomotod@gmail.com --domain yokomotod.io
```

Then, prompt shows informaton about ACME challange.
So prapare contents according to it.

```
mkdir -p public/.well-known/acme-challenge/
echo -n m93MIV9iKfPGoZNKmDC-Una1cjD3w_FUWZYGbRsqBz0.DIv0_zl8TCm68ozhMeQWjeLCmlY0aj0PztQApIP1Lhw > public/.well-known/acme-challenge/m93MIV9iKfPGoZNKmDC-Una1cjD3w_FUWZYGbRsqBz0
gsutil -m rsync -R public gs://yokomotod.io
```

After created contents, press enter key on certbot prompt.
Certificates files will be generated if it could respond correct response for ACME challange.

Register certificates to CGP.

Go to [Networking] > [Load Balancer] > [advanced menu] > [Certificates] > [CREATE SSL CERTIFICATE], and paste contents of generated `cert.pem`、`chain.pem`、`privkey.pem` to each fields.


### Set up Load Balancer

Create backend backet (this is alpha feature)

```
gcloud alpha compute backend-buckets create yokomotod-io-backend-bucket \
    --description "" \
    --gcs-bucket-name "yokomotod.io"
```

Create url map

```
gcloud alpha compute url-maps create yokomotod-io-url-map \
    --default-backend-bucket=yokomotod-io-backend-bucket
```

Create target proxy

```
gcloud compute target-http-proxies create yokomotod-io-target-http-proxy \
    --url-map yokomotod-io-url-map
gcloud compute target-https-proxies create yokomotod-io-target-https-proxy \
    --url-map yokomotod-io-url-map --ssl-certificate yokomotod-io-ssl-certificate-2017-01-01
```

Create global ip address

```
gcloud compute addresses create yokomotod-io-address --global
```

Create global forwarding rule

```
gcloud compute forwarding-rules create yokomotod-io-http-forwarding-rule \
    --address 35.186.204.77 --global \
    --target-http-proxy yokomotod-io-target-http-proxy --ports=80

gcloud compute forwarding-rules create yokomotod-io-https-forwarding-rule \
    --address 35.186.204.77 --global \
    --target-https-proxy yokomotod-io-target-https-proxy --ports=443
```

Finally, register yokomotod.io A record 35.186.204.77 to DNS, and https://yokomotod.io is established.

### TODOs

- Update Let's Encrypt Certificates automation
  - Cron on Compute Instance can achieve it, but I don't want to have instance only for that.
  - I'm waiting Cloud Function cron job or like that. Until that, maybe GAE Cron Service can be used but not checked.
- 301 Redirect http requests to https
  - AWS Cloud Front support http→https redirect but GCP doesn't yet.
  - This is also can be done with nginx on instance, but I don't want to ...
  - Or else, I hope the feature of Cloud Function backend for Load Balancer


