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

As mentioned above, you can test your connection by pointing your browser to http://localhost:5601/ and you can try some commands by clicking the "Dev tools" (wrench) icon on the left of the screen (you may need to scroll the menu down). This brings up the Kibana console. You may need to wait a bit for Kibana to get ready for you.

If you get into trouble, you may want to try rebooting your VM with

****
sudo reboot
****

and restarting the services.

**Getting tweets into Kibana**

If you bring up the Kibana console (click the wrench in the menu on the left) and run the following command:

****
GET _cat/indices?v
****

You should see something like

****
health status index                  uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .kibana_task_manager_1 1q4SJze_RuKgFzyATwkayw   1   0          2            0     34.2kb         34.2kb
green  open   .kibana_1              8Uz_g8dtTZauIwX9sOF21g   1   0          6            1     36.9kb         36.9kb
****

which indicates that Kibana is running, but no user data has been indexed yet.

For this project, it was supposed to use the same collection of tweets for the Kibana/elasticsearch exercises as for the mongodb exercises. Unfortunately, there are incompatibilities in terms of how the two tools format their tweets. We will use the jq command to re-format the tweets so that elasticsearch can consume them. Copy the following (very long) command into your SSH window and run it. This should generate a file called tweets.elastic.json which we will feed into elastic search. Creating the file will take 3 or 4 minutes.

****
./jq -c '# Apply f to composite entities recursively, and to atoms
def walk(f):
  . as $in
  | if type == "object" then
    reduce keys_unsorted[] as $key
      ( {}; . + { ($key):  ($in[$key] | walk(f)) } ) | f
  elif type == "array" then map( walk(f) ) | f
  else f
  end;
  { index: {_index: "tweets"} },del(._id)|walk(if (type == "object" and has("$numberLong")) then .["$numberLong"] else . end)' tweets.json > tweets.elastic.json
****

**Upload and index the tweets**

We are nearly ready to upload the tweets. Before we do, we need to provide elasticsearch with a "mapping" that tells how each field in a json object that represents a tweet is to be interpreted. This is called the "type" of the field. Elasticsearch will guess what type should be, but it is not very smart. In particular, it is not good at identifying dates. In this section, you will take the default mapping, which we provide, and modify it so that dates are correctly interpreted by Elasticsearch.

Consider the created_at field. Here is an example.

****
Mon Apr 17 02:49:03 +0000 2017
****

We can see that the date is formatted as Weekday (text), Month (text), Hour, Minute, Second (in 24-hour time), TimeZone, and Year. A date type in Elasticsearch within a mapping looks like the following:

****
{
      "type":   "date",
      "format": "yyyy-MM-dd"
}
****

Here, the "format" string must be written so that it describes the date format we actually have in our data. More information on mappings can be found here: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html and detailed information on date formats can be found here: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-date-format.html and here: http://www.joda.org/joda-time/apidocs/org/joda/time/format/DateTimeFormat.html. Hint: Date formats are tricky; for example, note that "M" and "MMM" have different meanings; note also that there are several options for timezone offsets; the one you want is "Z".

**1) Take the associated mapcommand.txt file and modify it so that the created_at field is correctly interpreted as a date.** 

**You can find the modified version called "modified-mapcommand.txt"**

The file is set up such that if you copy and paste the whole thing into the Kibana console, it will create a new index called tweets with the given mapping.

After creating the index with the mapping, to populate the index with tweets, run the following command:

****
curl -H "Content-Type: application/x-ndjson" -XPOST localhost:9200/tweets/_bulk --data-binary @tweets.elastic.json | ./jq '.' > upload_output.json
****

This will take about 3 minutes. If things have worked, and you bring up the Kibana console and run the following command:

****
GET _cat/indices?v
****

You should see something like

****
health status index   uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   tweets  hYLTgFO-QPiwTxoBKexC2Q   5   1     106508            0    361.5mb        361.5mb
green  open   .kibana nDzjp2JaTYqA3w-VuW4PIg   1   0          2            1     89.1kb         89.1kb
****

If not, you can check the status of the uploads by running this command in your SSH window:

****
nano upload_output.json
****

If something goes wrong, you can start again from scratch by running

****
DELETE tweets
****

in the Kibana console.

**Creating an index pattern**

Kibana needs to know which indices it should look at for queries; it does so by defining "index patterns" which match all indexes of interest for a particular task. Since we only have one index, this is trivial; create an index pattern called tweets* and choose created_at for the Time Filter. If you click the "Discover" icon (compass on left) and ensure that your time range is set from 16 April 2017 00:00 to 25 April 2017 00:00. Having done so, you should see a bar chart of when the tweets occurred, and see a list of tweets arranged by date.

If you need to edit or remove your index pattern, you can do so by clicking the gear in the menu on the left (again you may need to scroll down).

**2) You can find a screenshot of bar graph shown in the "Discover" window, with filename "discover.jpg"**

**Exploring and Visualizing the Data**

The questions below will use the "Visualize" component of Kibana, accessible from the menu on the left hand side of the screen. Note that any time you modify your settings, you will need to click the "Apply changes" button (right-pointing triangle) to get Kibana to re-generate the graph. You may wish to refer to these materials: https://www.elastic.co/guide/en/kibana/current/visualize.html

You can save your visualizations in your instance of Kibana.

For the questions below, you can find the answers in a file called "Answers.pdf". Answer the questions in complete sentences. For Question 5.2, you can check out the csv file called "Q5.1,csv".

**3) Create a vertical bar chart of number of tweets with in_reply_to_screen_name.keyword on the X-axis. Show the top 100 most common screen names.**

  - 3.1) Save the visualization in your instance of Kibana with the name "Q3.1".
  
    You can find it in this project folder as well: "Q31.png"
  
  - 3.2) How could peaks of in_reply_to_user_id be helpful to a company examining a dataset of tweets?
  
**4) Split the series produced in Question 3 by place.country.keyword using the Sub Aggregation. Aggregate into the top 5 terms, plus an "other" category.**

  - 4.1) Save the visualization in your instance of Kibana with the name Q4.1 and upload a screenshot as Q41.png
  
  - 4.3) Speculate on the source of the less-common entries. How could entries like this disrupt an optimized workflow? (Thinking question)
  
  - 4.4) Are there any missing values for the place.country.keyword field among these users?
  
  - 4.5) How many tweets to the most-replied screen name came from "Canada"?
  
**5) Create a bar graph of the number of tweets per hour. Use a one-hour interval.**

  - 5.1) Save the visualization in your instance of Kibana with the name "Q5.1".
  
   You can find it in this project folder as well: "Q51.png"

  - 5.2) At the bottom of the page, click the button to pull up a table of the results. Export this as a formatted result.
  
   You can find it in this project folder as well: "Q5.1.csv"
  
  - 5.3) Adapt your bar graph to show the number of tweets per hour of the top 5 reply recipients, as defined in Question 3. 
  
   You can find it in this project folder as well: "Q5.3.PNG"
   
** 6) Any Interesting Visualization**

  - X) Create any visualization of the twitter data, and explain why it is interesting.
  
Save the visualization in your instance of Kibana with the name "Interesting Visualization"

You can find it in this project folder as well: "Interesting Visualization.png". The explanation can be found in "Interesting Visualization.pdf".
