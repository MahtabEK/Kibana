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
