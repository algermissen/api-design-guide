# API Design Guide

## Characteristics of a well-designed API

In general, an effective API design will have the following characteristics:

- Easy to read and work with: A well designed API will be easy to work with, and its resources and associated operations can quickly be memorized by developers who work with it constantly.
- Hard to misuse: Implementing and integrating with an API with good design will be a straightforward process, and writing incorrect code will be a less likely outcome. It has informative feedback, and doesn’t enforce strict guidelines on the API’s end consumer.
- Complete and concise: Finally, a complete API will make it possible for developers to make full- fledged applications against the data you expose. Completeness happens over time usually, and most API designers and developers incrementally build on top of existing APIs. It is an ideal which every engineer or company with an API must strive towards.

[Swagger Best Practice API Design]: https://swagger.io/resources/articles/best-practices-in-api-design/

#API Structure & Design

## Dokument your API with OpenAPI (Swagger)

https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md

## Versioning our APIs

### URI Versioning

```
https://api.schwarz/v2/customers/`
```

### Querystring

```
https://api.schwarz/customers/3?version=2
```

### Header

```
GET https://api.schwarz/customers/3 HTTP/1.1
Custom-Header: api-version=1
```

## Organize your API at least Restful

https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm

##Organize the API around resources

Focus on the business entities that the web API exposes. For example, in an e-commerce system, the primary entities might be customers and  orders. Creating an order can be achieved by sending an HTTP POST  request that contains the order information. The HTTP response indicates whether the order was placed successfully or not. When possible,  resource URIs should be based on nouns (the resource) and not verbs (the operations on the resource).

## Avoid using verbs in URIs

**Do not** use something like this:

```
GET: /articles/:slug/generateBanner/
```

But the `GET` method is semantically sufficient here to say that we're retrieving ("GETting") a banner. So, let's just use:

```
GET: /articles/:slug/banner/
```

Similarly, for an endpoint that creates a new article:

**Don't**

```
POST: /articles/createNewArticle/
```

**Do**

```
POST: /articles/
```

## Use plural resource nouns

To prevent this kind of ambiguity, let's be consistent (life advice!) and use plural everywhere:

```
GET: /articles/2/
POST: /articles/
...
```

## Usage of HATEOAS, to enable navigation zu related resources

## Nesting resources for hierarchical objects 
**WIP** Nested resources should be used very wisely. Try to restrict yourself to 3 levels of nested resources.

## Define operations in terms of HTTP methods
[Swagger Best Practice API Design]: https://swagger.io/resources/articles/best-practices-in-api-design/
source: https://docs.microsoft.com/de-de/azure/architecture/best-practices/api-design

The HTTP protocol defines a number of methods that assign semantic  meaning to a request. The common HTTP methods used by most RESTful web  APIs are:

- **GET** retrieves a representation of the resource at  the specified URI. The body of the response message contains the details of the requested resource.
- **POST** creates a new resource at the specified URI.  The body of the request message provides the details of the new  resource. Note that POST can also be used to trigger operations that  don't actually create resources.
- **PUT** either creates or replaces the resource at the  specified URI. The body of the request message specifies the resource to be created or updated.
- **PATCH** performs a partial update of a resource. The request body specifies the set of changes to apply to the resource.
- **DELETE** removes the resource at the specified URI.

The effect of a specific request should depend on whether the  resource is a collection or an individual item. The following table  summarizes the common conventions adopted by most RESTful  implementations using the e-commerce example. Not all of these requests  might be implemented—it depends on the specific scenario.

| **Resource**           | **POST**                          | **GET**                             | **PUT**                                       | **DELETE**                       | Patch                           |
| ---------------------- | --------------------------------- | ----------------------------------- | --------------------------------------------- | -------------------------------- | ------------------------------- |
| /v1/customers          | Create a new customer             | Retrieve all customers              | Bulk update of customers                      | Remove all customers             |                                 |
| /v1/customers/1        | Error                             | Retrieve the details for customer 1 | Update the details of customer 1 if it exists | Delete the customer 1            | partial update of thee customer |
| /v1/customers/1/orders | Create a new order for customer 1 | Retrieve all orders for customer 1  | Bulk update of orders for customer 1          | Remove all orders for customer 1 |                                 |

The differences between POST, PUT, and PATCH can be confusing.

- A POST request creates a resource. The server assigns a URI for  the new resource, and returns that URI to the client. In the REST model, you frequently apply POST requests to collections. The new resource is  added to the collection. A POST request can also be used to submit data  for processing to an existing resource, without any new resource being  created.

- A PUT request creates a resource *or* updates an existing  resource. The client specifies the URI for the resource. The request  body contains a complete representation of the resource. If a resource  with this URI already exists, it is replaced. Otherwise a new resource  is created, if the server supports doing so. PUT requests are most  frequently applied to resources that are individual items, such as a  specific customer, rather than collections. A server might support  updates but not creation via PUT. Whether to support creation via PUT  depends on whether the client can meaningfully assign a URI to a  resource before it exists. If not, then use POST to create resources and PUT or PATCH to update.

  PUT requests must be idempotent. If a client submits the same PUT  request multiple times, the results should always be the same (the same  resource will be modified with the same values). POST and PATCH requests are not guaranteed to be idempotent.

- A PATCH request performs a *partial update* to an existing resource. The client specifies the URI for the resource. The request body specifies a set of *changes* to apply to the resource. This can be more efficient than using PUT,  because the client only sends the changes, not the entire representation of the resource. Technically PATCH can also create a new resource (by  specifying a set of updates to a "null" resource), if the server  supports this.



### Asynchrone Vorgänge

In einigen Fällen kann ein POST-, PUT-, PATCH- oder DELETE-Vorgang eine Verarbeitung erfordern, die einige Zeit in Anspruch nimmt. Wenn Sie vor dem Senden einer Antwort an den Client den Abschluss abwarten, kann dies zu einer nicht akzeptablen Wartezeit führen. Wenn dies der Fall ist, erwägen Sie die Verwendung eines asynchronen Vorgangs. Geben Sie den HTTP-Statuscode 202 (Akzeptiert) zurück, um anzugeben, dass die Anforderung zur Verarbeitung angenommen wurde, aber noch nicht abgeschlossen ist.

Sie müssen einen Endpunkt verfügbar machen, der den Status einer asynchronen Anforderung zurückgibt, damit der Client den Status durch Abfragen des Statusendpunkts überwachen kann. Nehmen Sie den URI des Statusendpunkts in den Location-Header der 202-Antwort auf. Beispiel:

```
HTTP/1.1 202 Accepted
Location: /api/status/12345
```

Wenn der Client eine GET-Anforderung an diesen Endpunkt sendet, muss die Antwort auf den aktuellen Status der Anforderung enthalten. Optional kann sie auch die geschätzte Zeit bis zum Abschluss oder einen Link zum Abbrechen des Vorgangs enthalten.

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "status":"In progress",
    "link": { "rel":"cancel", "method":"delete", "href":"/api/status/12345" }
}
```

