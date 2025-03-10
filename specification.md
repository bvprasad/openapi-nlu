# Enhanced OpenAPI Specification for Natural Language API Interaction and Domain Knowledge Graph Generation

**Proposed Standard Specification**

**Version 1.1**

---

## 1. Introduction

This specification proposes extensions to the OpenAPI Specification (OAS) to enhance API descriptions with semantic information crucial for enabling natural language interaction with APIs and for generating domain knowledge graphs. While existing OpenAPI specifications focus on technical aspects of API endpoints, payloads, and invocation methods, these extensions, prefixed with `x-nl-`, aim to enrich specifications with natural language descriptions, domain entity definitions, and relationship semantics. This bridges the gap between technical API details and domain-level understanding.

The enhanced specification facilitates:

- Training intelligent agents capable of understanding and interacting with APIs through natural language.
- Automated construction of domain knowledge graphs for improved API understanding and discovery.
- Enhanced API documentation with richer semantic information.

---

## 2. Terminology

- **API Domain**: The specific business or functional area that an API serves (e.g., "Weather Information," "E-commerce Orders").
- **Domain Entity**: A key concept or object within the API domain (e.g., "Location," "Product," "Customer").
- **Relationship**: A semantic connection or association between domain entities or between API components and domain entities (e.g., "hasWeatherCondition," "unitOf," "managesEntity").
- **Intent**: A natural language phrase expressing the user's goal or purpose when interacting with an API endpoint (e.g., "get current weather," "place an order").
- **Intent Catalog**: A collection of intents associated with API endpoints, used for intent recognition and mapping user requests to API operations.
- **Domain Knowledge Graph**: A graph-based representation of the API domain, comprising domain entities as nodes and relationships between them as edges, derived from the enhanced OpenAPI specification.

---

## 3. OpenAPI Extension Specifications

This section details the proposed `x-nl-` extensions to the OpenAPI Specification. All extensions are optional unless explicitly stated as required. These extensions are designed to supplement existing OpenAPI fields, utilizing standard fields wherever possible to avoid redundancy.

### 3.1 API Level Extensions

These extensions are defined at the root level of the OpenAPI specification document.

#### 3.1.1 x-nl-domain

- **Identifier**: `x-nl-domain`
- **Description**: Specifies the domain or business area to which the API belongs.
- **Data Type**: String
- **Context of Use**: OpenAPI root level
- **Cardinality**: Optional

**Example**:

```yaml
openapi: 3.0.0
info:
  title: Weather API
  version: 1.0.0
  description: API providing weather data.
x-nl-domain: "Weather Information"
```

#### 3.1.2 x-nl-entities

- **Identifier**: `x-nl-entities`
- **Description**: Defines the core domain entities managed by the API. This serves as the foundation for the domain knowledge graph.
- **Data Type**: Array of Objects
- **Context of Use**: OpenAPI root level
- **Cardinality**: Optional

**Object Properties** (for each entity definition):

- `name` *(String, Required)*: The name of the domain entity (e.g., "Location").
- `description` *(String, Optional)*: A natural language description of the entity.
- `relationships` *(Array of Objects, Optional)*: Relationships associated with this entity. Each relationship object contains:
  - `name` *(String, Required)*: Descriptive name for the relationship (e.g., "currentWeather").
  - `type` *(String, Required)*: The type of relationship (e.g., "hasCurrentWeather").
  - `targetEntity` *(String, Required)*: The name of the target domain entity (must be listed in `x-nl-entities`).

**Example**:

```yaml
openapi: 3.0.0
info:
  title: Weather API
  version: 1.0.0
  description: API providing weather data.
x-nl-domain: "Weather Information"
x-nl-entities:
  - name: "Location"
    description: "Represents a geographical location."
    relationships:
      - name: "currentWeather"
        type: "hasCurrentWeather"
        targetEntity: "WeatherCondition"
  - name: "WeatherCondition"
    description: "Represents current weather conditions."
  - name: "UnitOfMeasure"
    description: "Represents units of measurement."
```

### 3.2 Path/Endpoint Level Extensions

These extensions are defined within the `paths` section, under each path and HTTP method.

#### 3.2.1 x-nl-intent

- **Identifier**: `x-nl-intent`
- **Description**: Provides a concise natural language phrase that captures the primary intent of invoking this specific API endpoint. This is crucial for intent recognition.
- **Data Type**: String
- **Context of Use**: Operation Object (under each HTTP method)
- **Cardinality**: Recommended

**Example**:

```yaml
paths:
  /weather/current:
    get:
      summary: Get current weather
      operationId: getCurrentWeather
      x-nl-intent: "get current weather"
```

