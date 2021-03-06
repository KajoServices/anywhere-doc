
# Anywhere API calls
## Root call
To see all available resources, go to: `/api/?format=json`
## Resource calls
Resource consists of data and schema.

Resource data: `/api/resource_name/?format=json`
Example: `/api/tweet/?format=json`

Resource schema: `/api/resource_name/schema/?format=json`
Example: `/api/category/schema/?format=json`

## Representation
Resource's default mode of representation is a list of objects.
Every single object has a `resource_uri` attribute, which leads to a detailed representation of a particular object.

In the list mode `meta` container displays limit and offset (see parameters below), urls for previous and next portion of data (in case limit and offset are used) and total number of records in the output.
## Parameters
### Format
    /api/resource_name/?param1_name=param1_value&param2_name=param2_value&...&paramN_name=paramN_value
Parameters that serve as filters, allow for modifiers. Each modifier can be applied to a field of a certain type
* `exact` - equality: *any type*
* `iexact` - equality, case insensitive: *strings*
* `exists` - *any type*
* `startswith` - *strings*
* `istartswith` - case insensitive "startswith": *strings*
* `endswith` - *strings*
* `iendswith` - case insensitive "endswith": *strings*
* `contains` - *strings*
* `icontains` - case insensitive "contains": *strings*
* `match` - matching pattern (can include \*, e.g. racoon\*): *strings*
* `in` - inclusion: *lists*
* `nin` - not in: *lists*
* `lt` - less than: *numeric values, dates, timestamps*
* `lte` - less than or equal: *numeric values, dates, timestamps*
* `gt` - greater than: *numeric values, dates, timestamps*
* `gte` - greater than or equal: *numeric values, dates, timestamps*
* `ne` - not equal: *strings, numeric values, dates, timestamps*

Format: `paramname__modifier=value`
Examples:

	...&description__startswith=RedLion
	...&description__contains=Controls

See the full list of available filters and their modifiers in a /schema of each endpoint.
### Common parameters
* __format__ - available formats: xml, json, yaml
* __username__ - not a param, but a part of authentication token (together with "api_key"). NB: this can also be sent as Authorization header.
* __api_key__ -  a part of authentication token (together with "username"). NB: this can also be sent as Authorization header.
* __limit__ - limits number of objects returned. Applicable only in case of detailed reports. Default: 36. Example: ...&limit=100
* __offset__ - number of records to skip from the beginning. Together with "limit" is used to divide data to pages (paginators). Default: 20. Example: ...&offset=40

## Endpoints
### Tweets
#### Plain list of tweets (GET)
    http://hostname/api/tweet/?username=username&api_key=api_key
#### Sorting tweets
By default news items are sorted by the field *created_by*, descending (latest first).
Custom sorting is performed using parameter `order_by` followed by the name of the field.

Examples.

Sorting by country:
    http://hostname/api/tweet/?order_by=country

Sorting by flood_probability descending:
    http://hostname/api/tweet/?order_by=-flood_probability

Sorting by multiple fields is done by adding an *order_by* parameter for each field to sort by:
    http://hostname/api/tweet/?order_by=-created_by&order_by=-flood_probability

WARNING! Order matters. Consider the following examples:
Sort by `country`, and within a set of each country sort by `created_at` descending:

	...&order_by=country&order_by=-created_at
#### Filtering
Use names of fields for filtering in the same manners as parameters (see "Parameters" above):

    http://hostname/api/tweet/
        ?username=username
        &api_key=4f23...d3c4
        &country=FR

Filters can be combined:

    http://hostname/api/tweet/
        ?country=FR
        &lang=en
        &created_at__lte=2018-01-12T12:30

Filtered data can be then sorted:

    http://hostname/api/tweet/ \
        ?country=FR \
        &created_at__lte=2018-01-12T12:30 \
        &order_by=-created_at

#### Filtering by created_at
In addition to the standard modifiers (`__gt`, `__lte`, etc.) filtering by the field `created_at` can be performed using time-ranges. Time-range is a string that consists of two dates (start and end), divided by vertical bar (|):

    http://hostname/api/tweet/
        &created_at=2015-05-08T10:00|2015-05-09T12:15

It is possible to use both date- and time-stamps as values for ranges, and to combine them in the same query:

    http://hostname/api/tweet/
        &created_at=2015-05-08|2015-05-09T12:15

Time range can be specified in human readable format:

    http://hostname/api/tweet/
        &created_at=2 hours ago|now

It is possible to use other human readable keywords, e.g. "1 day ago", "January 12, 2017", "Saturday", etc.

Examples:

    http://hostname/api/tweet/
        &created_at=2012 Feb|yesterday

    http://hostname/api/tweet/
        &created_at=1st of Jul 2012|in 2 hours

**WARNING!** In those cases *two values* are necessary: start and end date (divided by vertical bar). The following example will cause **400 Bad Request**:

    http://hostname/api/tweet/
        &created_at=1 day ago

If the goal is to filter the records for the last day, use the following request:

    http://hostname/api/tweet/
        &created_at=1 day ago|now

Finally, there are reserved keywords, that don't require a pair of values: `today`, `yesterday`, `this week`, `last week`, `this month`, `last month`, `this year`, `last year`.

    http://hostname/api/tweet/
        ?country=IT
        &created_at=yesterday

Filters based on time-ranges and reserved keywords are *inclusive*, i.e. they automatically stretch filters from the beginning of the starting date (0:00, or 12am) to the end of the ending date (23:59:59 or 11:59pm). So, the following examples are equivalent:

    http://hostname/api/tweet/
        ?country=IT
        &created_at=2018-12-31|2018-12-31

    http://hostname/api/tweet/
        ?country=IT
        &created_at=2018-12-31T00:00:00|2018-12-31T23:59:59

