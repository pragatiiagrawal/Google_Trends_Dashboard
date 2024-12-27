
# Google_Trends_Dashboard
## Power Bi Dashboard




After Pasting the codes into Power BI, make sure you replace your API Key in the code. If you don't know how to get the key, simply log in to https://serpapi.com/ and navigate to the account section where you'll find the key.

Keyword Limit : Maximum 5 Keywords can be passed


## Country API
This code is used to get data based on the Keywords
```bash
 let
    // Define the API endpoint
    apiUrl = "https://serpapi.com/search.json",


// Define the parameters
queryParams = [ engine = "google_trends", q = Keywords , data_type = "GEO_MAP", date = "today 5-y", tz="-330",api_key =Key],

// Combine the endpoint and parameters
fullUrl = apiUrl & "?" & Uri.BuildQueryString(queryParams),

// Make the HTTP request
response = Web.Contents(fullUrl),

// Parse the JSON response
jsonResponse = Json.Document(response),

// Convert the response to a table
dataTable = Table.FromRecords({jsonResponse}),

// Extract the relevant data
comparedBreakdownByRegion = dataTable{0}[compared_breakdown_by_region]
in
    comparedBreakdownByRegion
```
## Date API
This code is used to get data based on the date.

Keyword Limit : Maximum 5 Keywords can be passed
```bash
let
    // Define the base URL for the API call
    BaseUrl = "https://serpapi.com/search",

// Define the query parameters with engine, terms, data type, date, and time zone
QueryParams = [engine = "google_trends", q = Keywords,data_type = "TIMESERIES", date = "all", tz = "-330",    api_key = Key],

// Generate the full URL with query parameters
UrlWithParams = BaseUrl & "?" & Text.Combine(List.Transform(Record.FieldNames(QueryParams), 
    each _ & "=" & Uri.EscapeDataString(Record.Field(QueryParams, _))), "&"),

// Fetch data from the API
JsonResponse = Json.Document(Web.Contents(UrlWithParams)),

// Extract the "interest_over_time" part from the JSON response
InterestOverTime = JsonResponse[#"interest_over_time"]
in
    InterestOverTime
```
## Past 7 Days Searches API
Code for past 7 days keywords performance data.

Keyword Limit : Maximum 5 Keywords can be passed
```bash
let
    // Define the base URL for the API call
    BaseUrl = "https://serpapi.com/search",

// Define the query parameters with engine, terms, data type, date, and time zone
QueryParams = [engine="google_trends",q=Keywords, data_type = "TIMESERIES",date = "now 7-d",tz = "-330",api_key = Key],

// Generate the full URL with query parameters
UrlWithParams = BaseUrl & "?" & Text.Combine(List.Transform(Record.FieldNames(QueryParams), 
    each _ & "=" & Uri.EscapeDataString(Record.Field(QueryParams, _))), "&"),

// Fetch data from the API
JsonResponse = Json.Document(Web.Contents(UrlWithParams)),

// Extract the "interest_over_time" part from the JSON response
InterestOverTime = JsonResponse[#"interest_over_time"]
in
    InterestOverTime
```
## Related Keyword API
Code for related keywords where you'll get two categories of data: Rising Keywords and Top Keywords.

Keyword Limit : Maximum 1 Keywords can be passed
```bash
let
    // Define the API endpoint
    apiUrl = "https://serpapi.com/search.json",

    // Define the parameters
    queryParams = [engine = "google_trends", q = Related_Keywords, data_type = "RELATED_TOPICS", api_key = Key],

    // Combine the endpoint and parameters to create the full URL
    fullUrl = apiUrl & "?" & Uri.BuildQueryString(queryParams),

    // Retry logic with delay
    RetryRequest = (url, retries) =>
        let
            result = try Web.Contents(url, [
                Headers = [Accept = "application/json", #"User-Agent" = "Power Query"]
            ]) otherwise null,
            output = if result = null and retries > 0 then 
                        (Function.InvokeAfter(() => @RetryRequest(url, retries - 1), #duration(0, 0, 0, 5)))
                     else result
        in
            output,

    // Make the HTTP request and retry up to 3 times
    response = RetryRequest(fullUrl, 3),

    // Check if the request was successful
    responseText = if response <> null then Text.FromBinary(response) else error "Failed to fetch data from the API",
    jsonResponse = if response <> null then Json.Document(response) else error "Invalid JSON response",

    // Log and handle 'related_topics'
    relatedTopicsFieldExists = List.Contains(Record.FieldNames(jsonResponse), "related_topics"),
    relatedTopics = if relatedTopicsFieldExists then jsonResponse[related_topics] else error "No related topics found in the response"
in
    relatedTopics
```

