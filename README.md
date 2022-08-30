# GraphQL-Batching-Attack
BatchQL BatchQL is a GraphQL security auditing script with a focus on performing batch GraphQL queries and mutations. This script is not complex, and we welcome improvements.  When exploring the problem space of GraphQL batching attacks, we found that there were a few blog posts on the internet, however no tool to perform GraphQL batching attacks.  GraphQL batching attacks can be quite serious depending on the functionalities implemented. For example, imagine a password reset functionality which expects a 4 digit pin that was sent to your email. With this tool, you could attempt all 10k pin attempts in a single GraphQL query. This may bypass any rate limiting or account lockouts depending on the implementation details of the password reset flow.  Here’s what the output from BatchQL looks like when in detection mode:  ❯ python batch.py -e http://re.local:5000/graphiql -p localhost:8080  Schema suggestions enabled. Use Clairvoyance to recover schema: https://github.com/nikitastupin/clairvoyance CSRF GET based successful. Please confirm that this is a valid issue. CSRF POST based successful. Please confirm that this is a valid issue. Query name based batching: GraphQL batching is possible... preflight request was successful. Query JSON list based batching: GraphQL batching is possible... preflight request was successful. Most provide query, wordlist, and size to perform batching attack. Detections This tool is capable of detecting the following:  Introspection query support Schema suggestions detection Potential CSRF detection Query name based batching Query JSON list based batching Attacks Currently, this tool only supports sending JSON list based queries for batching attacks. It supports scenarios where the variables are embedded in the query, or where they are provided in the JSON input.  Using BatchQL Download the tool from GitHub: https://github.com/assetnote/batchql  Check out the README at the following URL for more instructions on how to use BatchQL: https://github.com/assetnote/batchql/blob/master/README.md  Introspection One of the powerful features of GraphQL is that comprehensive documentation can be generated for any GraphQL API through an introspection query. When introspection is possible, an attacker can obtain the GraphQL schema and understand the entire attack surface of the API.  Upon coming across a GraphQL API, the first step is usually to run an introspection query to obtain a copy of the schema. The schema will help with understanding the attack surface of the exposed GraphQL API.  We recommend using the following Chrome Extension to load up an interactive documentation view, if the introspection query is working: Altair Chrome Extension.  The raw introspection query can be found below:  Show Introspection Query  Suggestions So, the introspection query doesn’t work because it has been disabled. What now?   GraphQL Suggestions   The next step is to check whether or not the GraphQL API is returning schema suggestions. The suggestions feature can be leveraged to recover parts of the GraphQL schema.  Check out the presentation by Nikita Stupin where they discuss the process of recovering the GraphQL schema through the suggestions returned by the GraphQL API: GraphQL APIs from bug hunter’s perspective.  In addition to the presentation above, Nikita also released an excellent tool named Clairvoyance which is capable of recovering a GraphQL schema in an automated fashion based off the schema suggestions and further enumeration.  If you’re having issues recovering the schema using Clairvoyance, then give the fork Clairvoyancex a go. This fork also has support for a HTTP proxy, so that you can debug the recovery process as it runs.  We highly suggest you attempt to recover the schema when introspection is disabled. It will often inform you of the attack surface that may be exploitable to other issues.

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

TIME TO CONVERT INTO THE DOS WITH BURPSUITE (*NOTE NOT TESTED IN BUGBOUNTY PROGRAM)





