Integrating Kafka with Elastic Search using Logstash

Step 1
Install all the required softwares- Zookeeper server, Kafka, Logstash, Elastic Search and Logstash.

Steps 2,3,4,5 - Creating the .conf file
Step 2
Create a file with a .conf extension, in the logstash main directory. The file will have 3 parts- input, filter and output. The Logstash software is like a pipeline which acts as a bridge between various input and output softwares. 

Step 3
Logstash provides an input plugin to take input from Kafka, in which various things can be specified. But the preliminary things required are the URL of the server and the topic from which the messages are to be consumed from. The plugin basically writes a Kafka consumer with all the specified properties.

Step 4
Any processing on the input message can be done using the filter plugins available in Logstash. Ruby code can be executed using this filter plugin.

Step 5
Logstash also provides a plugin to write the output to elastic search, using a dynamic mapping method. Only the index and the type of the mapping have to be specified. Logstash processes the incoming message as a variable namely event. This variable is an associative array, with various fields containing values. The incoming message is stored in 'message' index of the variable. The desired mapping of the Elastic Search index has to be created by assigning values to various fields in the event variable. Logstash maps the indices of the event variable to ES mapping indices. Example shown below.

Steps 6,7- Starting and testing the pipeline

Step 6
Start the Zookeeper and Kafka servers and create a topic.Mention the name of this topic in the .conf file. Start the producer and push some messages to the topic. Start Elastic Search and Kibana.

Step 7
Check the configuration of the .conf file by running the command bin\logstash -f myfilter.conf -v --configtest
If the configuration is OK, run the command bin\logstash -f myfilter.conf -v --debug --verbose . This starts the pipeline which listens to the topic mentioned in the .conf file. If everything goes right, you can see the message with appropriate mapping indices in Kibana.

Example .conf file - 
input{
	kafka{
		topic_id =>"starttopic"
		zk_connect =>"localhost:2181"
		type => "user"
	}
}
filter{
	ruby{
		code =>	"
			sample = event['message'].split('~');
			event['Name']= sample[0];
			event['GroupId']= sample[1];
			event['Address']= sample[2];
			event['PhoneNumber']= sample[3];" 
		remove_field => ["message"]
	}
}
output{
	elasticsearch{
		index => "barclays"
	}
}
The type of the mapping has to be specified in the input part of the plugin. The topic name and the zookeeper url also have to be specified. The sample filter written here creates the mapping namely Name, GroupId, Address and PhoneNumber with all string type. Any mapping created as above will be a default string index. Any other mapping which has to be created can be created by a PUT request, to the ES url. 

The output to ES has only one specification to be made, the index to be pushed to. 
