# CI/CD Pipeline Challenges

Trevor Westin - 2/8/2024

## Challenges

- Last Week
  - [How can we synchronize deployment of updates to Frontend, Backend, and Database?](https://github.com/BBITWestin/CI-CD-Pipeline-Challenges/blob/main/PipelineConcurrency.md)
- This Week
  - Once deployments are live, how to get user's frontend to udpate without asking them to refresh their web page.
  - Pipelines for Bugs and Features?

### Scenario 1

Using Blue/Green deployment we've just pushed new changes to the previously inactive prod branch and finished testing. It's time to switch users over to the new features but the client side will not be able to use these new features until they refresh. So how can we refresh for them? Or notify them to refresh? How should we handle this situtation?

### Goal

- Notify the clients frontend to refresh on deployments.
- Interupt the users workflow as little as possible.
- Handle all errors or avoid incompatibility between newly deployed backend and cleints unrefreshed frontend all together.

## Solution 1: Version Checking

The following implementations all involve some type of **version checking** where the frontend and backend compare a version number / hash. If we plan on using a Green/Blue Deployment strategy then when the version hashes don't match we can either...

1. Auto refresh the client side imediadtely
2. Or prompt the user to finish what they are working on and refresh afterwards since the old environment is still running and we can use the version number to direct the call to the correct backend.

But version checking and refreshing comes with some downsides regardless of the implementation:

- How do we version our deployments to be something the frontend and backend can use? Hard coded? Environment Variables? Fetch on page load?
- Simply triggering a refresh may not be enough to new changes from cashed resources like _server side components_. One work around to this is called `Cache Busting` where we would incorperate the version key / hash into file names forcing the frontend to update these cached resources but more research and testing is needed for this. See [here](https://medium.com/@soheilbakhtyari/understanding-cache-busting-a-practical-approach-for-react-razzle-applications-e3e5ddf0aeaf) for more details.

### Implementation 1: WebSockets

WebSockets are continuous connections. Websockets are typically used when the server needs to be able to send data to a client when the data becomes available.

#### Pros

- Server CPU load is proportional to how many sockets are busy at any given time as an idle socket does't take any CPU.
- Cheap ($2/day/1000 concurrent connections). See Azure Web [PubSub pricing](https://azure.microsoft.com/en-us/pricing/details/web-pubsub/).

#### Cons

- Keeping an open connection 24/7 feels like overkill for handling even daily deployments which likely won't be the case.

### Implementation 2: Server-Sent Events

Traditionally, a web page has to send a request to the server to receive new data; that is, the page requests data from the server. With server-sent events, it's possible for a server to send new data to a web page at any time, by pushing messages to the web page. These incoming messages can be treated as Events + data inside the web page.

To open a connection to the server to begin receiving events from it, create a new EventSource object with the URL of the SSE endpoint. For example:

```js
// JavaScript code on the client side
const eventSource = new EventSource("/api/stream/events");

eventSource.onmessage = function (event) {
  console.log("New message:", event.data);
};

eventSource.onerror = function (error) {
  console.error("EventSource failed:", error);
  eventSource.close();
};
```

```c#
// Implementing the SSE logic in .NET Web API project
using Microsoft.AspNetCore.Mvc;

namespace SseDemo.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class SseController : ControllerBase
    {
        [HttpGet("events")]
        public async Task GetEvents()
        {
            var response = Response;
            response.Headers.Add("Content-Type", "text/event-stream");

            for (var i = 0; i < 10; i++)
            {
                await response.WriteAsync($"data: Message {i + 1}\n\n");
                await response.Body.FlushAsync();
                await Task.Delay(1000);
            }
        }
    }
}
```

^^ Snippert from [Mastering Server-Sent Events](https://medium.com/@iamsalmankarim/mastering-server-sent-events-sse-in-net-6-f1848cf4c5e1)

#### Pros

- Fast and simple implementation.

#### Cons

- Just like websockets this strategy involves keeping open connections between server and client increasing server resource consumption and creates potential scalability challenges.
- How does this solve our problem? When the client side connects to the SSE endpoint after a backend deployment what events do we send? How are we differentiating between users who need to refresh or not?
- This is a one-way connection, so you can't send events from a client to a server.
- Browsers will automatically try to reconnect to an SSE stream if the connection is lost. However, managing these reconnections on the server side can add complexity. What happens if they disconnect during a SSE? Once they reconnect how do we know if they didn't get an event?

### Implementation 3: Polling

Polling involves the client regularly sending requests to the server to check for updates.

The frontend periodically checks to see if a newer version of the frontend is available. This requries the backend endpoint to remain constant between deployments.

#### Pros

- Fast and simple implementation.

#### Cons

- Network overhead
- Latency: If we're making this call every 60 seconds then we may have up to a 60 seconds of latency between deployment and refreshing the clients frontend leaving plenty of time for 404 errors in the case of incompatible frontend / backends.

### Implementation 4: Version-Based Conditional Request Handling

Rather than having a dedicated endpoint for polling, this implementation uses [HTTP Conditional Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests) to attach a version key from the frontend to each and every outbound call to the backend. Each time a request is made the backend compares the incoming version key it to it's own. If it matches continue the call per usual, but it they mismatch send back a certain error code that the frontend will recognise as mismatched versions triggering a notification to the user that they need to refresh.

This method is the same thing as polling but rather than having dedicated calls for checking version compatibility we just include the version in each call that's being made regardless.

#### So how do `HTTP Conditional Requests` work?

Conditional Requests are based on the use of specific HTTP request headers that the client includes to indicate the conditions under which the server should process the request. The most commonly used headers for Conditional Requests are:

- `If-Modified-Since`: The client sends the last known modification timestamp of the resource. The server will only send the resource if it has been modified since that time.
- `If-None-Match`: Works with ETags (Entity Tags). The server only sends the resource if the ETag does not match any of the ETags provided by the client.
- `If-Match` and `If-Unmodified-Since`: These headers are used to ensure that updates to a resource are only made if the client's version matches the server's version.

#### Demo

Backend Middleware Implementation (C# .NET): Add middleware to check if the ETag (which represents the version) in the incoming request matches the backend's current version. If they don't match, you'll return a custom response.

```C#
// VersionCheckMiddleware.cs
public class VersionCheckMiddleware
{
    private readonly RequestDelegate _next;
    // Hardcoded backend version, represented as an ETag
    private const string CurrentVersionETag = "1.0.0";

    public VersionCheckMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Headers.ContainsKey("If-None-Match"))
        {
            var incomingETag = context.Request.Headers["If-None-Match"].ToString();
            // Check if the incoming ETag (version) matches the current version
            if (!CurrentVersionETag.Equals(incomingETag))
            {
                // If versions don't match, return a custom status code to prompt refresh
                // Assuming 409 Conflict as the custom code for simplicity
                context.Response.StatusCode = 409; // Conflict
                await context.Response.WriteAsync("Version mismatch, please refresh.");
                return; // Prevent further processing
            }
        }

        await _next(context);
    }
}

// And in Startup.cs, add this middleware to the pipeline
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // Other configurations...

    app.UseMiddleware<VersionCheckMiddleware>();

    // Continue with the rest of the pipeline...
}
```

Frontend Implementation (React): On the frontend, you'll include the version ETag in each request. If you receive the custom response code indicating a version mismatch, you'll prompt the user to refresh.

```jsx
// ETagDemoCaller.js
import axios from "axios";
import config from "../config";

const baseURL = config.api.baseUrl;
const frontendVersionETag = '"1.0.0"'; // The current frontend version ETag

// Create an Axios instance with default headers, including the ETag
const ETagDemoCaller = axios.create({
  baseURL,
  headers: {
    "Content-Type": "application/json",
    "If-None-Match": frontendVersionETag, // Include the ETag header here
  },
});

// Response interceptor to handle version mismatches
ETagDemoCaller.interceptors.response.use(
  (response) => {
    return response;
  },
  (error) => {
    // Check if the error is due to a version mismatch
    if (error.response && error.response.status === 409) {
      // Notify the user about the version mismatch and suggest refreshing the page
      window.alert(
        "A new version of the application is available. Please refresh the page."
      );
      // Further error handling or rejection
    }
    return Promise.reject(error);
  }
);

export default ETagDemoCaller;
```

For [my demo](https://github.com/BBITWestin/pipeline-demo) I continued off of my last weeks auto db migration demo forked from the [todo-csharp-sql](https://github.com/azure-samples/todo-csharp-sql/tree/main/). I didn't bother deploying to azure for this demo as we can change the version numbers just fine locally. If you want to see an end-to-end implementation of an azure sample app check out [PipelineConcurrency.md #steps-to-recreate-demo](https://github.com/BBITWestin/CI-CD-Pipeline-Challenges/blob/main/PipelineConcurrency.md#steps-to-recreate-demo)

To run a azure sample app locally follow the steps [here](https://learn.microsoft.com/en-us/azure/developer/azure-developer-cli/debug?pivots=ide-vs-code).

See program.cs line 38 where we insert the `VersionCheckMiddleware` after CORS but before routing and endpoints are configured.

We put the `VersionCheckMiddleware` class in ./src/api/Middleware/VersionCheckMiddleware.cs using the same code snippet above.

On the frontend I defined `ETagDemoCaller` in ./src/web/src/services/ETagDemoCaller.js with the snipper above as well.

Start up the frontend and backend and click on the Red ETagDemoCaller button to test out different version numbers.

#### Notes

- The middleware will intercept the request before the controllers do so we don't need to worry about situations where the backend was updated to change or remove an endpoint.
- If we only need to make updates to the backend then we should't change the backend E-Tag as to not force the frontend to update.
- If we onlt need to make updates to the frontend then we need to change the backend E-Tag so that the client frontend refreshed!

#### Pros

- Simple, fast, easy.
- Reliable: The only time the versions won't match will be when the clients frontend hasn't refreshed since last deployment because the backend has a new version now.

#### Cons

- The client won't recieve the refresh notification until they try and make their first call after a deployment.
- If the client doesn't refresh after deployment by chance, then they will always get interupted once per deployment.

## Links

- [My Pipeline and Conditional Requests Demo](https://github.com/BBITWestin/pipeline-demo) forked from [Azure Sample App Repo](https://github.com/azure-samples/todo-csharp-sql/tree/main/)
- [Cache Busting](https://medium.com/@soheilbakhtyari/understanding-cache-busting-a-practical-approach-for-react-razzle-applications-e3e5ddf0aeaf)
- [Mastering Server-Sent Events](https://medium.com/@iamsalmankarim/mastering-server-sent-events-sse-in-net-6-f1848cf4c5e1)
- [HTTP Conditional Requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests)

## Coming Up:

- Authorization and Authentication

## Later On:

- Blue Green strategy working demo for entire CI/CD pipeline.
  - 1 Dev branch, 2 Prod branches (green/blue) environments.
  - Updated bicep files to handle these new environments.
  - Rollback demo
- Rollback demo independant of Blue Green deployment.
