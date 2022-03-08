# winston-logstash-tcp


```js
{
  "timestamp": new Date().toISOString(), // The time the payload was created
  "message": "",                         // JSON Stringified version of your message
  "level": "",                           // The logger level of the message
  "label": `${options.label}`,
  "application": `${options.applicationName}`,
  "serverName": `${options.localhost}`,
  "pid": `${options.pid}`
}
```

In this case when the log message is a string, boolean, or Number value, then the message is a stringified as:
```js
{
  "data": `${message}`
}
```

If `options.formatted` is set to `false`, then the entire Winston log message object is `JSON.stringified` and then set to logstash.

## Logstash Configuration
Having logstash ingest preformatted messages delivered by this module can be done with a configuration file similar to below:
```conf
input {
  # Sample input over TCP
  tcp {
    codec => json
    port => 28777
    add_field => { "category" => "winston_log" }
  }
}
filter {
  if [category] == "winston_log" {
    json {
      source => "message"
    }
    json {
      source => "data"
      remove_field => [ "[headers][secret]", "[headers][apikey]" ]
    }
  }
}
output {
  if [category] == "winston_log" {
    stdout {
      codec => json
    }
    elasticsearch {
      id => "winston_log_tcp"
      index => "winston_log-%{+YYYY.MM.dd}"
      hosts => ["192.168.1.1:9200"] # Use the address of your Elasticsearch server
    }
  }
}

