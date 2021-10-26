# crunchy-postgresql-db
This repo is used to create the Crunchy postgresql Database on OpenShift



# Demo1 [Create Postgresql Cluster and test the active-active connection] steps

Step1: Create a Subscription.
```
oc login --token=<your-sha-token> --server=https://openshift-api-server:6443
oc apply -k crunchy-db-main/base
```

Step2: Create a Postgresql DB cluster
```
oc apply -k crunchy-db-pgcluster/base
```

Step3: Login and push data to primary-active server 
```
oc  exec -it <primary-db-pod-name> -n <namespace> -- bash
psql << EOF 
CREATE TABLE dpmtable AS 
SELECT * FROM generate_series(1,100) as id, 
md5(random()::text) as data;
EOF
```

Step4: check if the data is updated in the other active db pod
```
oc  exec -it <primary-db-pod-name> -n <namespace> -- bash
psql -c "select * from dpmtable;"
```

This concludes the Install and testing of the Postgresql DB on OpenShift 


# Demo2 [Connect to the DB [hackathon2021db] using the "redhat" user] steps

Step1: Get the password and connection details of redhat from  pg-db-primary-pguser-redhat Secret
```
DB_NAME=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.dbname' -r| base64 -d)
DB_HOST=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.host' -r| base64 -d)
DB_PASSWORD=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.password' -r| base64 -d)
PG_BOUNCER_HOST=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.pgbouncer-host' -r| base64 -d)
PG_BOUNCER_PORT=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.pgbouncer-port' -r| base64 -d)
PG_BOUNCER_URI=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.pgbouncer-uri' -r| base64 -d)
DB_PORT=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.port' -r| base64 -d)
DB_HOST_URI=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.uri' -r| base64 -d)
DB_USER=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.user' -r| base64 -d)
DB_VERIFIER=$(oc get secret  pg-db-primary-pguser-redhat -o json | jq '.data.verifier' -r| base64 -d)
```
Step2: 