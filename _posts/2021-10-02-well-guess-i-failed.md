---
layout: post
title: Well, guess I failed!
categories: Miscellaneous Networking
---

My plan to keep coding the game did no last a single month.. that's ok.. I'm used to It.. 

This last months are moving so fast that I really lost track of another goals I still need to achieve this year, and when I stopped to count the days I still have to accomplish them, I realized that maybe I don't have enough time..

So now, I'm studying Kubernetes again.. YAY!

If everything goes accordingly with the plan, I'll get the CKS Certification at the end of next month (or in november).

Maybe afterwards I write a new post about my studying path and which resources I've used. As well as some exam tips..

Summed to that, I've been changing projects at work and I'm looking for a place to move.. maybe this time I'll live close to the beach!

## But..

As I dont like to have a post just talking about my life, I'll talk about a little side project I've made recently..

The code can be found [here](https://github.com/vc0sta/aws-sso), and it's a little tool I created to work with AWS in a SAML federated authentication environment.

It's not complex at all, and the main logic was just copy/pasted from AWS official documentation (you can find the link at the app's repo). I've just added some "quality of life" improvements, like:
 - a more friendly name for your account, instead of listing the arns for each IAM Role
 - a interactive way to choose which role or profile should be used (kudos to inquirer, very cool library!)

The motivation on creating this script was that, the company I'm working for is already using a similar one, but it works in a very BAD way, but I had no access to the source code to just modify it to work properly, so what I want to write about is how I found the correct url to use as target from my script.

## Intercepting outbound calls

I've found a very cool tool called [mitmproxy]() to create a proxy and inspect any HTTPS calls that pass through it.

It was almost as simple as just running the tool and setting my session to use it with `export HTTPS_PROXY=127.0.0.1:8080` but I had to make some further configuration before.

The first thing I had to do was to add the proxy Certification Authority trusted by OSX. As we are talking about HTTPS calls, all of them have a TLS/SSL certificate attached to it, and when you are proxying a secure call, you must make it trusted.

So, with the proxy running, I've proxyied all my connections calls through it as you can se in the image below:

![System Wide Proxy]({{site.baseurl}}/post_images/2021-10-02/system-wide-proxy.png)

Then I just downloaded the certificates accesing ***http://mitm.it/***, thats a very cool feature from `mitmproxy` actually. This URL is only accessible when you run a proxyied connection, due to it's own DNS resolution.

Well, after downloading and opening this certificate, I only changed the setting for it to be trusted:

![Trusting Certificate]({{site.baseurl}}/post_images/2021-10-02/trusting-certificate.png)

And thats enough to capture any HTTPS calls!

## The Tool

Once everything was configured, you just need to go to `mitmproxy` and press **i**, it will start a query with:

```
set intercept ''
```

In this command you can add some filters, for example:

```
# Capture everything
set intercept '~q'

# Filtering
set intercept '~d google\.com'
```

Here is a simple example:

![Curl request]({{site.baseurl}}/post_images/2021-10-02/curl-request.png)
![mitmproxy]({{site.baseurl}}/post_images/2021-10-02/mitmproxy.png)

## Intercepting the script calls

Just one more step to get the URL!

As I tried to intercept the call using mitmproxy, the script failed because of an untrusted Certificate Authority. Even after adding the CA ina system wide level..

```
requests.exceptions.SSLError: HTTPSConnectionPool(host='company-sso-domain.com', port=443): Max retries exceeded with url: /adfs/ls/IdpInitiatedSignOn.aspx?loginToRp=urn:amazon:webservices (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1129)'))
```

The logs showed that it was related to the `requests` Python lybrary, so I forced requests to use `mitm CA` as well.

```
export REQUESTS_CA_BUNDLE=~/Desktop/mitmproxy-ca-cert.cer
```

Yeah, I could have stopped on the previous step, as the url was already showing in the logs.. but as I was already here, I went a bit further to capture it using `mitmproxy`..

![Captured Request]({{site.baseurl}}/post_images/2021-10-02/captured-request.png)