#### Search
Search is a special case of filtering. Use parameter `&search=` for searching:
http://hostname/api/tweet/?search=policy

Search are performed by the text (or list of items) stored the following fields (properties): `text`, `tokens`, `place`, `user_name`, `user_location`, `user_description`

If search by a single field (or by several, but not all) is required, use `match` modifier on a chosen field(s):

    http://hostname/api/tweet/
        ?country=IT
    	&text__match=cruis*
    	&place__match=verona

#### Search with filtering
Search can be combined with filters:

    http://hostname/api/tweet/?countries=United%20States
    	&flood_propbability__gte=0.6
    	&created_at__lte=2018-01-12T12:30
    	&search=don't trust cruise ships
    	&order_by=country
    	&order_by=-created_at

#### Aggregated data (GET)
Tweets can be aggregated by geo-location (path in the response content: `["features"][<docindex>]["geometry"]["coordinates"]`) and timestamp (path: `["features"][<docindex>]["properties"]["created_at"]`).

If any of aggregation parameter appear in the request, the response contains additional field "aggregations", where summarised number of documents are being gathered in "buckets", and sorted accordingly (see below).

If you are interested in aggregated results only, it is possible not to include original documents (tweets) entirely by setting `size` param to zero (see example below). In this case the field `features` will still be present in the response to comply with GeoJSON format, but it will be an empty list.

    http://hostname/api/tweet/
	    ?agg_hotspot=true
    	&size=0

##### Aggregation by geo-location

    http://hostname/api/tweet/
	    ?country=FR
    	&agg_hotspot=true
    	&agg_hotspot__precision=4
    	&agg_hotspot__size=1000

`agg_hotspot__precision` parameter (integer) takes values between 1 and 12 and indicates how precise an aggregation is on a map: *1* is 5,009.4km x 4,992.6km, *12* is 3.7cm x 1.9cm (full list - https://www.elastic.co/guide/en/elasticsearch/reference/6.2//search-aggregations-bucket-geohashgrid-aggregation.html#_cell_dimensions_at_the_equator). Default is 5. Be careful to use very high precision, as it is RAM-greedy.

`agg_hotspot__size` - maximum number of buckets to return (default is 10,000). When results are trimmed, buckets are prioritised based on the volumes of documents they contain (`doc_count`).

Buckets are sorted by `doc_count` descending (bigger at the top).

##### Aggregation by created_at (date-time histogram)
    http://hostname/api/tweet/
	    ?country=FR
    	&agg_timestamp=true
    	&agg_timestamp__interval=90m

`agg_timestamp__interval` defines interval for collecting tweets. Available expressions for interval:
* year - `1y`
* quarter - `1q`
* month - `1M`
* week - `1w`
* day - `1d`
* hour - `1h`
* minute - `1m`
* second - `1s`

Fractional time values are not supported, but it is possible to achieve the goal shifting to another time unit: e.g., `1.5h` could instead be specified as `90m`

**Warning**: time intervals larger than than days do not support arbitrary values but can only be one unit large, e.g. `1w` is valid, `2w` is not. For large intervals use days: for example, for a precision value of two weeks use `14d`.

Buckets are sorted by timestamps of the intervals, ascending.

##### Date-time histogram with average flood probability
    http://hostname/api/tweet/
	    ?country=FR
    	&agg_floodprob=true
    	&agg_floodprob__interval=90m

This indicates calculation of the average flood probability for each timestamp bucket. In the example above in each bucket (1.5hrs long) in addition to `doc_count` will contain `avg_flood_probability`.
#### User feedback (PATCH)
A registered user can leave a feedback for any particular tweet:

    PATCH http://hostname/api/tweet/<tweetid>/?api_key=<user_api_key>&username=<username-or-email>
    Content-Type: application/json
    Connection: keep-alive
    cache-control: no-cache
    data = '{
        "feedback": {
            "classes": [
	            "disaster",
                "fire",
                "forestfire"
                ],
            "relevant": true,
            "note": "This has nothing to do with a flood: it is about forest fire!"
        }
    }'

All the data specified in the `data` parameter will be added to all documents that satisfy the conditions, i.e. will have the same `<tweetid>` (there can be several documents in the system under the same `tweetid`, if, for example, there are several locations mentioned in the same tweet). The comment will automatically be stamped with a date-time mark and user's email. **Warning**: feedback don't have history, i.e. if a user can only *re-write* previously left comment, but cannot add a new one.  However, if a user wants to leave a feedback for a tweet that is already commented by someone else, a new comment will be added.

The structure of the feedback itself is free-form for as long as user keep all valuable information in a form of dictionary under the `feedback` key. However, for the procedural simplicity we suggest keeping the following keys, when giving a feedback:
*  `classes` - `<list>` of classes that are considered as relevant to the subject of a current tweet. This can either be a subset of original `tokens` key, or entirely new classes, or combination of both. By "class" here is meant a category or a keyword that somehow describes a topic of a given document. *Proposed implementation*: a checkbox'ed list of `tokens`, from which checked items become `classes` in the feedback.
* `relevant` - `<bool>` indicating whether a tweet (in its entirety) is relevant to the topic. This key is optional, if tweet is considered to be relevant, while a user wants to specify a list of classes (see above) or simply to leave a note.
* `note` - `<str>` (optional) comment on the reason of leaving feedback.