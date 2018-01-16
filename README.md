# Customer API

This API designed in RAML exposes a Customer API. This API exposes the following functions:-

	1. List customers
	2. Create a new customer
	3. Update a customer
	4. Delete a customer

The whole API design is modularised for easy understanding. The API is designed in RAML specification 1.0

The main file is customer-api.raml.

This file contains the definitions of the base URI, version and media type (JSON is used here).

The "lib" folder in the root directory contains the type definitions used in the API.

The "customers.raml" file defines  two types named customer and customers.

The "security" folder contains the custom security definition.

The "traits" folder contains the traits used in the API definition.

The "traits/secure.raml" defines the security header required to call the API. Here a JWT token is expected in the Header to invoke all the APIs. The APIs protected with "secured" trait will return a 401 HTTP response, if a valid token is not found.

The "traits/pageable.raml" defines the pagination details required as a query parameter. It expects a "page" and "per_page" parameters to define page number and records per page respectively. This query parameters are applied to GET request which tries to read all customer data. This trait prevents consumers from reading large volume of data at an instant by enforcing pagination.

The "traits/cacheable.raml" declares the API is cacheable for 300000 ms (5 min). The API returns the  resource's current representation along with its corresponding ETag value, which is placed in an HTTP response header "ETag" field. The client may  decide to cache the representation, along with its ETag.  Later, if the client wants to call the same API again, it will first determine whether the local cached version of the URL has expired (through the Cache-Control and the Expire headers). If the URL has not expired, it will retrieve the local cached resource. If it determined that the URL has expired (is stale), then the client will contact the server and send its previously saved copy of the ETag along with the request in a "If-None-Match" field.
On this subsequent request, the server may now compare the client's ETag with the ETag for the current version of the resource. If the ETag values match, meaning that the resource has not changed, then the server may send back a very short response with a HTTP 304 Not Modified status. The 304 status tells the client that its cached version is still good and that it should use that.

More details on ETag can be found here https://en.wikipedia.org/wiki/HTTP_ETag and here https://tools.ietf.org/html/rfc7232#section-2.3

The "examples" folder contains the sample JSON requests and responses used in the API.

The GET request provided at URL /customers also expects an optional query parameter named "postCode" to discourage consumers from requesting all data without an area code.

The API is designed keeping in mind the following use cases.

1.	A consumer may periodically (every 5 minutes) consume the API to enable it (the consumer) to maintain a copy of the provider API's customers (the API represents the system of record)
2.	A mobile application used by customer service representatives that uses the API to retrieve and update the customers details
3.	Simple extension of the API to support future resources such as orders and products

Use case - 1
------------
The "cacheable" trait explained above allows the consumers to cache the response for 300000 ms (5 min). The response body will only be send , if there is any change in data. The consumer only need to invoke the API every 5 min. This allows consumers who can cache response to maintain always a copy of response.

use case - 2
------------
The GET request has an optional parameter named "lastName" which allows customer API to be searched with a last name. API consumers like mobile applications are expected to use this parameter to minimise data received from the API, there by reducing network usage. This along with "cacheable" trait makes this API very efficient  for mobile applications.

use case - 3
------------
The Customer API is extended with Orders API and Product API. The raml files "order-api.raml" and "product-api.raml" defines the APIs for Orders and Product API. both order-api and product-api use RAML 1.0 Extension features.
"orders.raml" amd "products.raml" defined in "lib" folder define the required order and product data types.

Notes:-
-------
The APIs follow best practices in RESTful API design. The 'examples' folder contains JSON example files used in the API.
The order JSON files defines relationship to customer entity. A JSON link provides a URL to access the related resource.
For example Order ID 3125 is related to customer ID 1001. The API provider is expected to provide navigable links to associated resources in the body of the HTTP response message. This will help the consumer to make less calls to get a customer data for an order id. similarly the product to order relationship is defined in "product.json".

The design tries to balance the trade-off between Chatty I/O (Too many requests needed to fetch a resource) and Extraneous Fetching (fetching bulky data that the client doesn't need). More details on Chatty I/O and  Extraneous Fetching anti patters can be found here  https://docs.microsoft.com/en-us/azure/architecture/antipatterns/chatty-io/index and https://docs.microsoft.com/en-us/azure/architecture/antipatterns/extraneous-fetching/index

This design uses custom security to secure the APIs instead of OAuth 2.0 for simplicity.

RAML types are used instead of JSON and XML schemas. RAML 1.0 introduces the notion of data types, which provide a concise and powerful way of describing the data in an API. The RAML type syntax, is designed to be considerably easier and more succinct than JSON and XML schemas while retaining their flexibility and expressiveness.
