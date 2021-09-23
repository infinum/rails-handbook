Some API endpoints are designed to return a list of resources. We call those *list* or *index* endpoints.

Servers can in practice have thousands of resources and in theory return all of them when asked to, but this can be costly, slow and impractical. It's costly and slow for the server to fetch everything from the database and serialize it, and for the client to download it and parse the response. It is also impractical because in most cases users don't care about all resources and instead only want to see top results (e.g. if the endpoint is used for search).

Rather than showing everything at once, we can limit the number of resources returned in a request and provide a way to fetch the rest. In other words, we can page through resources.

There are different ways to enable pagination and to paginate resources, however the following paragraphs are limited to our most commonly used methods. That is not to say these are the only methods available, you are encouraged to use something else if it benefits your project.

## Enabling pagination

In most cases, pagination is enabled through the query string. The query string makes it possible for clients to pass additional parameters which are not suitable for sending in headers or the body. A URL with pagination applied might look like this: `/api/v1/countries?page=5`. In the example, we're fetching countries and requesting the 5th page of the results. The query string is the preferred way to enable pagination and you should also keep it enabled by default, so a request without the pagination query string would be handled as `page=1`.

## Types

There are many pagination types, and working with APIs you'll find different implementations. We'll stick to two types which are the most prevalent.

### Page-based

Page-based pagination divides a set of records into pages of equal size (except for the last page which can have fewer records than other pages). A number is assigned to every page, starting with 1.

![page-based-pagination](/img/page-based-pagination.png)

The API should support client-set page size and page number params. An endpoint with pagination params applied could look like this: `/api/v1/countries?page[size]=50&page[number]=3`. In the example, the client requests the 3rd page of countries with maximum 50 records.
This means that the API will respond with countries from 101 to 150, if the API database contains that many countries. Here are some cases when it doesn't:

- 120 countries: the client would receive only 20 countries (from 101 to 120)
- 160 countries: the client would receive 50 of them (from 101 to 150)

If the client omits the page size param, you should set a default value. You should also set a maximum value (so clients can't request all records by providing an absurdly large number). If a client requests more records than the maximum allows, you can either clamp the value to the maximum or respond with an error, whatever you think is the right choice.

When the page number param is omitted, you should respond with the first page. Respond with an error if the param is less than 1, but if it's greater than the total number of pages, then return the last page. Returning the last page instead of responding with an error prevents an edge case when pages disappear due to records being deleted but clients are still requesting the missing page (which they think exists).

The response should optionally include pagination metadata:

- current page (in case it differs from the requested page)
- total number of pages
- total number of items

This information is optional because it won't always be used by clients, and, since it typically requires executing an additional SQL COUNT query, it can be a costly operation.

Since this type of pagination allows clients to request any page, it is usually represented in the UI with a paginator where you can select a page (e.g. `<- 2 3 4 5 6 ->`). If this is the kind of UI you're implementing the pagination for, then the page-based type is the right choice.

This type is also popular because it's easy to implement using SQL's `OFFSET` clause, so you may often hear it being referred to as offset-based pagination. `page[size]=50&page[number]=3` simply translates to `LIMIT 50 OFFSET 50 * (3 - 1) => LIMIT 50 OFFSET 100` in SQL. However, offset-based pagination has a negative side. The clause requires scanning _all_ rows included by the offset value, which is inefficient for large offsets and causes performance to suffer the more pages you have (e.g. page 100 is slower to fetch than page 1). If you don't have to paginate many records, then it's not that much of an issue.

### Cursor-based

Cursor-based pagination, also called keyset pagination, uses a cursor which points to a specific record in the dataset and returns a number of records before or after it.

![cursor-based-pagination](/img/cursor-based-pagination.png)

An endpoint with pagination params applied could look like this: `/api/v1/countries?page[after]=dQw4w9`. `page[after]` can be replaced with `page[before]` if we want to retrieve records preceding a cursor.

Notice that the cursor in the example is a random unique string — this is intentional. Some implementations opt for opaque strings (which have no meaning to clients), while others use record IDs for cursors (take for example [Stripe API](https://stripe.com/docs/api/pagination)).

This pagination type should also support the client-set page size param (same rules apply as for the page-based type).

Due to the nature of this pagination type, you cannot show a paginator element in the UI (like you can for the page-based type). Based on the current page (which is defined with a cursor), clients can jump only page-by-page back or forth, but only one page at a time. This type is appropriate for the so-called infinite scroll design, where we load the next page when the client reaches the end of the current page. It can also be used in situations where users can paginate only with `Previous page` and `Next page` buttons.

## Common pitfalls

Always ensure that the records are sorted by a unique attribute (eg: ID). If the collection contains duplicate values of the attribute by which they are sorted, there may be occurrences when records appear to be missing.
Imagine a scenario where we have an API that responds with countries sorted by the number of counties and we only show 2 countries per page:

- Canada, 50
- Chile, 45
- Colombia, 45
- Denmark, 40
- Ecuador, 35

The 1st page will show Canada and Chile, whereas the 2nd page shows Chile and Denmark, instead of Colombia and Denmark. This issue should be fixed by sorting the collection by the count **and** some unique attribute of the record (especially if it also has an index).
