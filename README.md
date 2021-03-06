## DialogFlow Machine Learning Chatbot 
![chatbot](images/dialogflow-icon.png)

## Project Overview
This repository contains a backend service for an intelligent chatbot that onboards clients, answers questions, and sets up appointments on a retail website. The purpose of the backend service is to improve the customer experience that users have when interacting with a Dialogflow chatbot.

## Prerequisites
* [Create Dialogflow Agent](https://dialogflow.cloud.google.com/#/agent/)
* [Google Cloud Setup](https://cloud.google.com/dialogflow/es/docs/quick/setup)
* [Set up Node.JS](https://www.pluralsight.com/guides/getting-started-with-nodejs)
* [Redis Server - Managing Sessions](https://www.sitepoint.com/using-redis-node-js/)


## Google Cloud Platform Setup
* [Create a Cloud Platform project](https://console.cloud.google.com/cloud-resource-manager)
* [Enable Project Billing](https://cloud.google.com/billing/docs/how-to/modify-project)
* [Enable the Dialogflow API](https://console.cloud.google.com/marketplace/product/google/dialogflow.googleapis.com)
* [Authentication - Service Accounts](https://cloud.google.com/docs/authentication/getting-started)

## NodeJS Set Up
Node.js is an open-source, server-side runtime environment that allows you to create backend applications by writing JavaScript code. This repository uses Node to connect to 3rd party services such as Google Maps API, Google Sheets, Google Calendar, and Twilio in order to provide our Dialogflow agent the ability to schedule appointments on a corporate webiste for one of our clients.  

### Set up an Express Server using Node.JS

```javascript

const express = require('express')
const app = express()

```

### Navigate to root directory and install the following packages:

```
$ npm install express
```
 

### Run Backend Application on Local Host:

```
$ node service.js
```

### Set up a Public URL with NGROK Tunnel:

```
$ ngrok http <port>
```
 
## Copy forwarding link from ngrok 

- For this particular service, the public endpoint is: https://d81c-2601-1c2-8100-6540-5ddd-a1c3-a27f-aeaf.ngrok.io

## Add '/webhook' to the end of your public endpoint

```javascript

    app.post('/webhook', async(request, response) => {

```

- Example: https://d81c-2601-1c2-8100-6540-5ddd-a1c3-a27f-aeaf.ngrok.io/webhook

## Enter Webhook URL in Dialogflow Agent Console:

![Fulfillment-Service](images/Fulfillment-Screenshot.png)

- Navigate to https://dialogflow.cloud.google.com
- On the left side of the page, click the 'Fulfillment' tab
- In the 'Webhook' section, click 'Enable'
- Enter your public endpoint in the 'URL' field
- Click 'Save' at the bottom of the page
  - You can define when you want to send webhooks to this public endpoint in the DialogFlow console

## Create a Dialogflow Client

```javascript 

    const dialogflow = require('dialogflow');
    const sessionClient = new dialogflow.SessionsClient({
      projectId,
      credentials,
    });

```

## Project ID

Define Project ID and Session ID in Application 

- You can find your project ID in your Dialogflow Agent Settings

```javascript

    const projectId = '<project-id-here>';
    const sessionId = '<put-chat-session-id-here>';

```


## Session Management - Redis  

### Redis Server Installation
https://docs.redis.com/latest/rs/references/client_references/client_nodejs/

- To use Redis with Node.js, you need to install a Node.js Redis client. The following sections explain how to use node_redis, a community-recommended Redis client for Node.js.

- Another community-recommended client for Node.js developers is ioredis. You can find additional Node.js clients for Redis in the Node.js section of the Redis Clients page.

### Install node_redis 

```
$ npm install node_redis
```

## Connect to Redis Instance
- Run the following code in your service.js backend file to create a Redis Instance:

```javascript

const redis = require('redis');

```

## Create a new Redis Client

```javascript

const client = redis.createClient({
    socket: {
        host: '<hostname>',
        port: <port>
    },
    password: '<password>'
});

client.on('error', err => {
    console.log('Error ' + err);
});


```
https://www.sitepoint.com/using-redis-node-js/

## Install Redis Server:
- For Mac and Linux users, the Redis installation is pretty straightforward. Open your terminal and type the following commands:


```
$ wget https://download.redis.io/releases/redis-6.2.4.tar.gz
$ tar xzf redis-6.2.4.tar.gz
$ cd redis-6.2.4
$ make
```

## After the installation ends, start the server with this command:

```
$ redis-server
```

## If the previous command did not work, try the following command:

```
$ src/redis-server 
```

## Service Account Authentication
First you have to create a service account and download a .JSON format file of credentials on your local system. Now, there are three ways to use that credentials for authentication/authorisation in dialogflow library.

### Method 1
- Create an environment variable named 'GOOGLE_APPLICATION_CREDENTIALS'
- Set the value equal to the absolute path of that JSON credentials file.

```
$ export GOOGLE_APPLICATION_CREDENTIALS="<absolute-path-of-json-file>"
```

- Then, run your code. Google library will automatically load your credentials file 

### Method 2
- Assume that you know the absolute path of your JSON file and put that as value in below snippet of credentials_file_path variable.
-  You can find your project ID in your Dialogflow agent settings

```javascript

    const projectId = '<project-id-here>';
    const sessionId = '<put-chat-session-id-here>'; 
    const credentials_file_path = '<absolute-file-path-of-JSON-file>';

    // Instantiate a DialogFlow client.
    const dialogflow = require('dialogflow');

    const sessionClient = new dialogflow.SessionsClient({
      projectId,
      keyFilename: credentials_file_path,
    });

```


## Customer Onboarding
- The body of the request contains relevant information such as the user query, action, and parameters from the client-side.

```javascript
    
    const query = request.body.queryResult    
    const intent = request.body.queryResult.intent.displayName;
    const score = request.body.queryResult.intentDetectionConfidence;
    const fulfillmentText = request.body.queryResult.fulfillmentText;
    const session_path = request.body.session;
    const parameters = request.body.queryResult.parameters
    const action = request.body.queryResult.action;
       

```

## Extracting the Sesssion ID from Request:

```javascript

    const session_path = request.body.session;
    const session_array = session_path.split('/')
    const session_id = session_array[session_array.length -1]

```

### Save the First and Last Name of Customer in Session Cache

```javascript

    var FirstName = await parameters["first-name"]
    var LastName = await parameters["last-name"]
    try {
      const client = new Redis()
      const hashMap = {
          "FirstName": FirstName,
          "LastName": LastName
      }
      await client.hmset(session_id, hashMap)
      const sessData = await client.hgetall(session_id)
    } catch(err) {
  }

```

## Use Session Cache to Get Information about Customer
- Use Redis client to get information about the customer from the session cache, such as their first and last name, in order to personalize the interaction between the chatbot and the user.

```javascript
 
    if (action == "input.welcome") {
        const client = new Redis()
        const sessData = await client.hgetall(session_id)
        if (sessData.FirstName && sessData.LastName) {           
                var agentResponse = {
                    "fulfillmentText": `Hey, what's up ${sessData.FirstName}! What could I help you with today?`
                }
                response.json(agentResponse)            
        }    
    }

```






## Creating Appointments 

### Using Google Maps API:
- This feature maps the distance between a customer pickup location and a corporate business address in order to check whether the pickup distance falls within the range of 100 miles

```javascript
if (context_name == session_path + '/contexts/' + 'appointment-booking-create') {
  console.log("Calculating distance of the destination from driver location...")
  var API_KEY = process.env.DIRECTIONS_API_KEY;
  var axios = require('axios')
  var config = {
  method: 'get',
  url: `https://maps.googleapis.com/maps/api/directions/json?origin=${Origin}&destination=${Destination}&key=${API_KEY}`
  }
  axios(config).then(async function(res) {
  data = res.data
  
  distance = data["routes"][0]["legs"][0]["distance"]["text"]
  kilometers = data["routes"][0]["legs"][0]["distance"]["value"]
  console.log(`The distance between origin and destination is: ${distance}`)
  
          if (kilometers < 160394) {
              console.log("The destination is within range!")
              var jsonResponse  = {
                  "fulfillmentText" : `Perfect! I just looked up the address, it's only ${distance} away, which meets the fundraiser requirements. How many bags do you plan on selling?`
              }
              console.log("Attempting to store DestinationAddress in memory...")
              redis_client.set("DestinationAddress", Destination, function(err, response) {
                  if(err) {
                      console.log(err)
                  }
                  
              })
              console.log("Saved destination address to session cache...")
              const hashMap = {
                  "DestinationAddress": Destination 
              }
              try {
                  const client = new Redis()
                  await client.hmset(session_id, hashMap)
                  const sessData = await client.hgetall(session_id) 
                  console.log(sessData)
              } catch(err) {
                  console.log(err)
              }
          
          }
          else {
              console.log("The destination is out of range")
          
              var jsonResponse = {
                  "fulfillmentText" : "I'm so sorry, I just looked up the address on the map. It looks like that address is out of driving range, we're only allowed to book appointments that are within 100 miles of the corporate office. Would you prefer to book the appointment at a different location?"
              }
              console.log("Sent response from server to customer...")
              
          }
          response.send(jsonResponse)
  
      }).catch(function(error) {
          console.log(error)
      })
  }

```

### Validating a customer's phone number:
- This feature uses regex to validate the phone number that a customer provided to the Dialogflow agent

```javascript
  if (action == "appointment-booking-getphone") {
        const Phone = parameters["phone-number"]
        
        if (Phone.replace(/\D/g,'').length !== 10) {
            var jsonResponse = {
                "fulfillmentText": "Sorry, it looks like that phone number is invalid, could you tell me the full phone number, please?"
                }
                response.json(jsonResponse)
            
        } else {
            var jsonResponse = {
                "fulfillmentText": "Thanks! And the address of the location for the pick-up?"
                }
        try {         
            const client = new Redis()
            const hashData = { "Phone": Phone }
            await client.hmset(session_id, hashData)
            const sessData = await client.hgetall(session_id)
            redis_client.set("Phone", Phone, function(err, response) {
                if(err) {
                    console.log(err)
                }
            })
            } catch(err) {
                console.log("Error:", err)
            }      
        }
    }
```



  
