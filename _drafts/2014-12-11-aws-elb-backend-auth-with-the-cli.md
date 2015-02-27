---
layout: post
title: "Configuring ELB Backend Authentication with the AWS CLI"
excerpt: ""
categories: writing
tags: [chef, knife, knife-block]
comments: true
share: true
---

http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/configure_backendauth_clt.html

# Configuring ELB Backend Authentication with the AWS CLI


## step 1 - make public key policy


```
aws elb create-load-balancer-policy --load-balancer-name lookout-borderpatrol-stage1-my --policy-name BorderPatrolStage1MyAppPubKey --policy-type-name PublicKeyPolicyType --policy-attributes AttributeName=PublicKey,AttributeValue=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
```

check that it worked

```
aws elb describe-load-balancer-policies --region us-west-2 --load-balancer-name lookout-borderpatrol-stage1-my --output text --policy-names BorderPatrolStage1MyAppPubKey
```

## step 2 - make backend authentication policy, using public key policy

http://docs.aws.amazon.com/cli/latest/reference/elb/create-load-balancer-policy.html

- The name of the load balancer policy being created. The name must be unique within the set of policies for this load balancer.

```
aws elb create-load-balancer-policy --load-balancer-name lookout-borderpatrol-stage1-my --policy-name BorderPatrolStage1MyBackendAuth --policy-type-name BackendServerAuthenticationPolicyType --policy-attributes AttributeName=PublicKeyPolicyName,AttributeValue=BorderPatrolStage1MyAppPubKey
```

check that it worked

```
aws elb describe-load-balancer-policies --region us-west-2 --load-balancer-name lookout-borderpatrol-stage1-my --output text --policy-names BorderPatrolStage1MyBackendAuth
```

## step 3 - apply backend authentication policy to load balancer

```
aws elb set-load-balancer-policies-for-backend-server  --load-balancer-name lookout-borderpatrol-stage1-my --instance-port 443 --policy-names BorderPatrolStage1MyBackendAuth
```

check that it worked

```
aws elb describe-load-balancer-policies --load-balancer-name lookout-borderpatrol-stage1-my | grep POLICYD
```

