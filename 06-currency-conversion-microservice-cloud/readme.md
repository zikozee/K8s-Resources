# Currency Conversion Micro Service
Run com.in28minutes.microservices.currencyconversionservice.CurrencyConversionServiceApplication as a Java Application.

## Resources

- http://localhost:8100/currency-conversion/from/EUR/to/INR/quantity/10

```json
{
id: 10002,
from: "EUR",
to: "INR",
conversionMultiple: 75,
quantity: 10,
totalCalculatedAmount: 750
}
```

## Containerization

### Troubleshooting

- Problem - Caused by: com.spotify.docker.client.shaded.javax.ws.rs.ProcessingException: java.io.IOException: No such file or directory
- Solution - Check if docker is up and running!
- Problem - Error creating the Docker image on MacOS - java.io.IOException: Cannot run program “docker-credential-osxkeychain”: error=2, No such file or directory
- Solution - https://medium.com/@dakshika/error-creating-the-docker-image-on-macos-wso2-enterprise-integrator-tooling-dfb5b537b44e

### Creating Containers

- mvn package

### Running Containers

```
docker run --publish 8100:8100 --network currency-network --env CURRENCY_EXCHANGE_URI=http://currency-exchange:8000 in28min/currency-conversion:0.0.1-SNAPSHOT
```

#### Test API 
- http://localhost:8100/currency-conversion/from/EUR/to/INR/quantity/10

```
docker login
docker push @@@REPO_NAME@@@/currency-conversion:0.0.1-SNAPSHOT
```

#### Environment Variable

```
        env:     #CHANGE
          - name: CURRENCY_EXCHANGE_URI
            valueFrom:
              configMapKeyRef:
                key: CURRENCY_EXCHANGE_URI
                name: currency-exchange-uri-demo
```

### KUBERNETES CLIENT SIDE LOAD BALANCING
- ```html
        <dependency> <!-- CHANGE ::: LoadBalancing-->
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
        </dependency>

        <dependency> <!-- CHANGE spring Kubernetes-->
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-kubernetes-all</artifactId>
        </dependency>

  ```
- we use feign client in conjuction with ribbon client
- ```java
    @FeignClient(name = "currency-exchange")//Kubernetes Service Name
    @RibbonClient(name = "currency-exchange")
  ```
  
- to fix: Forbibben!Configured service account doesn't access
- we use 02-rbac.yaml

### BENEFIT OF SPRING_CLOUD_KUBERNETES dependency

- it integrates with config map and secrets which has the same name as the application name
- so if we have new properties added to the application, we only need add those values to the config-map or secrets 
- and it is picked up at startup