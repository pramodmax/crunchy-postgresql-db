# crunchy-postgresql-db
This repo is used to create the Crunchy postgresql Database on OpenShift



# Demo steps

Step1: Create a Subscription.
oc login --token=<your-sha-token> --server=https://openshift-api-server:6443
oc kustomize crunchy-db-main/base

Step2: Create a Postgresql DB cluster
oc kustomize crunchy-db-pgcluster/base

Step3: Login and push data to primary-active server 
oc  exec -it <primary-db-pod-name> -n <namespace> -- bash
psql << EOF 
CREATE TABLE dpmtable AS 
SELECT * FROM generate_series(1,100) as id, 
md5(random()::text) as data;
EOF

Step4: check if the data is updated in the other active db pod
oc  exec -it <primary-db-pod-name> -n <namespace> -- bash
psql -c "select * from dpmtable;"

This concludes the Install and testing of the Postgresql DB on OpenShift