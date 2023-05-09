# Manual Setup
These instructions should be followed until we get the app created in Data Dog's marketplace

1. Configure the Cyral -> Datadog log integration as per [Cyral documentation](https://cyral.com/docs/integrations/siem/datadog-logs)
2. Create a Cyral Pipeline to be used for various mappings (See the Pipeline Setup Instructions below)
3. Create the facets needed for proper dashboard operation (See the Facet Creation below)
4. Install the Cyral Dashboard (See the Dashboard Installation below)

## Pipeline Setup Instructions

1. Login to your DataDog
2. Navigate to Logs -> Configuration -> Pipelines
3. Click the `New Pipeline` button
4. Add the following information on the `Create Pipeline` screen
   - Filter: `source:cyral-*-wire`
   - Name: `CyralPipeline`
5. Click the `Create` button.
6. Once the `CyralPipeline` pipeline has been created, create the processors listed in the `Processor Creation` section)

## Processor Creation
In order for the Cyral dashboards to function as expected, we need to create a few processors in the pipeline.
This section outlines the processors that are required along with the order in which they should be created.

1. Service Remapper : Cyral Service Mapper
   - service attribute(s) : svc
2. GeoIP Parser : IP Geo Mapper
   - Processor Type : GeoIP Parser
   - source IP attribute : client.host
   - target attribute path : network.client.geoip
   - Name : IP Geo Mapper
3. Remapper : Map Client IP
   - Processor Type : Remapper
   - attribute(s) or tag key to remap : client.host
   - target attribute or tag key : network.client.ip
   - Preserve source attribute : Enabled
   - Override on conflict : Disabled
   - Force attribute type : String
   - Name : Map Client IP
4. String Builder Processor : %{request.statementType}_%{repo.type} - in attribute lookup.categoryKey
   - Processor Type : String Builder Processor
   - target attribute path : lookup.categoryKey
   - target attribute value : %{request.statementType}_%{repo.type}
   - Replace missing attribute by an empty string : Enabled
   - Name : %{request.statementType}_%{repo.type} - in attribute lookup.categoryKey
5. Lookup Processor : Category Lookup
   - Processor Type : Lookup Processor
   - lookup mapping : Manual Mapping (Upload the ./lookups/CategoryLookupTable.csv from this directory)
   - Fallback value : Unknown
   - source attribute : lookup.categoryKey
   - target attribute path : lookup.categoryActivity
   - Name : Category Lookup
6. Lookup Processor : Term Lookup
   - Processor Type : Lookup Processor
   - lookup mapping : Manual Mapping (Upload the ./lookups/TermLookupTable.csv from this directory)
   - Fallback value : Unknown
   - source attribute : lookup.categoryKey
   - target attribute path : lookup.TermActivity
   - Name : Term Lookup
7. Lookup Processor : Privileged Command Lookup
   - Processor Type : Lookup Processor
   - lookup mapping : Manual Mapping (This should contain at least the line `Privileged Commands,True`. If you would like to include additional categories as "privileged", then list those here one per line)
   - source attribute : lookup.categoryActivity
   - target attribute path : lookup.priviledgedCmd
   - Name : Privileged Command Lookup

   * Some additional categories that could be used for this Manual Mapping are:
     - Access Changes
     - Data Changes
     - Data Reads
     - Schema Changes

   To consider additional categories as `Privileged` simply add them to the lookup, (The example below adds both `Privileged Commands` and `Access Changes` as "privileged" commands)
   `Privileged Commands,True
    Access Changes,True`

## Facet Creation
In order for the dashboards to function properly, you will need to create the following facets from the Cyral logs:

| Path                         | Display Name                | Type    | Group | Notes                                                                    |
|------------------------------|-----------------------------|---------|-------|--------------------------------------------------------------------------|
| @activityTypes               | activityTypes               | String  | Cyral |                                                                          |
| @client.applicationName      | client.applicationName      | String  | Cyral |                                                                          |
| @repo.name                   | repo.name                   | String  | Cyral |                                                                          |
| @sidecar.name                | sidecar.name                | String  | Cyral |                                                                          |
| @identity.repoUser           | identity.repoUser           | String  | Cyral |                                                                          |
| @identity.endUser            | identity.endUser            | String  | Cyral |                                                                          |
| @identity.group              | identity.group              | String  | Cyral |                                                                          |
| @client.connectionId         | client.connectionId         | String  | Cyral |                                                                          |
| @response.bytes              | response.bytes              | Integer | Cyral |                                                                          |
| @response.isError            | response.isError            | Boolean | Cyral |                                                                          |
| @response.executionTimeNanos | response.executionTimeNanos | Integer | Cyral |                                                                          |
| @repo.type                   | repo.type                   | String  | Cyral |                                                                          |
| @lookup.categoryActivity     | lookup.categoryActivity     | String  | Cyral |                                                                          |
| @lookup.TermActivity         | lookup.TermActivity         | String  | Cyral |                                                                          |
| @lookup.priviledgedCmd       | lookup.priviledgedCmd       | String  | Cyral |                                                                          |
| @action                      | action                      | String  | Cyral | This will only be available if you have integrated `Cyral Activity Logs` |
| @subject.user                | subject.user                | String  | Cyral | This will only be available if you have integrated `Cyral Activity Logs` |
| @details.repoName            | details.repoName            | String  | Cyral | This will only be available if you have integrated `Cyral Activity Logs` |

## Dashboard Installation

1. While logged into DataDog, navigate to Dashboards -> New Dashboard
2. On the `Create a Dashboard` screen, enter any Dashboard Name (this will get overwritten) and click the `New Dashboard` button
3. On the resulting screen, click the gear icon in the upper right corner of the screen
4. From the gear menu, click on `Import dashboard JSON...`
5. Import the ./dashboards/Cyral-DataActivityLogsStandardDashboard.json file from this repo
6. At the `Paste dashboard JSON` prompt, click the `Yes, Replace` button