#### 3.2.2 x-nl-operations

- **Identifier**: `x-nl-operations`
- **Description**: Specifies the domain-level operations performed by this endpoint. These operations should align with the domain model and can be used for categorizing endpoint functionality.
- **Data Type**: Array of Strings
- **Context of Use**: Operation Object (under each HTTP method)
- **Cardinality**: Optional

**Example**:

```yaml
paths:
  /weather/current:
    get:
      summary: Get current weather
      operationId: getCurrentWeather
      x-nl-intent: "get current weather"
      x-nl-operations: ["Read", "RetrieveWeather"]
```

### 3.3 Parameter Level Extensions

These extensions are defined within the `parameters` array of an operation.

#### 3.3.1 x-nl-entity-type

- **Identifier**: `x-nl-entity-type`
- **Description**: Specifies the domain entity type that the parameter represents. This links API parameters to the domain knowledge graph entities.
- **Data Type**: String
- **Context of Use**: Parameter Object
- **Cardinality**: Recommended for parameters representing domain entities.

**Example**:

```yaml
parameters:
  - in: query
    name: city
    schema:
      type: string
    required: true
    description: "The city for which to retrieve weather information. Must be a valid city name."
    x-nl-entity-type: "Location"
```

#### 3.3.2 x-nl-relationships

- **Identifier**: `x-nl-relationships`
- **Description**: Defines parameter-specific relationships that override or add to the default relationships defined at the entity level. This allows for context-specific relationship definitions when a parameter is used in a particular operation.
- **Data Type**: Array of Objects
- **Context of Use**: Parameter Object
- **Cardinality**: Optional

**Relationship Object Properties**:

- `name` *(String, Required)*: Descriptive name for the relationship.
- `type` *(String, Required)*: The type of relationship.
- `targetEntity` *(String, Required)*: The name of the target domain entity.

**Example**:

```yaml
parameters:
  - in: query
    name: units
    schema:
      type: string
      enum: [metric, imperial]
    description: "Units for temperature. Use 'metric' for Celsius and 'imperial' for Fahrenheit."
    x-nl-entity-type: "UnitOfMeasure"
    x-nl-relationships:
      - name: "unitContext"
        type: "unitOf"
        targetEntity: "WeatherCondition"
```

### 3.4 Request Body/Response Body Extensions

These extensions are defined within the `requestBody` and `responses` sections of an operation.

#### 3.4.1 x-nl-entity-type (Body)

- **Identifier**: `x-nl-entity-type`
- **Description**: For request or response bodies that represent a primary domain entity, this extension specifies the entity type.
- **Data Type**: String
- **Context of Use**: Media Type Object (under `content` in Request Body Object and Response Object)
- **Cardinality**: Recommended for bodies representing domain entities.

**Example**:

```yaml
responses:
  '200':
    description: Successful response
    content:
      application/json:
        schema:
          $ref: '#/components/schemas/Weather'
        x-nl-entity-type: "WeatherCondition"
```

---

## 4. Domain Knowledge Graph Model

The enhanced OpenAPI specification is used to construct a domain knowledge graph. The graph model consists of the following node and relationship types:

### 4.1 Node Types

- **API Node**: Represents the API as a whole.
  - **Properties**: `name`, `description`, `domain`.

- **Endpoint Node**: Represents an API endpoint.
  - **Properties**: `path`, `intent`, `description`, `operations`, `operationId`.

- **Parameter Node**: Represents an API parameter.
  - **Properties**: `name`, `description`, `entityType`, `exampleValues`, `in` (parameter location).

- **Schema Node**: Represents a schema definition.
  - **Properties**: `name`, `description`, `entityType`.

- **Entity Node**: Represents a domain entity.
  - **Properties**: `name`, `description`, `domain`.

### 4.2 Relationship Types

- `:CONTAINS_ENDPOINT` (API Node → Endpoint Node)
- `:MANAGES_ENTITY` (Endpoint Node → Entity Node)
- `:HAS_PARAMETER` (Endpoint Node → Parameter Node)
- `:REPRESENTS_ENTITY` (Parameter Node → Entity Node)
- `:DEFINES_ENTITY` (Schema Node → Entity Node)
- Custom Relationship Types: Defined by `x-nl-relationships.type`. Examples: `:hasCurrentWeather`, `:unitOf`.

### 4.3 Relationship Inference Rules

