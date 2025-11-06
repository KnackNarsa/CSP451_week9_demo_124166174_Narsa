# Azure Demo Presentation
## CSP451 Presentation Demo - Sentiment Analysis Dashboard
## Nicholas Narsa - 124166174

**Date:** November 7, 2025
**Subscription:**  Seneca College : CSP451NIA - 1001
**Resource Group:** Student-RG-1886175
**Region:** Canada East (canadaeast)

---

## Overview

This proof of concept demonstrates how text is interpreted with Azure Cognitive Services Text Analytics to determine positive, negative, and neutral feelings to be stored in a database to be presented in Power BI via a Sentiment Analysis Dashboard

- **Azure Cognitive Services Text Analytics** - Interprets text by calling the keyphrases and sentiment api
- **CosmosDB** - Stores keyphrases and sentiment data to be used in Power BI
- **Logic App Trigger URL**: When user submits a review, the logic app runs the language service and stores the data to be used in Power BI, refreshing everytime a new entry is added into the database for a live sentiment analysis dashboard


---

## Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │ Logic App Trigger URL
       ▼
┌───────────────────────────────────────────────────┐  ┌────────────────┐
│   Azure Cognitive Services Text Analytics (APIs)  │  │                │
│                   - keyphrases                    │  │                │
│                   - sentiment                     │  │                │
└───────────────────────┬───────────────────────────┘  │                │
                        │                              │                │
                        ▼                              │                │
┌────────────────────────────────────────────────────┐ │                │
│Compose result { text, sentiment, confidenceScores, │ │                │
│keyPhrases, createdAt, ... } to store into database │ │                │
│                                                    │ │                │
└───────────────────────┬────────────────────────────┘ │                │
                        │                              │     Logic      │
                        ▼                              │      App       │
┌────────────────────────────────────────────────────┐ │                │
│               Cosmos DB stores data,               │ │                │
│                                                    │ │                │
│                                                    │ │                │
└───────────────────────┬────────────────────────────┘ │                │
                        │                              │                │
                        ▼                              │                │
┌────────────────────────────────────────────────────┐ │                │
│               Refresh POWER BI REST                │ │                │
│                                                    │ │                │
│                                                    │ │                │
└───────────────────────┬────────────────────────────┘ └────────────────┘
                        │                           
                        ▼
┌────────────────────────────────────────────────────┐ 
│               Power Bi Presents Results            │
│               (Sentiment Dashboard)                │
│                                                    │
└────────────────────────────────────────────────────┘                                                                         
```

## Resources Created

### 1. Azure Language Service
- **Name:** `nnarsa-presentation`
- **API Kind:** TextAnalytics
- **Tier:** Free
- **Location:** Canada East
- **Endpoint:** `https://nnarsa-presentation.cognitiveservices.azure.com/`
- **API Endpoints:** 
**URL**:`https://nnarsa-presentation.cognitiveservices.azure.com/text/analytics/v3.1/sentiment`
**URL**:`https://nnarsa-presentation.cognitiveservices.azure.com/text/analytics/v3.1/keyPhrases`           

### 2. Azure Cosmos DB
- **Name:** `nnarsa`
- **Read Locations:** Canada Central **Canada East was not available to choose**
- **Write Location:** Canada Central **Canada East was not available to choose**
- **Total Throughput:** 400 RU/s 
- **Endpoint:** `https://nnarsa.documents.azure.com:443/`

### 3. Logic App
- **Name:** `sentientassignment`
- **Location:** Canada East
- **Definition:** 1 trigger, 6 actions
- **Workflow Url:** `https://prod-31.canadaeast.logic.azure.com:443/workflows/92cb53728ab847d5a30f1e2c86cdf6d5/triggers/When_an_HTTP_request_is_received/paths/invoke?api-version=2016-10-01&sp=%2Ftriggers%2FWhen_an_HTTP_request_is_received%2Frun&sv=1.0&sig=DdHZc29qL0sFO0lV4vLYMV09c1ooPuuqLrllbNgf5HI`

## Logic App Workflow

