# Week 4: API Gateways

Welcome to Week 4 of our course on Mobile and Distributed Systems. This week, we will dive into API Gateways, a crucial component of modern web development architectures. Our focus will be on understanding, implementing, and deploying an API Gateway using Node.js and Express.

## Learning Goals

By the end of this tutorial, you will:

- Understand API Gateway fundamentals.
- Implement an API Gateway with Node.js and Express.
- Secure APIs with Gateway policies.
- Design efficient API routing.
- Gain practical experience in API Gateway deployment.

## Prerequisites

- A GitHub account and familiarity with GitHub repositories.
- Proficiency in terminal or command-line interfaces.
- [Node.js and NPM](https://nodejs.org/en/download) installed on your development machine (for local VS Code only).
- Basic knowledge of Express.js.

# Tutorial Walkthrough

## Initial Setup in GitHub Codespaces

### Prepare your development environment:

Log into GitHub, navigate to the `api-gateway` repository, and open it in GitHub Codespaces. Alternatively, you can use your local VS Code instance

### Project Installation

- Run `npm install` in the Codespaces terminal to install project dependencies.

### Creating the API Gateway

- Use `npm run init` to initialize the Express Gateway.

### Following the configuration wizard

When initialising, you will see the following console prompt:

```console
> api-gateway@1.0.0 init
> npx eg gateway create

? What's the name of your Express Gateway?
```

- Enter the name of your API gateway, use a code-friendly name such as `test-gateway` or `my-gateway`.
- After you type the name, press <kbd>Enter</kbd> on the keyboard.
- Then press <kbd>Enter</kbd> for the rest of the prompts.

### Accessing and Testing your API Gateway

- Ensure the port is public and accessible.
- Test your API using tools like Postman.

Let's incorporate the detailed instructions for configuring the API within the README file, focusing on key aspects such as adding services to the gateway, locating the configuration file, and setting up API and service endpoints.

# Configuring the API Gateway

Configuring the API Gateway involves several key steps to ensure that it can manage and route requests effectively to different services. Follow these steps to configure your API Gateway to work with the SeismicAPI:

### Locating the Configuration File

First, identify the location of the configuration file. This file is essential for adding new services and configuring the behavior of the Express Gateway.

- **File Location**: `/path/to/your/gateway/config/gateway.config.yml`

### Adding Services to the Gateway

To connect REST APIs to your gateway, you will need to define service endpoints and API endpoints in the configuration file.

#### Step 1: Define Service Endpoints

Under the `serviceEndpoints` section of your `gateway.config.yml` file, add entries for the APIs you wish to route through the gateway. For example, to add a Seismic Activity Service Endpoint:

```yaml
serviceEndpoints:
  httpbin:
    url: "https://httpbin.org"
  seismic:
    url: "https://65ca483b3b05d29307e01640.mockapi.io/api"
```

This tells the Express Gateway about the location of the Seismic Activity API.

#### Step 2: Configure API Endpoints

Define a new `apiEndpoints` entry for routing requests. For instance, to set up an endpoint for seismic data:

```yaml
apiEndpoints:
  api:
    host: localhost
    paths: "/ip"
  seismicData:
    host: "localhost"
    paths: "/seismic/*"
```

This creates a new API endpoint that listens for requests on the `/seismic` path.

#### Step 3: Route Requests to the Service

Define a new pipeline that tells the Express Gateway how to handle requests coming to the `/seismic` endpoint:

```yaml
pipelines:
  default:
    apiEndpoints:
      - api
    policies:
      # Uncomment `key-auth:` when instructed to in the Getting Started guide.
      # - key-auth:
      - proxy:
          - action:
              serviceEndpoint: httpbin
              changeOrigin: true
  seismic:
    apiEndpoints:
      - seismicData
    policies:
      - log:
          action:
            message: "${req.method} ${req.originalUrl}"
      - proxy:
          - action:
              serviceEndpoint: seismic
              changeOrigin: true
```

This pipeline configuration routes requests to the seismic service endpoint based on the `seismicData` API endpoint.

#### Example of full `gateway.config.yml`

Here is an example of the fully configured API Gateway for SesimicAPI. You can use this as a reference to add other services.

```yaml
http:
  port: 8080
admin:
  port: 9876
  host: localhost
log:
  level: debug
apiEndpoints:
  api:
    host: localhost
    paths: "/ip"
  seismicData:
    host: "localhost"
    paths: "/seismic/*"
serviceEndpoints:
  httpbin:
    url: "https://httpbin.org"
  seismic:
    url: "https://65ca483b3b05d29307e01640.mockapi.io/api"
policies:
  - basic-auth
  - cors
  - expression
  - key-auth
  - log
  - oauth2
  - proxy
  - rate-limit
pipelines:
  default:
    apiEndpoints:
      - api
    policies:
      # Uncomment `key-auth:` when instructed to in the Getting Started guide.
      # - key-auth:
      - proxy:
          - action:
              serviceEndpoint: httpbin
              changeOrigin: true
  seismic:
    apiEndpoints:
      - seismicData
    policies:
      - log:
          action:
            message: "${req.method} ${req.originalUrl}"
      - proxy:
          - action:
              serviceEndpoint: seismic
              changeOrigin: true
```

### Accessing the URL of Your Gateway

After configuring the gateway, ensure the port is publicly accessible:

1. Go to the ports tab in your terminal.
2. Right-click on the port (the default is `8080`) and navigate to "Port Visibility".
3. Select "Public" to make the port accessible.

### Testing Your API

Use tools like Postman to test the API Gateway. Make sure to replace placeholders with actual values relevant to your configuration:

```plaintext
GET http://<your-url>:8080/seismic/london
```

**NOTE:** Replace `<your-url>` with the actual codespaces/localhost url you have.

# Week 4 Tasks

This week, you are tasked with integrating multiple APIs into your API Gateway. These tasks are designed to simulate a real-world scenario where an API Gateway serves as a central point for managing requests to different services.

### Task 1: Integrate the Daylight API

- **Objective**: Integrate the Daylight API to provide daylight information for different locations.
- **Endpoint Example URL**: `https://65ca483b3b05d29307e01640.mockapi.io/api/daylight/london`

**Response**

```json
{
  "sunrise": "07:39",
  "sunset": "16:48",
  "daylight": "9",
  "id": "london"
}
```

- **Instructions**:
  1. Add a service endpoint for the Daylight API in your `gateway.config.yml`.
  2. Configure an API endpoint for daylight data requests.
  3. Create a pipeline that routes `/daylight/{city}` requests to the Daylight service.

### Task 2: Integrate the International Space Station (ISS) API

- **Objective**: Integrate the ISS API to provide real-time location data of the ISS and information on astronauts currently in space.
- **Endpoints**:
  - ISS Location: `http://api.open-notify.org/iss-now`
  - People in Space: `http://api.open-notify.org/astros`
- **Instructions**:
  1. Define service endpoints for the ISS location and astronauts in space.
  2. Set up API endpoints for `/iss-location` and `/astronauts`.
  3. Implement pipelines that proxy requests to the corresponding ISS service endpoints.

### EXTRA CHALLENGE: Integrate the WeatherAPI

- **Objective**: Integrate the WeatherAPI you created in a previous week into the API gateway.
- **Instructions**:
  1. Open a new Codespace and start the WeatherAPI.
  2. Add a service endpoint for the WeatherAPI in your `gateway.config.yml`.
  3. Configure an API endpoint that routes weather requests through your gateway.
  4. Test the integration to ensure that weather data can be successfully retrieved through the API Gateway.

### Guidelines

- Ensure that your API Gateway is correctly configured to route requests to each of the integrated APIs.
- Test each integration thoroughly using tools like Postman or any HTTP client of your choice.
- Document any challenges or interesting findings you encounter during the integration process.

### Tips for Success

- Pay close attention to the configuration details for each service and API endpoint.
- Validate your `gateway.config.yml` file syntax to avoid common YAML pitfalls.
- Use the provided examples as a guide, but don't hesitate to explore additional functionalities or enhancements to your API Gateway.

## API Specification of the example API Gateway

```yaml
openapi: 3.0.0
info:
  title: Express Gateway API
  description: API Gateway configuration transformed into OpenAPI Specification
  version: "1.0.0"
servers:
  - url: http://localhost:8080
    description: Local API Gateway Server
  - url: https://shiny-couscous-646vg7p5x4j24j7v-8080.app.github.dev
    description: Codespaces API Gateway Server
paths:
  /ip:
    get:
      summary: Proxy to httpbin service
      operationId: getIp
      tags:
        - httpbin
      responses:
        "200":
          description: Successful response from httpbin
          content:
            application/json:
              schema:
                type: object
                properties:
                  origin:
                    type: string
                    description: Origin IP
        "default":
          description: Unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
  /seismic/{city}:
    parameters:
      - name: city
        description: The city to be searched
        in: path
        required: true
        schema:
          $ref: "#/components/schemas/CityName"
    get:
      summary: Proxy to seismic data service
      operationId: getSeismicData
      tags:
        - seismic
      responses:
        "200":
          description: Successful response with seismic data.
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/SesimicData"
        "default":
          description: Unexpected error
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/Error"
components:
  schemas:
    CityName:
      description: The name of the city to be searched.
      type: string
      example: london
    SesimicData:
      type: object
      properties:
        id:
          $ref: "#/components/schemas/CityName"
        magnitute:
          type: number
          example: 1.8
        latitude:
          type: string
          example: "49.2028"
        longitude:
          type: string
          example: "0.8642"
    Error:
      type: object
      properties:
        code:
          type: integer
          format: int32
        message:
          type: string
```

## Additional Resources

- [OpenAPI Specification](https://swagger.io/specification/)
- [Codespaces documentation](https://docs.github.com/en/codespaces)
- [Express Gateway Documentation](https://www.express-gateway.io/docs/)
