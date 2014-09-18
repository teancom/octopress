---
layout: post
title: "Intro to Cloud Foundry and Bosh Part 3"
date: 2014-09-14 22:41
comments: true
categories: 
---

https://github.com/cloudfoundry/cf-mysql-release into ~/workspace/

http://docs.cloudfoundry.org/deploying/openstack/uploading_bosh_stemcell.html

bosh upload stemcell https://s3.amazonaws.com/bosh-jenkins-artifacts/bosh-stemcell/vsphere/bosh-stemcell-2682-vsphere-esxi-ubuntu-trusty-go_agent.tgz

./update
bosh create release
bosh upload release
./bosh-lite/make_manifest_spiff_mysql
bosh deploy

rule.json (ip of the haproxy vm in the cf-warden-mysql deployment, as per bosh vms):
[
   {
      "destination": "10.244.1.18",
      "protocol": "all"
   }
]

cf create-security-group p-mysql rule.json
cf bind-running-security-group p-mysql

cf marketplace

cf create-service p-mysql 100mb-dev mysql-test

rails new testapp -d mysql

manifest.yml:

applications:
- name: testapp
  command: bundle exec rake db:migrate && bundle exec rails s -p $PORT
  services:
  - mysql-test

cf push testapp

open page, yes it's a 404 but that means your app can talk to the database.

Done!