```json
{
    "definition": {
        "$schema": "https://schema.management.azure.com/providers/Microsoft.Logic/schemas/2016-06-01/workflowdefinition.json#",
        "contentVersion": "1.0.0.0",
        "triggers": {
            "When_an_HTTP_request_is_received": {
                "type": "Request",
                "kind": "Http",
                "inputs": {
                    "schema": {
                        "text": "Sample",
                        "source": "review",
                        "userId": "demo-user",
                        "language": "en"
                    }
                }
            }
        },
        "actions": {
            "Sentiment": {
                "type": "Http",
                "inputs": {
                    "uri": "https://nnarsa-presentation.cognitiveservices.azure.com/text/analytics/v3.1/sentiment",
                    "method": "POST",
                    "headers": {
                        "Ocp-Apim-Subscription-Key": "7DmCKuUcaitAsMxxmB3HgLwbgUxS0c4AmpDmbMsFFRyVGrRWx3blJQQJ99BKACREanaXJ3w3AAAaACOGChr2",
                        "Content-Type": "application/json"
                    },
                    "body": {
                        "documents": [
                            {
                                "id": "1",
                                "language": "@{triggerBody()?['language']}",
                                "text": "@{triggerBody()?['text']}"
                            }
                        ]
                    }
                },
                "runAfter": {},
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "KeyPhrases": {
                "type": "Http",
                "inputs": {
                    "uri": "https://nnarsa-presentation.cognitiveservices.azure.com/text/analytics/v3.1/keyPhrases",
                    "method": "POST",
                    "headers": {
                        "Ocp-Apim-Subscription-Key": "7DmCKuUcaitAsMxxmB3HgLwbgUxS0c4AmpDmbMsFFRyVGrRWx3blJQQJ99BKACREanaXJ3w3AAAaACOGChr2",
                        "Content-Type": "application/json"
                    },
                    "body": {
                        "documents": [
                            {
                                "id": "1",
                                "language": "@{triggerBody()?['language']}",
                                "text": "@{triggerBody()?['text']}"
                            }
                        ]
                    }
                },
                "runAfter": {
                    "Sentiment": [
                        "Succeeded"
                    ]
                },
                "runtimeConfiguration": {
                    "contentTransfer": {
                        "transferMode": "Chunked"
                    }
                }
            },
            "Compose": {
                "type": "Compose",
                "inputs": {
                    "id": "@{guid()}",
                    "userId": "@{coalesce(triggerBody()?['userId'],'demo-user')}",
                    "source": "@{coalesce(triggerBody()?['source'],'review')}",
                    "createdAt": "@{utcNow()}",
                    "language": "@{coalesce(triggerBody()?['language'],'en')}",
                    "text": "@{triggerBody()?['text']}",
                    "sentiment": "@{body('Sentiment')?['documents']?[0]?['sentiment']\n}",
                    "confidenceScores": "@body('Sentiment')?['documents']?[0]?['confidenceScores']",
                    "keyPhrases": "@{body('KeyPhrases')?['documents']?[0]?['keyPhrases']\n\n\n}"
                },
                "runAfter": {
                    "KeyPhrases": [
                        "Succeeded"
                    ]
                }
            },
            "Create_or_update_document_(V3)": {
                "type": "ApiConnection",
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['documentdb']['connectionId']"
                        }
                    },
                    "method": "post",
                    "body": "@outputs('Compose')",
                    "path": "/v2/cosmosdb/@{encodeURIComponent('AccountNameFromSettings')}/dbs/@{encodeURIComponent('sentimentdb')}/colls/@{encodeURIComponent('results')}/docs"
                },
                "runAfter": {
                    "Compose": [
                        "Succeeded"
                    ]
                }
            },
            "Response": {
                "type": "Response",
                "kind": "Http",
                "inputs": {
                    "statusCode": 200,
                    "body": "@outputs('Compose')"
                },
                "runAfter": {
                    "Refresh_a_dataset": [
                        "Succeeded"
                    ]
                }
            },
            "Refresh_a_dataset": {
                "type": "ApiConnection",
                "inputs": {
                    "host": {
                        "connection": {
                            "name": "@parameters('$connections')['powerbi']['connectionId']"
                        }
                    },
                    "method": "post",
                    "path": "/v1.0/myorg/groups/@{encodeURIComponent('myworkspace')}/datasets/@{encodeURIComponent('7fc57dcb-0d5e-4b95-b177-d3392c8424f3')}/refreshes",
                    "queries": {
                        "pbi_source": "powerAutomate"
                    }
                },
                "runAfter": {
                    "Create_or_update_document_(V3)": [
                        "Succeeded"
                    ]
                }
            }
        },
        "outputs": {},
        "parameters": {
            "$connections": {
                "type": "Object",
                "defaultValue": {}
            }
        }
    },
    "parameters": {
        "$connections": {
            "type": "Object",
            "value": {
                "documentdb": {
                    "id": "/subscriptions/71d310bf-1718-4d11-87d1-99a7d4e2053f/providers/Microsoft.Web/locations/canadaeast/managedApis/documentdb",
                    "connectionId": "/subscriptions/71d310bf-1718-4d11-87d1-99a7d4e2053f/resourceGroups/Student-RG-1886175/providers/Microsoft.Web/connections/documentdb",
                    "connectionName": "documentdb"
                },
                "powerbi": {
                    "id": "/subscriptions/71d310bf-1718-4d11-87d1-99a7d4e2053f/providers/Microsoft.Web/locations/canadaeast/managedApis/powerbi",
                    "connectionId": "/subscriptions/71d310bf-1718-4d11-87d1-99a7d4e2053f/resourceGroups/Student-RG-1886175/providers/Microsoft.Web/connections/powerbi",
                    "connectionName": "powerbi"
                }
            }
        }
    }
}
```

