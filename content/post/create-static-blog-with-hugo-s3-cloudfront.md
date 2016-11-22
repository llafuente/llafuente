+++
draft = false
images = []
description = ""
categories = [
  "webdev",
]
date = "2016-11-21T16:31:35+01:00"
title = "Create static blog with hugo, s3 & cloudfront"
tags = [
  "aws",
  "hugo",
  "s3",
  "cloudfront",
  "webdev",
]

+++

S3 have website hosting support for static sites long time, but never
have time to check. Lately I spend some time again with AWS and decide to
create this blog and share everything in a more readable way
(rather than the github-readme way).

I decide to create the blog using [hugo](http://www.gohugo.io) because I
already know [Jekyll](https://jekyllrb.com/). Jekyll is in ruby... and I
prefer not to code anything in ruby... anymore.

The first step is to have a [hugo](http://www.gohugo.io) blog ready
Follow their instructions it's mostly a searching and reading process, no magic.

Here are the initial steps for completeness.

{{< highlight bash >}}
# create the hugo site in CWD
hugo new site .
# choose a theme
mkdir themes/
# modify the url with the one you want
wget -O theme.zip \
  'https://github.com/alanorth/hugo-theme-bootstrap4-blog/archive/master.zip'
cd themes/
unzip ../theme.zip
cd ..
# create a post
hugo new post/xxx.md
# listen to changes, this will create a server see the output for the url
hugo server -w
{{< /highlight >}}

Now you have a hugo blog with one post and a theme.
Time to read themes/xxx/readme.md and configure your blog-theme editing
`./config.toml`.

After having a nice and readable first version I pushed to github at
[llafuente/llafuente](https://github.com/llafuente/llafuente).

And now it's time to do the real work, create and deploy.

This process will be manual in my case. Having codeship/travis/* to deploy
the web should be easy, but I don't want to handle branches in this small repo.
When I finish writting something I could `git push && s3 sync`, no big deal.

*Cloudfront warning*: Took me quite some time to realize cloudfront
changes are not realtime-ish, in fact there are 10-30 minutes delay.

I'm a big fan of create and destroy `AWS` services. So I will share both.

Steps:

* ./create-static-website.sh
* follow the links displayed to check everything is ok
* configure your domain CNAME
* enjoy
* ./create-static-website.sh to update the website


### <i class="fa fa-file-o" aria-hidden="true"></i> ./create-static-website.sh

**NOTE**: this script call ./sync-static-website.sh, both should be in the
same path.

Usage:

Create s3 bucket, configure website, www redirection

{{< quote >}}
./create-static-website.sh \\\
--domain=llafuente.com \\\
--source=/var/home/llafuente/blog/public
{{< / quote >}}

Create s3 bucket, configure website, www redirection and create a
cloudfront distribution.

{{< quote >}}
./create-static-website.sh \\\
--domain=llafuente.com \\\
--source=/var/home/llafuente/blog/public \\\
--cloudfront
{{< / quote >}}

Code contains code and configuration you need to read...
Don't just copy&paste!

{{< highlight bash >}}
#!/bin/sh

set -ex

PROTOCOL="http"
CLOUDFRONT=""

for i in "$@"
do
case $i in
  --domain=*)
    DOMAIN="${i#*=}"; shift;
  ;;
  --source=*)
    SOURCE="${i#*=}"; shift;
  ;;
  --protocol=*)
    PROTOCOL="${i#*=}"; shift;
  ;;
  --cloudfront)
    CLOUDFRONT="yes";
  ;;
  *)
    # unknown option
  ;;
esac
done


if [ -z ${SOURCE} ]; then
  echo "--source is required"
  echo "KO"
  exit 1
fi

if [ -z ${DOMAIN} ]; then
  echo "--domain is required"
  echo "KO"
  exit 1
fi

# full list: "AllowedMethods": ["GET", "PUT", "POST", "DELETE"]
tee /tmp/cors.json <<EOF >/dev/null
{
  "CORSRules": [
    {
      "AllowedOrigins": ["*"],
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET"]
    }
  ]
}
EOF

# if your are going to use cloudfront, maybe consider using:
#"Condition": {
#  "StringEquals": {
#    "aws:UserAgent": "Amazon CloudFront"
#  }
#}

tee /tmp/policy.json <<EOF >/dev/null
{
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": "*",
         "Action": [
           "s3:GetObject"
          ],
         "Resource": "arn:aws:s3:::${DOMAIN}/*"
      },
      {
         "Effect": "Allow",
         "Principal": "*",
         "Action": [
           "s3:ListBucket"
          ],
         "Resource": "arn:aws:s3:::${DOMAIN}"
      }
   ]
}
EOF

tee /tmp/www.policy.json <<EOF >/dev/null
{
   "Statement": [
      {
         "Effect": "Allow",
         "Principal": "*",
         "Action": [
           "s3:GetObject"
          ],
         "Resource": "arn:aws:s3:::www.${DOMAIN}/*"
      },
      {
         "Effect": "Allow",
         "Principal": "*",
         "Action": [
           "s3:ListBucket"
          ],
         "Resource": "arn:aws:s3:::www.${DOMAIN}"
      }
   ]
}
EOF

tee /tmp/website.json <<EOF >/dev/null
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "404.html"
  },
  "RoutingRules": [
    {
      "Condition": {
        "HttpErrorCodeReturnedEquals": "404"
      },
      "Redirect": {
        "HostName": "${DOMAIN}",
        "ReplaceKeyPrefixWith": "#/"
      }
    }
  ]
}
EOF

# create bucket, cors, policy & website
aws s3 mb "s3://${DOMAIN}"
aws s3api put-bucket-cors --bucket ${DOMAIN} \
  --cors-configuration file:///tmp/cors.json
aws s3api put-bucket-policy --bucket ${DOMAIN} \
  --policy file:///tmp/policy.json
aws s3api put-bucket-website --bucket ${DOMAIN} \
  --website-configuration file:///tmp/website.json

./sync-static-website.sh --source=${SOURCE} --domain=${DOMAIN}

# create a dummy website for www.${DOMAIN}, redirect everything to non-www
aws s3 mb "s3://www.${DOMAIN}"
aws s3api put-bucket-policy --bucket "www.${DOMAIN}" \
  --policy file:///tmp/www.policy.json
aws s3api put-bucket-cors --bucket "www.${DOMAIN}" \
  --cors-configuration file:///tmp/cors.json

tee /tmp/www.website.json <<EOF >/dev/null
{
  "RedirectAllRequestsTo" : {
    "HostName" : "${DOMAIN}",
    "Protocol" : "${PROTOCOL}"
  }
}
EOF

aws s3api put-bucket-website --bucket "www.${DOMAIN}" \
  --website-configuration file:///tmp/www.website.json

REGION=$(aws s3api get-bucket-location --bucket ${DOMAIN} \
  --query 'LocationConstraint' --output text)

echo "s3 domain: http://${DOMAIN}.s3-website.${REGION}.amazonaws.com"

# couldfront

if [ ! -z ${CLOUDFRONT} ]; then

  tee /tmp/couldfront.json <<EOF >/dev/null
{
    "Comment": "",
    "CacheBehaviors": {
        "Quantity": 0
    },
    "Logging": {
        "Bucket": "",
        "Prefix": "",
        "Enabled": false,
        "IncludeCookies": false
    },
    "WebACLId": "",
    "Origins": {
        "Items": [
            {
                "S3OriginConfig": {
                    "OriginAccessIdentity": ""
                },
                "OriginPath": "",
                "CustomHeaders": {
                    "Quantity": 0
                },
                "Id": "S3-${DOMAIN}",
                "DomainName": "${DOMAIN}.s3.amazonaws.com"
            }
        ],
        "Quantity": 1
    },
    "DefaultRootObject": "/index.html",
    "PriceClass": "PriceClass_100",
    "Enabled": true,
    "DefaultCacheBehavior": {
        "TrustedSigners": {
            "Enabled": false,
            "Quantity": 0
        },
        "TargetOriginId": "S3-${DOMAIN}",
        "ViewerProtocolPolicy": "allow-all",
        "ForwardedValues": {
            "Headers": {
                "Quantity": 0
            },
            "Cookies": {
                "Forward": "none"
            },
            "QueryStringCacheKeys": {
                "Quantity": 0
            },
            "QueryString": false
        },
        "MaxTTL": 31536000,
        "SmoothStreaming": false,
        "DefaultTTL": 86400,
        "AllowedMethods": {
            "Items": [
                "HEAD",
                "GET"
            ],
            "CachedMethods": {
                "Items": [
                    "HEAD",
                    "GET"
                ],
                "Quantity": 2
            },
            "Quantity": 2
        },
        "MinTTL": 0,
        "Compress": false
    },
    "CallerReference": "1479735850029",
    "ViewerCertificate": {
        "CloudFrontDefaultCertificate": true,
        "MinimumProtocolVersion": "SSLv3",
        "CertificateSource": "cloudfront"
    },
    "CustomErrorResponses": {
        "Quantity": 0
    },
    "HttpVersion": "http2",
    "Restrictions": {
        "GeoRestriction": {
            "RestrictionType": "none",
            "Quantity": 0
        }
    },
    "Aliases": {
        "Quantity": 0
    }
}
EOF

  DISTRIBUTION_ID=$(aws cloudfront create-distribution \
    --origin-domain-name "${DOMAIN}.s3.amazonaws.com" \
    --default-root-object index.html
    --query 'Id' --text)

  CLOUDFRONT_DOMAIN=$(aws cloudfront get-distribution \
    --id ${DISTRIBUTION_ID} --query 'Distribution.DomainName')

  echo "cloudfront domain: ${CLOUDFRONT_DOMAIN}"
  echo "NOTE! Now you have to wait a lot for cloudfront to be ready!"

fi

{{< /highlight >}}


### <i class="fa fa-file-o" aria-hidden="true"></i> ./sync-static-website.sh


Usage:

Sync source path (gzipped)

{{< quote >}}
./sync-static-website.sh \\\
  --domain=llafuente.com
  --source=/var/home/llafuente/blog/public
{{< / quote >}}


{{< highlight bash >}}
#!/bin/sh

set -x

for i in "$@"
do
case $i in
  --domain=*)
    DOMAIN="${i#*=}"; shift;
  ;;
  --source=*)
    SOURCE="${i#*=}"; shift;
  ;;
  *)
    # unknown option
  ;;
esac
done


if [ -z ${SOURCE} ]; then
  echo "--source is required"
  echo "KO"
  exit 1
fi

if [ -z ${DOMAIN} ]; then
  echo "--domain is required"
  echo "KO"
  exit 1
fi

#public/index.html: HTML document, UTF-8 Unicode text
#public/index.html: gzip compressed data, was "index.html", \
#from Unix, last modified: Tue Nov 22 16:43:35 2016, max compression
IS_COMPRESSED=$(file "${SOURCE}/index.html" | grep 'gzip')

# do not re-compress
if [ -z ${IS_COMPRESSED} ]; then
  echo "compressing source path"

  find ${SOURCE} | xargs gzip -9
  find ${SOURCE} -type f -name '*.gz' | \
    while read f; do mv "$f" "${f%.gz}"; done
fi

# sync files
# html: no cache
aws s3 sync \
  --content-encoding gzip \
  --exclude "*" --include "*.html" \
  "${SOURCE}" "s3://${DOMAIN}"
# rest of files ensure cache!
aws s3 sync \
  --content-encoding gzip \
  --cache-control "max-age=2592000" --acl "public-read" --sse "AES256" \
  --exclude "*.html" \
  "${SOURCE}" "s3://${DOMAIN}"

{{< / highlight >}}

### <i class="fa fa-file-o" aria-hidden="true"></i> ./delete-static-website.sh

Usage:

delete s3 buckets
{{< quote >}}
./delete-static-website.sh \\\
  --domain=llafuente.com
{{< / quote >}}

delete s3 buckets and cloudfront
{{< quote >}}
./delete-static-website.sh \\\
  --domain=llafuente.com \\\
  --id=???
{{< / quote >}}

{{< highlight bash >}}
#!/bin/sh

set -ex

for i in "$@"
do
case $i in
  --id=*)
    DISTRIBUTION_ID="${i#*=}"
    shift # past argument=value
  ;;
  --domain=*)
    DOMAIN="${i#*=}"
    shift # past argument=value
  ;;
  *)
    # unknown option
  ;;
esac
done

if [ -z ${DOMAIN} ]; then
  echo "--domain is required"
  echo "KO"
  exit 1
fi

# dev: select first, if you have a clean aws account this is useful :)
# DISTRIBUTION_ID=$(aws cloudfront list-distributions \
#   --query 'DistributionList.Items[0].Id' --output text)

if [ ! -z ${DISTRIBUTION_ID} ]; then

  DISTRIBUTION_ETAG=$(aws cloudfront get-distribution-config \
    --id ${DISTRIBUTION_ID} --query 'ETag' --output text)

  aws cloudfront get-distribution-config --id ${DISTRIBUTION_ID} \
    --query 'DistributionConfig' > /tmp/dist-config.json

  sed -i 's/    "Enabled": true,/    "Enabled": false,/g' /tmp/dist-config.json

  echo "disable configuration for: ${DISTRIBUTION_ID} / ${DISTRIBUTION_ETAG}"
  STATUS=$(aws cloudfront update-distribution --id ${DISTRIBUTION_ID} \
    --if-match ${DISTRIBUTION_ETAG} \
    --distribution-config file:///tmp/dist-config.json \
    --query 'Distribution.Status' --output text)
  # wait until in not InProgress, then we can delete
  echo "${DISTRIBUTION_ID}.Status = ${STATUS}"
  while [ "${STATUS}" = "InProgress" ]
  do
    STATUS=$(aws cloudfront list-distributions \
      --query 'DistributionList.Items[0].Status' --output text)
    echo "."
    sleep 5
  done
  # while doing some testing this fail to me a few times
  # maybe this sleep some more is what aws need to properly delete...
  sleep 5
  # get the new etag & wait a few minutes...
  DISTRIBUTION_ETAG=$(aws cloudfront get-distribution-config \
    --id ${DISTRIBUTION_ID} --query 'ETag' --output text)
  sleep 5
  aws cloudfront delete-distribution --id ${DISTRIBUTION_ID} \
    --if-match ${DISTRIBUTION_ETAG}

fi

aws s3 rb s3://${DOMAIN} --force
aws s3 rb s3://www.${DOMAIN} --force
# aws s3 ls

{{< /highlight >}}
