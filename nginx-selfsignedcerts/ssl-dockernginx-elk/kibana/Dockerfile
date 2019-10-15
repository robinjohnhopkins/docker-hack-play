# FROM kibana:latest
FROM kibana:5.4.3

RUN apt-get update && apt-get install -y netcat

COPY entrypoint.sh /tmp/entrypoint.sh
RUN chmod +x /tmp/entrypoint.sh

#Sense was renamed to Console and it is already available on Kibana 5.*.
#RUN kibana-plugin list
#RUN kibana-plugin install elastic/sense

CMD ["/tmp/entrypoint.sh"]
