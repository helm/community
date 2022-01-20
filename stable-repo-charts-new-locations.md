# Stable chart repo new locations

## First try Artifact Hub

The canonical source for Helm charts is the [Artifact Hub](https://artifacthub.io/), an aggregator for distributed chart repos.

- [Chart end users](https://github.com/helm/community/blob/main/user-profiles.md#1-application-operator) looking for charts should look there.
- [Chart contributors and maintainers](https://github.com/helm/community/blob/main/user-profiles.md#2-application-distributor) – people who want to contribute features or fixes to individual chart source code, or help to maintain them – may also look there. Many existing chart maintainers list the chart source in the Artifact Hub `links` metadata, but this is optional and many do not.

## Ongoing effort to relocate charts to new repos

If you read this far, this file is a continuation of an ongoing effort to [relocate charts to new repos](https://github.com/helm/charts/issues/21103).
It continues to serve two needs:

1. Points end users for the now archived `stable` or `incubator` charts ([the releases prior to November 13, 2020](https://helm.sh/blog/new-location-stable-incubator-charts/)) to new chart locations, so the users can upgrade to newer and currently maintained versions of those charts.
2. Helps previous and/or potential new chart maintainers coordinate where to continue maintaining the chart source and repo automation tooling for each chart (or set of related charts) as a community. Thanks for your continuing work on Helm charts! ✨

We will maintain this file until these needs are solved in another way, or are no longer necessary to solve, whichever comes first 🤝

## Brief history to avoid confusion

When `helm/charts` stable and incubator [support plan](https://github.com/helm/charts/blob/master/README.md#status-of-the-project) and [Deprecation Timeline](https://github.com/helm/charts/blob/master/README.md#deprecation-timeline) were announced, the community (chart OWNERS, organizations, groups or individuals who want to host charts) began moving the source of those charts to new Helm repos according to the [Search of Distributed Repositories proposal](https://github.com/helm/community/blob/main/hips/archives/helm/distributed-search.md).

Those archived chart releases were listed first on the (now deprecated) [Helm Hub](https://hub.helm.sh/), and now on [Artifact Hub](#first-try-artifact-hub) along with the new versions of actively maintained chart source code hosted elsewhere.

The table below was moved from the  now in GitHub to help the community contribute to tracking this migration through Pull Requests.

## Status of relocated charts

### Notes on updating lists below

1. List all charts:

    ```bash
    find -d stable/ -mindepth 1 -maxdepth 1
    ```

2. List all charts marked `deprecated`:

    ```bash
    grep -l 'deprecated: true' stable/*/Chart.yaml | xargs -I {} dirname {}
    ```

3. Manually check each deprecated chart for status and deprecation issue link, and update applicable table and non-applicable list accordingly (apart from the flag above, the deprecation process is not consistent enough to automate this check)

### Non-applicable (purposefully deprecated)

- stable/acs-engine-autoscaler
- stable/ark
- stable/aws-cluster-autoscaler
- stable/dask-distributed
- stable/gcloud-endpoints
- stable/kube-lego
- stable/magic-namespace
- stable/mongodb-replicaset (#23747)
- stable/nginx-lego
- stable/rabbitmq-ha (#23746)
- stable/sematext-docker-agent

### Applicable

<!-- We use Inline HTML to make checkboxes in markdown tables for easier end user visual scanning. -->
<!-- markdownlint-disable MD033 -->
| Chart | Status | Issue/PR |
| -- | -- | -- |
| <ul><li>[ ] stable/aerospike</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/airflow</li></ul> | in progress: discussion started | https://github.com/apache/airflow/issues/10523 |
| <ul><li>[x] stable/ambassador</li></ul> | done | https://github.com/datawire/ambassador-chart/issues/9 |
| <ul><li>[x] stable/anchore-engine</li></ul> | done | https://github.com/helm/charts/pull/23509 |
| <ul><li>[ ] stable/apm-server</li></ul> | STATUS | URL |
| <ul><li>[x] stable/artifactory</li></ul> | done | https://github.com/helm/charts/pull/7627 |
| <ul><li>[x] stable/artifactory-ha</li></ul> | done | https://github.com/helm/charts/pull/7627 |
| <ul><li>[ ] stable/atlantis</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/auditbeat</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/aws-iam-authenticator</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/bitcoind</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/bookstack</li></ul> | STATUS | URL |
| <ul><li>[x] stable/buildkite</li></ul> | done | https://github.com/helm/charts/pull/9200 |
| <ul><li>[ ] stable/burrow</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/centrifugo</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/cerebro</li></ul> | STATUS | URL |
| <ul><li>[x] stable/cert-manager</li></ul> | done | https://github.com/helm/charts/pull/12970 |
| <ul><li>[ ] stable/chaoskube</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/chartmuseum</li></ul> | STATUS | URL |
| <ul><li>[x] stable/chronograf</li></ul> | done | https://github.com/helm/charts/pull/21233 |
| <ul><li>[ ] stable/clamav</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/cloudserver</li></ul> | STATUS | URL |
| <ul><li>[x] stable/cluster-autoscaler</li></ul> | done | https://github.com/kubernetes/autoscaler/pull/3341 |
| <ul><li>[ ] stable/cluster-overprovisioner</li></ul> | in progress | https://github.com/helm/charts/pull/23586 |
| <ul><li>[x] stable/cockroachdb</li></ul> | done | https://github.com/helm/charts/pull/23000 |
| <ul><li>[ ] stable/collabora-code</li></ul> | STATUS | URL |
| <ul><li>[x] stable/concourse</li></ul> | done | https://github.com/helm/charts/pull/19128 |
| <ul><li>[x] stable/consul</li></ul> | done | https://github.com/helm/charts/pull/22696 |
| <ul><li>[ ] stable/contour</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/coredns</li></ul> | in progress | https://github.com/coredns/coredns/issues/3905 |
| <ul><li>[ ] stable/cosbench</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/coscale</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/couchbase-operator</li></ul> | STATUS | URL |
| <ul><li>[x] stable/couchdb</li></ul> | done | https://github.com/helm/charts/pull/18079 |
| <ul><li>[x] stable/dask</li></ul> | done | https://github.com/helm/charts/pull/18419 |
| <ul><li>[X] stable/datadog</li></ul> | done | https://github.com/helm/charts/pull/23384 |
| <ul><li>[ ] stable/dex</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/distributed-jmeter</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/distributed-tensorflow</li></ul> | STATUS | URL |
| <ul><li>[x] stable/distribution</li></ul> | done | https://github.com/helm/charts/pull/7627 |
| <ul><li>[x] stable/dmarc2logstash</li></ul> | done | https://github.com/helm/charts/pull/22524 |
| <ul><li>[ ] stable/docker-registry</li></ul> | STATUS | URL |
| <ul><li>[x] stable/dokuwiki</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/drone</li></ul> | done | https://github.com/helm/charts/pull/21151 |
| <ul><li>[x] stable/drupal</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/efs-provisioner</li></ul> | STATUS | URL |
| <ul><li>[x] stable/elastabot</li></ul> | done | https://github.com/helm/charts/pull/22676 |
| <ul><li>[x] stable/elastalert</li></ul> | done | https://github.com/helm/charts/pull/22689 |
| <ul><li>[ ] stable/elastic-stack</li></ul> | STATUS | URL |
| <ul><li>[x] stable/elasticsearch</li></ul> | done | https://github.com/helm/charts/pull/21955 |
| <ul><li>[ ] stable/elasticsearch-curator</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/elasticsearch-exporter</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/envoy</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/etcd-operator</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/ethereum</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/eventrouter</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/express-gateway</li></ul> | STATUS | URL |
| <ul><li>[x] stable/external-dns</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/factorio</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/filebeat</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/fluent-bit</li></ul> | migrated: need to deprecate chart with migration path | https://github.com/helm/charts/issues/21235 |
| <ul><li>[ ] stable/fluentd</li></ul> | migrated: need to deprecate chart with migration path | https://github.com/helm/charts/issues/21235 |
| <ul><li>[x] stable/fluentd-elasticsearch</li></ul> | done | https://github.com/helm/charts/pull/10354 |
| <ul><li>[x] stable/g2</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[ ] stable/gangway</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/gce-ingress</li></ul> | STATUS | URL |
| <ul><li>[x] stable/gcloud-sqlproxy</li></ul> | done | https://github.com/helm/charts/pull/9219 |
| <ul><li>[ ] stable/gcp-night-king</li></ul> | STATUS | URL |
| <ul><li>[x] stable/ghost</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/gitlab-ce</li></ul> | done | https://github.com/helm/charts/pull/1876 |
| <ul><li>[x] stable/gitlab-ee</li></ul> | done | https://github.com/helm/charts/pull/1876 |
| <ul><li>[ ] stable/gocd</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/goldpinger</li></ul> | STATUS | URL |
| <ul><li>[x] stable/grafana</li></ul> | done | https://github.com/helm/charts/pull/23662 |
| <ul><li>[x] stable/graphite</li></ul> | done | https://github.com/helm/charts/pull/10350 |
| <ul><li>[x] stable/graylog</li></ul> | done | helm/charts#24011 |
| <ul><li>[ ] stable/hackmd</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hadoop</li></ul> | STATUS | URL |
| <ul><li>[x] stable/hazelcast</li></ul> | done | https://github.com/helm/charts/pull/22797 |
| <ul><li>[x] stable/hazelcast-jet</li></ul> | done | https://github.com/helm/charts/pull/22798 |
| <ul><li>[ ] stable/heapster</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/heartbeat</li></ul> | STATUS | URL |
| <ul><li>[x] stable/helm-exporter</li></ul> | migrated: need to update README | https://github.com/helm/charts/pull/20376 |
| <ul><li>[ ] stable/hl-composer</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hlf-ca</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hlf-couchdb</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hlf-ord</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hlf-peer</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hoard</li></ul> | STATUS | URL |
| <ul><li>[x] stable/home-assistant</li></ul> | done | https://github.com/helm/charts/pull/22745 |
| <ul><li>[ ] stable/horovod</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/hubot</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/ignite</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/inbucket</li></ul> | STATUS | URL |
| <ul><li>[x] stable/influxdb</li></ul> | done | https://github.com/influxdata/helm-charts |
| <ul><li>[ ] stable/ingressmonitorcontroller</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/instana-agent</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/ipfs</li></ul> | STATUS | URL |
| <ul><li>[x] stable/jaeger-operator</li></ul> | done | https://github.com/helm/charts/pull/19636 |
| <ul><li>[ ] stable/janusgraph</li></ul> | STATUS | URL |
| <ul><li>[x] stable/jasperreports</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/jenkins</li></ul> | done | https://github.com/helm/charts/issues/23562 |
| <ul><li>[x] stable/joomla</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/k8s-spot-rescheduler</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/k8s-spot-termination-handler</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/kafka-manager</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/kanister-operator</li></ul> | STATUS | URL |
| <ul><li>[x] stable/kapacitor</li></ul> | done | https://github.com/helm/charts/pull/21234 |
| <ul><li>[ ] stable/karma</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/katafygio</li></ul> | STATUS | URL |
| <ul><li>[x] stable/keel</li></ul> | done | https://github.com/helm/charts/pull/9514 |
| <ul><li>[x] stable/keycloak</li></ul> | done | https://github.com/helm/charts/pull/13316 |
| <ul><li>[x] stable/kiam</li></ul> | done | https://github.com/helm/charts/pull/17959 |
| <ul><li>[ ] stable/kibana</li></ul> | in progress | https://github.com/helm/charts/pull/23844 |
| <ul><li>[x] stable/kong</li></ul> | done | https://github.com/helm/charts/pull/20149 |
| <ul><li>[ ] stable/kube2iam</li></ul> | in progress: conversation started | https://github.com/jtblin/kube2iam/issues/277 |
| <ul><li>[ ] stable/kube-hunter</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/kube-ops-view</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/kube-slack</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/kube-state-metrics</li></ul> | in progress | https://github.com/kubernetes/kube-state-metrics/issues/1153 |
| <ul><li>[x] stable/kubed</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[x] stable/kubedb</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[x] stable/kuberhealthy</li></ul> | done | https://github.com/helm/charts/pull/23919 |
| <ul><li>[x] stable/kubernetes-dashboard</li></ul> | done | https://github.com/helm/charts/pull/22627 |
| <ul><li>[ ] stable/kuberos</li></ul> | STATUS | URL |
| <ul><li>[x] stable/kubewatch</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[X] stable/kured</li></ul> | done | https://github.com/weaveworks/kured/pull/150 |
| <ul><li>[ ] stable/lamp</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/linkerd</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/locust</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/logdna-agent</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/logstash</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/luigi</li></ul> | STATUS | URL |
| <ul><li>[x] stable/magento</li></ul> | done | https://github.com/helm/charts/pull/14555 |
| <ul><li>[ ] stable/magic-ip-address</li></ul> | STATUS | URL |
| <ul><li>[x] stable/mailhog</li></ul> | done | https://github.com/helm/charts/pull/13315 |
| <ul><li>[x] stable/mariadb</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/mattermost-team-edition</li></ul> | done | https://github.com/helm/charts/pull/13540 |
| <ul><li>[ ] stable/mcrouter</li></ul> | STATUS | URL |
| <ul><li>[x] stable/mediawiki</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/memcached</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/mercure</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/metabase</li></ul> | STATUS | URL |
| <ul><li>[x] stable/metallb</li></ul> | done | https://github.com/helm/charts/pull/23486 |
| <ul><li>[ ] stable/metricbeat</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/metrics-server</li></ul> | in progress | https://github.com/kubernetes-sigs/metrics-server/issues/572 |
| <ul><li>[ ] stable/minecraft</li></ul> | STATUS | URL |
| <ul><li>[x] stable/minio</li></ul> | done | https://charts.min.io |
| <ul><li>[x] stable/mission-control</li></ul> | done | https://github.com/helm/charts/pull/7627 |
| <ul><li>[x] stable/mongodb</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/moodle</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/msoms</li></ul> | in progress: discussion started | https://github.com/microsoft/charts/issues/19 |
| <ul><li>[ ] stable/mssql-linux</li></ul> | in progress: discussion started | https://github.com/microsoft/charts/issues/19 |
| <ul><li>[ ] stable/mysql</li></ul> | STATUS | URL |
| <ul><li>[x] stable/mysqldump</li></ul> | done | https://github.com/helm/charts/pull/23840 |
| <ul><li>[ ] stable/namerd</li></ul> | STATUS | URL |
| <ul><li>[x] stable/nats</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/neo4j</li></ul> | done | https://github.com/helm/charts/pull/22437 |
| <ul><li>[ ] stable/newrelic-infrastructure</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/nextcloud</li></ul> | done | https://github.com/helm/charts/pull/23627 |
| <ul><li>[ ] stable/nfs-client-provisioner</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/nfs-server-provisioner</li></ul> | STATUS | URL |
| <ul><li>[x] stable/nginx-ingress</li></ul> | done | https://github.com/helm/charts/pull/22823 |
| <ul><li>[ ] stable/nginx-ldapauth-proxy</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/node-problem-detector</li></ul> | STATUS | URL |
| <ul><li>[x] stable/node-red</li></ul> | done | https://github.com/helm/charts/pull/22739 |
| <ul><li>[ ] stable/oauth2-proxy</li></ul> | STATUS | URL |
| <ul><li>[x] stable/odoo</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/opa</li></ul> | STATUS | URL |
| <ul><li>[x] stable/opencart</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[X] stable/openebs</li></ul> | done | https://github.com/helm/charts/pull/22860 |
| <ul><li>[ ] stable/openiban</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/openldap</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/openvpn</li></ul> | STATUS | URL |
| <ul><li>[x] stable/orangehrm</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/osclass</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/owncloud</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/pachyderm</li></ul> | STATUS | URL |
| <ul><li>[x] stable/parse</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/percona</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/percona-xtradb-cluster</li></ul> | STATUS | URL |
| <ul><li>[x] stable/pgadmin</li></ul> | done | https://github.com/helm/charts/pull/21275 |
| <ul><li>[x] stable/phabricator</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/phpbb</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/phpmyadmin</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/pomerium</li></ul> | STATUS | URL |
| <ul><li>[x] stable/postgresql</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/prestashop</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/presto</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/prisma</li></ul> | STATUS | URL |
| <ul><li>[x] stable/prometheus</li></ul> | done | https://github.com/helm/charts/pull/23692 |
| <ul><li>[x] stable/prometheus-adapter</li></ul> | done | https://github.com/helm/charts/pull/23694 |
| <ul><li>[x] stable/prometheus-blackbox-exporter</li></ul> | done | https://github.com/helm/charts/pull/23695 |
| <ul><li>[x] stable/prometheus-cloudwatch-exporter</li></ul> | done | https://github.com/helm/charts/pull/23696 |
| <ul><li>[x] stable/prometheus-consul-exporter</li></ul> | done | https://github.com/helm/charts/pull/23697 |
| <ul><li>[x] stable/prometheus-couchdb-exporter</li></ul> | done | https://github.com/helm/charts/pull/23698 |
| <ul><li>[x] stable/prometheus-mongodb-exporter</li></ul> | done | https://github.com/helm/charts/pull/23699 |
| <ul><li>[x] stable/prometheus-mysql-exporter</li></ul> | done | https://github.com/helm/charts/pull/23700 |
| <ul><li>[x] stable/prometheus-nats-exporter</li></ul> | done | https://github.com/helm/charts/pull/23701 |
| <ul><li>[x] stable/prometheus-node-exporter</li></ul> | done | https://github.com/helm/charts/pull/23702 |
| <ul><li>[x] stable/prometheus-operator</li></ul> | done | https://github.com/helm/charts/pull/23738 |
| <ul><li>[x] stable/prometheus-postgres-exporter</li></ul> | done | https://github.com/helm/charts/pull/23703 |
| <ul><li>[x] stable/prometheus-pushgateway</li></ul> | done | https://github.com/helm/charts/pull/23704 |
| <ul><li>[x] stable/prometheus-rabbitmq-exporter</li></ul> | done | https://github.com/helm/charts/pull/23705 |
| <ul><li>[x] stable/prometheus-redis-exporter</li></ul> | done | https://github.com/helm/charts/pull/23706 |
| <ul><li>[x] stable/prometheus-snmp-exporter</li></ul> | done | https://github.com/helm/charts/pull/23708 |
| <ul><li>[x] stable/prometheus-to-sd</li></ul> | done | https://github.com/helm/charts/pull/23707 |
| <ul><li>[ ] stable/quassel</li></ul> | STATUS | URL |
| <ul><li>[x] stable/rabbitmq</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/redis</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/redis-ha</li></ul> | STATUS | URL |
| <ul><li>[x] stable/redmine</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/reloader</li></ul> | done | https://github.com/helm/charts/pull/23595 |
| <ul><li>[ ] stable/rethinkdb</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/risk-advisor</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/rocketchat</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/rookout</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/sapho</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/satisfy</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/schema-registry-ui</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/sealed-secrets</li></ul> | Discussion ongoing - not settled | https://github.com/bitnami-labs/sealed-secrets/issues/389 |
| <ul><li>[x] stable/searchlight</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[ ] stable/selenium</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/sematext-agent</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/sensu</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/sentry</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/seq</li></ul> | STATUS | URL |
| <ul><li>[x] stable/signalfx-agent</li></ul> | done | https://github.com/helm/charts/pull/13586 |
| <ul><li>[ ] stable/signalsciences</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/socat-tunneller</li></ul> | STATUS | URL |
| <ul><li>[x] stable/sonarqube</li></ul> | done | https://github.com/helm/charts/pull/21007 |
| <ul><li>[x] stable/sonatype-nexus</li></ul> | done | https://github.com/helm/charts/pull/21255 |
| <ul><li>[ ] stable/spark</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/spark-history-server</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/spartakus</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/spinnaker</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/spotify-docker-gc</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/spring-cloud-data-flow</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/stackdriver-exporter</li></ul> | STATUS | URL |
| <ul><li>[x] stable/stash</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[ ] stable/stellar-core</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/stolon</li></ul> | STATUS | URL |
| <ul><li>[x] stable/suitecrm</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/sumokube</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/sumologic-fluentd</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/superset</li></ul> | STATUS | URL |
| <ul><li>[x] stable/swift</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[ ] stable/sysdig</li></ul> | STATUS | URL |
| <ul><li>[x] stable/telegraf</li></ul> | done | https://github.com/helm/charts/pull/21232 |
| <ul><li>[ ] stable/tensorflow-notebook</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/tensorflow-serving</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/terracotta</li></ul> | STATUS | URL |
| <ul><li>[x] stable/testlink</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[ ] stable/tomcat</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/traefik</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/uchiwa</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/unbound</li></ul> | STATUS | URL |
| <ul><li>[x] stable/unifi</li></ul> | done | https://github.com/helm/charts/pull/23714 |
| <ul><li>[ ] stable/vault-operator</li></ul> | STATUS | URL |
| <ul><li>[x] stable/velero</li></ul> | done | https://github.com/helm/charts/pull/19719 |
| <ul><li>[ ] stable/verdaccio</li></ul> | STATUS | URL |
| <ul><li>[x] stable/voyager</li></ul> | done | https://github.com/helm/charts/pull/4957 |
| <ul><li>[ ] stable/vsphere-cpi</li></ul> | STATUS | URL |
| <ul><li>[x] stable/wavefront</li></ul> | done | https://github.com/helm/charts/pull/20055 |
| <ul><li>[ ] stable/weave-cloud</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/weave-scope</li></ul> | in progress: discussion underway | https://github.com/weaveworks/scope/issues/3807 |
| <ul><li>[x] stable/wordpress</li></ul> | done | https://github.com/helm/charts/issues/20969 |
| <ul><li>[x] stable/xray</li></ul> | done | https://github.com/helm/charts/pull/7627 |
| <ul><li>[ ] stable/zeppelin</li></ul> | STATUS | URL |
| <ul><li>[ ] stable/zetcd</li></ul> | STATUS | URL |