---

## Logic App Schemas

**Trigger Schema**

```json
{
  "text": "Sample",
  "source": "review",
  "userId": "demo-user",
  "language": "en"
}
```

**Sentiment/KeyPhrases Schema**

```json
{
  "documents": [
    {
      "id": "1",
      "language": "@{triggerBody()?['language']}",
      "text": "@{triggerBody()?['text']}"
    }
  ]
}
```

## Logic App example run
Showcases what the logic app does when the http trigger occurs.

**Test Payload**

```json
{
  "text": "This product was awesome!",
  "userId": "nnarsa",
  "source": "review",
  "language": "en"
}
```

**Sentiment API**
 
```json
{
  "documents": [
    {
      "id": "1",
      "sentiment": "positive",
      "confidenceScores": {
        "positive": 1,
        "neutral": 0,
        "negative": 0
      },
      "sentences": [
        {
          "sentiment": "positive",
          "confidenceScores": {
            "positive": 1,
            "neutral": 0,
            "negative": 0
          },
          "offset": 0,
          "length": 25,
          "text": "This product was awesome!"
        }
      ],
      "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2025-01-01"
}
```

**KeyPhrases Api**

```json
{
  "documents": [
    {
      "id": "1",
      "keyPhrases": [
        "product"
      ],
      "warnings": []
    }
  ],
  "errors": [],
  "modelVersion": "2022-10-01"
}
```

**Compose**

```json
{
  "id": "a6bfcb06-4e82-4c89-b269-371d0ddb2a28",
  "userId": "nnarsa",
  "source": "review",
  "createdAt": "2025-11-06T06:03:13.7483817Z",
  "language": "en",
  "text": "This product was awesome!",
  "sentiment": "positive",
  "confidenceScores": {
    "positive": 1,
    "neutral": 0,
    "negative": 0
  },
  "keyPhrases": "[\"product\"]"
}
```
### Test Results

- ✅ Trigger activates
- ✅ Sentiment and keyphrases API interpret text
- ✅ Data is stored into CosmosDB
- ✅ PowerBI is connected to the database
- ✅ PowerBI sees new entries into the database
- ⚠️ CosmosDB Throughput being 400 RU/s is the bare minimum computing power
---

## Monthly Cost Analysis (1000 monthly submissions) (Estimated)
- **Language Service:** $2
- **Logic App:** $0.38
- **CosmosDB:** $23.36 <---- Fixed cost due to throughput fixed to 400 RU/s
- **Total:** $25.74
- **Total with Power Bi license**: $40.74

---

## Lessons Learned

1. **Azure Cognitive Services Text Analytics:** How it reads text to interpret sentiment and highlight key phrases

2. **Power BI:** Understand how Power BI interacts with CosmosDB and how to present relevant data

3. **Cosmos DB:** Storing the sentiment data is the majority of the costs


---

## Cleanup Command

```bash
# Delete resource group (deletes all resources)
az group delete --name Student-RG-1886175 --yes --no-wait
```
---

## Steps for Production


1. **API & endpoint hardening**
2. **Data protection & governance**
3. **Power BI security**
4. **Monitoring, alerting, and incident response**
5. **Network isolation**
6. **Identity & access**
7. **Reliability**

---