- **Default Entity Relationships**: Relationships defined under `x-nl-entities[*].relationships` are created between Entity Nodes.
- **Parameter Relationships**: Relationships defined under `x-nl-relationships` in parameters are created from Parameter Nodes to Entity Nodes.
- **Entity Representation**: `x-nl-entity-type` on Parameters and Schemas implies `:REPRESENTS_ENTITY` or `:DEFINES_ENTITY` relationships to the corresponding Entity Nodes.
- **API Structure Relationships**: Relationships like `:CONTAINS_ENDPOINT`, `:HAS_PARAMETER` are derived directly from the OpenAPI structure.

---

## 5. Intent Catalog Generation

The intent catalog is generated by extracting the `x-nl-intent` values from each operation and associating them with the corresponding Endpoint Nodes in the knowledge graph. The catalog can be structured for efficient intent recognition, mapping user intents to API operations.

---

## 6. Usage Example (Combined)

```yaml
openapi: 3.0.0
info:
  title: Weather API
  version: 1.0.0
  description: API providing weather data.
x-nl-domain: "Weather Information"
x-nl-entities:
  - name: "Location"
    description: "Represents a geographical location."
    relationships:
      - name: "currentWeather"
        type: "hasCurrentWeather"
        targetEntity: "WeatherCondition"
      - name: "forecastWeather"
        type: "hasForecastWeather"
        targetEntity: "Forecast"
  - name: "WeatherCondition"
    description: "Represents current weather conditions."
  - name: "Forecast"
    description: "Represents weather forecast information."
  - name: "UnitOfMeasure"
    description: "Represents units of measurement."

paths:
  /weather/current:
    get:
      summary: Get current weather
      description: "Retrieve the current weather conditions for a given city."
      operationId: getCurrentWeather
      x-nl-intent: "get current weather"
      x-nl-operations: ["Read", "CurrentWeather"]
      parameters:
        - in: query
          name: city
          schema:
            type: string
          required: true
          description: "The city for which to retrieve weather information."
          x-nl-entity-type: "Location"
        - in: query
          name: units
          schema:
            type: string
            enum: [metric, imperial]
          description: "Units for temperature. Use 'metric' for Celsius and 'imperial' for Fahrenheit."
          x-nl-entity-type: "UnitOfMeasure"
          x-nl-relationships:
            - name: "unitContext"
              type: "unitOf"
              targetEntity: "WeatherCondition"
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Weather'
              x-nl-entity-type: "WeatherCondition"

components:
  schemas:
    Weather:
      type: object
      properties:
        city:
          type: string
          description: "The name of the city for which weather is reported."
        temperature:
          type: number
          format: float
          description: "The current temperature in the city."
        condition:
          type: string
          description: "A brief description of the current weather conditions, like 'Sunny', 'Cloudy', 'Rainy'."
```

---

## 7. Implementation Considerations

Tools implementing this specification should:

- **Parse Enhanced OpenAPI**: Extend OpenAPI parsers to recognize and process the `x-nl-` extensions.
- **Knowledge Graph Generation**: Implement logic to generate a domain knowledge graph based on the parsed specification and the relationship inference rules. Graph databases like Neo4j, Amazon Neptune, or JanusGraph can be used for storage.
- **Intent Catalog Extraction**: Extract `x-nl-intent` values and create an intent catalog for use in natural language understanding (NLU) components.
- **Agent Training Data Generation**: Utilize the `x-nl-intent`, `description`, `x-nl-entities`, and relationship information to generate training data for agents interacting with the API.
- **API Documentation Enhancement**: Leverage the `x-nl-` extensions to generate richer, more semantic API documentation.

---

## 8. Benefits and Considerations

### Benefits

- **Improved API Understanding**: Provides a semantic layer on top of technical API specifications, making APIs easier to understand for both humans and machines.
- **Natural Language API Interaction**: Enables the development of intelligent agents that can interact with APIs using natural language.
- **Automated Domain Knowledge Graph Construction**: Facilitates the automated creation of domain knowledge graphs, valuable for API discovery, analysis, and integration.
- **Enhanced API Documentation**: Allows for the generation of more informative and user-friendly API documentation.
- **Agent Training Data**: Provides structured semantic information that can be used to generate training data for NLU models and API agents.

### Considerations

- **Adoption Effort**: Requires API authors to adopt and populate the new `x-nl-` extensions in their OpenAPI specifications.
- **Tooling Ecosystem**: Requires development and adoption of tooling that supports parsing, processing, and utilizing these extensions.
- **Maintenance**: Maintaining the accuracy and completeness of the `x-nl-` extensions will be crucial for the effectiveness of the generated knowledge graphs and agent performance.
- **Standardization**: Further community review and standardization efforts would be beneficial to promote wider adoption and interoperability.

