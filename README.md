# openshift-fluentd
S2I configuration for making a FluentD instance that will forward data from Openshift to Splunk

`oc new-app registry.access.redhat.com/rhscl/ruby-24-rhel7~https://github.com/larkly/openshift-fluentd.git --name=secure-forwarder -l component=fluentd-forwarder -e GEM_PATH=/opt/app-root/src/bundle/ruby/2.4.0 -n logging`
