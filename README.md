**ELK Banking Data**
This repository contains demo banking logs, along with the mapping policy and Logstash configuration files for parsing and visualizing the data in the ELK stack.

 Prerequisites
 
**ELK Stack:** Ensure you have Elasticsearch, Logstash, and Kibana installed.

**Filebeat:** To forward the logs to Logstash.


**📄 Sample Log Line**

Below is an example log entry for the banking domain:
2023-11-27 10:32:15 | Transaction ID: 12345 | User: John Doe | Payment Method: Visa | Amount: $50.00 | Status: Success | Payment Gateway: Stripe

**This log contains details like:**

Timestamp
Transaction ID
User
Payment Method
Amount
Status
Payment Gateway

**Grok Filter**
To extract meaningful information from the log line, use the following Grok filter:

%{TIMESTAMP_ISO8601:date} \| %{DATA}: %{NUMBER:transaction_id} \| %{DATA}%{DATA:user} \|%{DATA}: %{WORD:payment_method} \| %{WORD}: \$%{NUMBER:amount} \| %{WORD}: %{WORD:status} \| %{DATA}: %{WORD:payment_gateway}

**Mapping Template**
Use this mapping template in Kibana to define the data structure:

PUT /_template/banking_data?pretty=true
{
  "index_patterns": [
    "banking_data*"
  ],
  "settings": {
    "index.lifecycle.name": "banking_data"
  },
  "mappings": {
    "properties": {
      "date": {
        "type": "date"
      },
      "transaction_id": {
        "type": "integer"
      },
      "user": {
        "type": "keyword"
      },
      "payment_method": {
        "type": "keyword"
      },
      "amount": {
        "type": "integer"
      },
      "status": {
        "type": "keyword"
      },
      "payment_gateway": {
        "type": "keyword"
      },
      "error": {
        "type": "keyword"
      }
    }
  },
  "aliases": {
    "banking_data_logs": {}
  }
}


**Logstash Pipeline**
Below is the Logstash pipeline configuration for processing the logs:

Input Configuration

input {
  beats {
    port => 5044
  }
}


**Filter Configuration**

filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:date} \| %{DATA}: %{NUMBER:transaction_id} \| %{DATA}%{DATA:user} \|%{DATA}: %{WORD:payment_method} \| %{WORD}: \$%{NUMBER:amount} \| %{WORD}: %{WORD:status} \| %{DATA}: %{WORD:payment_gateway}"
    }
  }
  date {
    match => ["date", "yyyy-MM-dd HH:mm:ss"]
    target => "date"
    timezone => "UTC"
  }
}


**Output Configuration**

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "banking_data"
    user => "elastic"
    password => "password"
  }
}


