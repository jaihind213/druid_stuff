############
To push metrics to graphite/grafana 
############

(0) start graphite

docker run -d --name graphite -p 2004:2004 -p 8080:80 -p 2003:2003 sitespeedio/graphite

(1) start grafana 

docker run -d --name=grafana -p 3000:3000 grafana/grafana
#and
#(create a graphite datasource with url http://ip:8080)


NOW ON EVERY DRUID BOX do the following:

(2) pull graphite emitter dep

java -classpath "$DRUID_HOME/lib/*" io.druid.cli.Main tools pull-deps  -c io.druid.extensions.contrib:graphite-emitter:0.9.1.1


(3) To help in debugging, set level to debug for TEMPORARY time period, undo this later

at $DRUID_HOME/conf/druid/_common/log4j2.xml 
-----------------------------------------------------------

<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="ERROR" monitorInterval="60">

    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5p %c{1}:%L - %msg%n" />
        </Console>
<!--TODO create /var/log/druid -->
        <RollingFile name="RollingFile" filename="/var/log/druid/druidlog.log"
            filepattern="${logPath}/%d{yyyyMMddHHmmss}-druid.log">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} [%t] %-5p %c{1}:%L - %msg%n" />
            <Policies>
                <SizeBasedTriggeringPolicy size="100 MB" />
            </Policies>
            <DefaultRolloverStrategy max="20" />
        </RollingFile>

    </Appenders>
    <Loggers>
        <Root level="DEBUG">
            <AppenderRef ref="Console" />
            <AppenderRef ref="RollingFile" />
        </Root>
    </Loggers>
</Configuration>
-----------------------------------------------------------



(4)in $DRUID_HOME/conf/druid/_common/common.runtime.properties, set the following:

-----------------------------------------------------------

druid.extensions.loadList=["druid-kafka-eight","druid-avro-extensions","druid-parquet-extensions","postgresql-metadata-storage","druid-s3-extensions","graphite-emitter"]

druid.monitoring.monitors=["com.metamx.metrics.JvmMonitor","io.druid.client.cache.CacheMonitor","io.druid.server.metrics.HistoricalMetricsMonitor","io.druid.server.metrics.QueryCountStatsMonitor","io.druid.server.metrics.HistoricalMetricsMonitor"]
druid.emitter=graphite
#use 2004 if using pickle , which is default else use 2003 for plaintext refer to 'druid.emitter.graphite.protocol'
druid.emitter.graphite.port=2004 
druid.emitter.graphite.hostname=10.2.10.234

#### if you want to send all metrics then use this event convertor
#druid.emitter.graphite.eventConverter={"type":"all", "namespacePrefix": "wakanda", "ignoreHostname":false, "ignoreServiceName":false}
##else if you want to have a whitelist of metrics sent -- the full whitelist available in this repo for your convenience.
druid.emitter.graphite.eventConverter={"type":"whiteList", "namespacePrefix": "wakanda", "ignoreHostname":false, "ignoreServiceName":false, "mapPath":"/path/to/file/graphite.extensive.whitelist"}

## i have created the extensive whitelist of all metrics in this repo itself. remove what u dont need. see file graphite.extensive.whitelist
#druid.emitter=logging
druid.emitter.logging.logLevel=debug
-----------------------------------------------------------



