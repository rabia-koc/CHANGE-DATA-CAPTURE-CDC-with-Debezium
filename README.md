   ![1](https://user-images.githubusercontent.com/73841520/219899544-8acb7985-5ee9-438a-91ed-ede1795ea9cd.png)

# Download the data:

curl https://raw.githubusercontent.com/erkansirin78/datasets/master/retail_db/customers.csv -o ~/datasets/customer.csv

## Start Docker service:

sudo systemctl status docker

sudo systemctl start docker

sudo systemctl status docker

## Stop local PostgreSQL:
sudo systemctl stop postgresql-10

# Step 1: Create docker-compose.yaml

## Services that should be in the container:

- spark
- postgresql
- minio
- kafka
- kafka-connect
- zookeeper

## Start and control docker-compose file:
- docker-compose up -d
- docker-compose ps 

# Step 2: Create Debezium PostgreSQL connector:
curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" http://localhost:8083/connectors/ -d @register-postgres.json


# Step 3: Create Kafka Console Consumer:
docker-compose exec kafka /kafka/bin/kafka-topics.sh --bootstrap-server kafka:9092 --list

docker-compose exec kafka /kafka/bin/kafka-console-consumer.sh --bootstrap-server kafka:9092 --from-beginning --property print.key=true --topic dbserver1.public.customers


# Step 4: Write to Postgtresql with data-generator

## PostgreSQL IP address:
docker inspect final_project_2-postgres-1 | grep "IPAddress"

## Connect data generator:
cd ..
cd data-generator/
source datagen/bin/activate

python dataframe_to_postgresql.py -hst 172.18.0.2 -p 5432 -u postgres -psw postgres -db postgres -i ~/datasets/customers.csv -t customers

## Connect PostgreSQL shell:
docker-compose exec postgres bash -c 'psql -p 5432 -h 172.18.0.2 -U $POSTGRES_USER postgres'

## Setting database configs:
ALTER TABLE public.customers REPLICA IDENTITY FULL;
ALTER TABLE public.customers ADD CONSTRAINT customers_pk PRIMARY KEY ("customerId");

## UPDATE and DELETE:
DELETE FROM customers WHERE "customerId"=1;
UPDATE customers SET "customerFName"='test' where "customerId"=3;


# Step 6: Write to Minio

## Connect spark-client
docker exec -it spark-client bash

## Add user spark
mc admin user add mlops_minio spark spark12345

## Set spark readwrite
mc admin policy set mlops_minio readwrite user=spark

## Create a bucket named cdc-data
mc mb mlops_minio/cdc-data

## Start jupyter lab
jupyter lab --ip 0.0.0.0 --port 8888 --allow-root

# Connect MinIO console:
http://localhost:9001


## Read full article on Medium:

