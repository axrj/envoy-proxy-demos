# HTTP Routing: Blue Green Traffic Splitting

## Demo Overview

![](../assets/demo-httproute-blue-green.png)

Key definition 1 - `virtual_hosts` in [front-envoy.yaml](front-envoy.yaml)
```yaml
    virtual_hosts:
    - name: backend
        domains:
        - "*"
        routes:
        - match:
            prefix: "/service/blue"
        route:
            weighted_clusters:
            clusters:
            - name: service_green
                weight: 10
            - name: service_blue
                weight: 90
        - match:
            prefix: "/service/red"
        route:
            cluster: service_red
```

Key definition 2 - `clusters` in [front-envoy.yaml](front-envoy.yaml)
```yaml
  clusters:
  - name: service_blue
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    load_assignment:
      cluster_name: service_blue
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service_blue
                port_value: 80
  - name: service_green
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    load_assignment:
      cluster_name: service_green
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service_green
                port_value: 80
  - name: service_red
    connect_timeout: 0.25s
    type: strict_dns
    lb_policy: round_robin
    http2_protocol_options: {}
    load_assignment:
      cluster_name: service_red
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: service_red
                port_value: 80
```

## Getting Started
```sh
git clone https://github.com/yokawasa/envoy-proxy-demos.git
cd envoy-proxy-demos/httproute-blue-green
```

> [NOTICE] Before you run this demo, make sure that all demo containers in previous demo are stopped!

## Run the Demo

### Build and Run containers

```sh
docker-compose up --build -d

# check all services are up
docker-compose ps --service

front-envoy
service_blue
service_green
service_red

# List containers
docker-compose ps

                Name                              Command               State                            Ports
---------------------------------------------------------------------------------------------------------------------------------------
httproute-blue-green_front-envoy_1     /docker-entrypoint.sh /bin ...   Up      10000/tcp, 0.0.0.0:8000->8000/tcp, 0.0.0.0:8001->8001/tcp
httproute-blue-green_service_blue_1    /bin/sh -c /usr/local/bin/ ...   Up      10000/tcp, 80/tcp                                        
httproute-blue-green_service_green_1   /bin/sh -c /usr/local/bin/ ...   Up      10000/tcp, 80/tcp                                        
httproute-blue-green_service_red_1     /bin/sh -c /usr/local/bin/ ...   Up      10000/tcp, 80/tcp
```

### Access each services

Access serivce_blue and check if blue background page is displayed with 90% possibility and green background page is displayed with 10% possibility

```sh
curl -s -v http://localhost:8000/service/blue
```

For example, you can repeat the access to '/service/blue' like this

```sh
while true; do curl -s http://localhost:8000/service/blue | grep Hello && sleep 1; done

Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from green (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from green (hostname: 14de8b900934 resolvedhostname:172.31.0.4)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
Hello from blue (hostname: 7d82da79d3bf resolvedhostname:172.31.0.5)
```


Access serivce_red and check if red background page is displayed
```
curl -s -v http://localhost:8000/service/red
```

## Stop & Cleanup
```sh
$ docker-compose down --remove-orphans --rmi all
```

---
[Top](../README.md)
