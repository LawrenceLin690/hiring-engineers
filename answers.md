<ins>Part 1: Collecting Metrics</ins><br/>
1a: Screenshot of Host Map showing tags set via Agent Config file<br/>
<br/>
![alt text](1a.png)<br/>
1b: Integrated PostgreSQL; screenshot of datadog-agent status<br/>
![alt text](1b.png)<br/>
1 Bonus: Not sure how to override the collection interval outside of modifying the yaml file, but you could set up a cron job and script to monitor a certain metric, send the data points to Datadog via API, and view/add a Dashboard.<br/>
<br/>
<ins>Part 2: Visualizing Data</ins><br/>
2a. I had trouble getting my script to create the 2nd requirement (integration metric with anomaly function applied).  I created a Timeboard manually with the required info,  requested all Timeboards via the API to get the id, and finally got that id's definition which returned: <br/>
<br/>
{"q":"anomalies(avg:postgresql.db.count{host:Lawrences-MacBook-Pro.local}, 'basic', 2)","style":{"palette":"dog_classic","width":"normal","type":"solid"}<br/>
<br/>
which is the same if I look at the JSON view of editor.  The response from the API with this request field was:  <br/>
<br/>
{"errors": ["Error parsing query: unable to parse anomalies(avg:postgresql.db.count{host:Lawrences-MacBook-Pro.local}, basic, 2: Rule 'scope_expr' didn't match at ', 2' (line 1, column 75)."]}<br/>
<br/>
I tried shifting the order to match the details when you hover over the timeseries but that also didn't.  I also tried with the outliers function with no success, and am included what I used, but applying the cumsum function instead of anomalies.<br/>
<br/>
curl  -X POST -H "Content-type: application/json" -d '{
      "graphs" : [{
          "title": "Custom Metric Timetable via API",
          "definition": {
              "events": [],
              "requests": [
                  {"q": "avg:my_metric{host:Lawrences-MacBook-Pro.local}"},
                  {"q": "cumsum(avg:postgresql.db.count{host:Lawrences-MacBook-Pro.local})"},
                  {"q": "avg:my_metric{host:Lawrences-MacBook-Pro.local}.rollup(sum, 3600)","style":{"palette":"dog_classic","width":"normal","type":"solid"},"type":"bars"}
              ],
              "viz": "timeseries"
          }
      }],
      "title" : "Custom Metric Shell via API",
      "description" : "A dashboard with custom metric info utilizing Datadog API.",
      "template_variables": [{
          "name": "var",
          "prefix": "null",
          "default": "*"
      }],
      "read_only": "True"
}' "https://api.datadoghq.com/api/v1/dash?api_key=<API_KEY>&application_key=<APP.KEY>"<br/>
<br/>
2 Bonus: The anomaly function highlights any unexpected behavior based on past data and a standard deviation defined by the user.<br/>
<br/>
<ins>Part 3: Monitoring Data</ins>
3a: Screenshots of Monitor Emails
[alt text](3alert.png)
[alt text](3warn.png)
[alt text](3nodata.png)
