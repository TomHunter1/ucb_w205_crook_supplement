### UCB MIDS W205 - Kevin Crook's supplement for Synchronous Session #3

We will try to follow the official slides as close as we can in class.  I will post commands here to make them easier for students to copy and paste.

#### Ownership issues between science and root

Files created in your droplet will be owned by science with group science. Files created in your Docker containers will be owned by root with group root.  The following command can be used in the **droplet** when logged in as science to change the owner to science and the group to science, recursively, for a directory:
```
sudo chown -R science:science w205
```

### First three queries of assignment 02

What's the size of this dataset? (i.e., how many trips)
```sql
#standardSQL
SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
983648
```

What is the earliest start time and latest end time for a trip?
```sql
#standardSQL
SELECT min(start_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`

#standardSQL
SELECT max(end_date) 
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
2013-08-29 09:08:00
2016-08-31 23:48:00
```

How many bikes are there?
```sql
#standardSQL
SELECT count(distinct bike_number)
FROM `bigquery-public-data.san_francisco.bikeshare_trips`
```
```
700
```

### Using some Linux command line utilities to process a csv (comma separated value) file

The website explainshell.com is really good at finding out about linux command line:
https://explainshell.com/

Save data into your `w205` directory:
```
cd ~/w205
curl -L -o annot_fpid.json https://goo.gl/rcickz
curl -L -o lp_data.csv https://goo.gl/rks6h3
```

What's in this file?
```
head lp_data.csv
tail lp_data.csv
```

What are variable in here?
```
head -n1 lp_data.csv
```

How many entries?
```
cat lp_data.csv | wc -l
```

How about sorting?
```
cat lp_data.csv | sort
```

Take a look at what options there are for sort
```
man sort
```

fix so sorting correctly

```
cat lp_data.csv | sort -g
cat lp_data.csv | sort -n
```

Find out which topics are more popular

What have we got in this file?
```
head annot_fpid.json
```

Up until now, we have been using the droplet. The jq command is not part of standard linux.  jq is installed in our docker container midsw205/base, so we can startup a container, and run the remaining command in our container:
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

Hmmm, what now? jq
pretty print the json
```
cat annot_fpid.json | jq .
```

Just the terms
```
cat annot_fpid.json | jq '.[][]'
```

Remove the ""s
```
cat annot_fpid.json | jq '.[][]' -r
```

Can we sort that?
```
cat annot_fpid.json | jq '.[][]' -r | sort 
```

Hmmm, there's lots of repeated terms
Unique values only
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq 
```

How could I find out how many of each of those unique values there are?
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c 
```

Now, how could I sort by that?
Ascending
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c | sort -g
```
Descending
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c | sort -gr
```

So, what are the top ten terms?
```
cat annot_fpid.json | jq '.[][]' -r | sort | uniq -c | sort -gr | head -10
```



### Google BigQuery command line interface: bq cli

The gcloud command is not part of standard linux.  It's a Google utility.  bq is installed in our docker container midsw205/base, so if you are **NOT** already in a docker container from the previous jq exercise, you will want to do the following to get into a docker container:
```
docker run -it --rm -v /home/science/w205:/w205 midsw205/base:latest bash
```

setup bq cli
auth the GCP client
```
gcloud init
```

Follow the instructions.  This will include pasting a link into a Google Chrome browser that is logged into your Google Cloud account.  You may need to use igcognito for this for it to work right.  You will get a string to paste back into your docker container.

Here is a link to the Google Zones.  If you are running docker in the Digital Ocean droplet, select a region near northern California:
https://cloud.google.com/compute/docs/regions-zones/

Google Cloud configuration data is stored here:
container:
```
ls -l /w205/.config/gcloud
or
ls -l ~/.config/gcloud
```
droplet:
```
ls -l /home/science/w205/.config/gcloud
or
ls -l ~/w205/.config/gcloud
```


associate `bq` with a project
```
bq
```
and select project if asked

Run some Google BigQuery queries using the bq cli
```
bq query --use_legacy_sql=false 'SELECT count(*) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
Answer: something like 107,501,619


How many stations are there?
```
bq query --use_legacy_sql=false 'SELECT count(distinct station_id) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
Answer: something like 75

How long a time period do these data cover?
```
bq query --use_legacy_sql=false 'SELECT min(time), max(time) FROM `bigquery-public-data.san_francisco.bikeshare_status`'
```
Answer:  2013-08-29 12:06:01.000 UTC  2016-08-31 23:58:59.000 UTC   

### Advanced options 

Sort by 'product_name'
```
cat lp_data.csv | awk -F',' '{ print $2,$1 }' | sort
```

Fix the ""s issue
```
cat lp_data.csv  | awk -F',' '{ print $2,$1 }' | sed 's/"//' | sort | less
```
