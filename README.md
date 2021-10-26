# crunchy-postgresql-db
This repo is used to create the Crunchy postgresql Database on OpenShift


---
---

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

Step5: To scale the cluster to 2 instances
```
oc apply -k crunchy-db-pgcluster/base
```

This concludes the Install and testing of the Postgresql DB on OpenShift 

---
---

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
Step2: Port forward the application to your local
```
oc get pods 
NAME                                     READY   STATUS      RESTARTS   AGE
pg-db-primary-00-k7qj-0                  3/3     Running     0          43m
pg-db-primary-backup-rcwh-x8j4w          0/1     Completed   0          43m
pg-db-primary-pgbouncer-d9b58546-t946f   2/2     Running     0          43m
pg-db-primary-repo-host-0                1/1     Running     0          43m

oc port-forward pod/pg-db-primary-00-k7qj-0 $DB_PORT:$DB_PORT
Forwarding from 127.0.0.1:5432 -> 5432
Forwarding from [::1]:5432 -> 5432
```

Step3: Run the DB client to connect to the server using the credentials. For example I am using the DBeaver community edition as a client to check the details
test the connectivity from you IDE to see if you can connect using the "redhat" user