Wenn der asynchrone Vorgang eine neue Ressource erstellt, muss der Statusendpunkt nach Abschluss des Vorgangs den Statuscode 303 (See Other [Siehe anderswo]) zurückgeben. Nehmen Sie in die 303-Antwort einen Location-Header auf, der den URI der neuen Ressource angibt:
HTTP

```
HTTP/1.1 303 See Other
Location: /api/orders/12345
```

Weitere Informationen finden Sie unter Asynchrones Anforderung-Antwort-Muster.

# Requests

### Specify the Content-Type header. 

Beispiel : application/json

##Handle trailing slashes gracefully

Both request should be handled the same way.

```
GET: /entities
GET: /entities/
```

## Allow filtering, sorting, and pagination

### Make use of the querystring for filtering and pagination

```
GET: /articles/?page=1&page_size=10
```

# Response

## Don't return plain text

return always a specified format

## Pay attention to status codes

The worst thing your API could do is return an error response with a 200 OK status code

### Use status codes consistently

Generally, stick to the following:

```
GET: 200 OK
POST: 201 Created
PUT: 200 OK
PATCH: 200 OK
DELETE: 204 No Content
```



| URL              | REST Verb | Action                                   | Success   | Failure   |
| ---------------- | --------- | ---------------------------------------- | --------- | --------- |
| v1/customers     | POST      | Create a new customer                    | 201 / 202 | 400 / 500 |
| v1/customers/new | GET       | Get the form for creating a new customer | 200       | 400 / 500 |


## Errors

### Handle errors gracefully and return standard error codes

### Return error details in the response body

return error details in the JSON body to help users with debugging

Problem JSON RFC SUCHEN!!! RFC-7808

```
{
  "error": "Invalid payoad.",
  "detail": {
    "surname": "This field is required."
  }
}
```

##Caching, provide caching information in the Header

## Security
### CORS Header
https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

### Validate Input

Always validate your user input