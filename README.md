# Kibana
The goal of this project is to gain familiarity with Kibana and ElasticSearch.

**Connecting to your VM**

To access elasticsearch and Kibana on your virtual machine, use the following command line. As long as the SSH connection is open, if you try to connect to Kibana on your local machine by going to http://localhost:5601 in your browser, that connection will be forwarded to your virtual machine, thus allowing you to use Kibana remotely. (In the command below, substitute the number of the VM you use).

****
ssh -i cs4417-lab-77.pem cloudera@cs4417-lab-77.gaul.csd.uwo.ca -L5601:localhost:5601
****

Note if you need to copy a file to your VM you can do:

****
scp -i cs4417-lab-77.pem thefile.txt cloudera@cs4417-lab-77.gaul.csd.uwo.ca:
****

**Installing Software**

Before you begin, update your VM. This could take 20 min or more:

****
sudo yum --disablerepo=\* --enablerepo=base,updates update
****

Now install ElasticSearch, Kibana, and jq on your virtual machine by executing the following commands in your SSH window:

****
sudo curl https://www.csd.uwo.ca/~dlizotte/elasticsearch.repo -o /etc/yum.repos.d/elasticsearch.repo

sudo yum -y install --enablerepo=elasticsearch elasticsearch-7.4.1

sudo yum -y install --enablerepo=elasticsearch kibana-7.4.1

sudo chown -R elasticsearch:elasticsearch /var/lib/elasticsearch/

wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.5/jq-linux64

chmod +x jq
****

**JVM Options**

Because we are using a fairly large example database (0.5gb), we need to adjust a few default options. Edit the JVM options file using the following command:

****
sudo nano /etc/elasticsearch/jvm.options
****

Edit the file so that the lines

****
#Xms represents the initial size of total heap space

#Xmx represents the maximum size of total heap space

-Xms1g

-Xmx1g
****

now read

****
#Xms represents the initial size of total heap space

#Xmx represents the maximum size of total heap space

-Xms4g

-Xmx4g
****

Make sure you edit the correct lines. Save with ctrl-o, exit with ctrl-x. This ensures that elasticsearch is allowed to use enough memory (4 gigabytes) to process all the tweets.

**Elastic Search Options**

Edit the elastic search options file using the following command:

****
sudo nano /etc/elasticsearch/elasticsearch.yml
****

Add the following lines to the top of the file

****
http.max_content_length: 1000mb

bootstrap.system_call_filter: false
****

Save with ctrl-o, exit with ctrl-x. This ensures that we can upload the entire file of tweets at once (about 500mb worth).

**Starting ElasticSearch and Kibana**

Having set these options, we can now start elasticsearch and kibana. Once started, you should never have to stop them, and they will remain running even if you disconnect from your VM.

****
sudo service elasticsearch start

sudo service kibana start
****

