# Push-based Architectures with Angular & RxJS

> all this time you have been coding wrong!

![image](https://user-images.githubusercontent.com/17650005/212478720-476dddcf-e721-4b2b-b9a8-8ccb960303da.png)


Before I can show you HOW to implement Push-Based architectures, I need to first describe WHY Pull-Based solutions are flawed… and WHY Push-Based systems are better.

Most developers learn to program, code, and build software architectures using traditional Pull-based approaches. In the world of web applications and asynchronous, rich user experiences this approach is flawed, rampant with myriad problems, and is fundamentally wrong.

## Traditional Pull-Based Solutions
With Pull-based architecture, the views will explicitly call service methods to force-reload (aka ‘pull’) the data. This is the traditional approach employed by most developers.

Consider the simple function call findAllUsers() used to retrieve a list of users… here the code is pulling data to a view. If the user list subsequently changes, views (or services) must issue another pull request with findAllUsers(). But how do the views know when to issue another request?


> Notice I have not stated whether the data is currently in-memory or must be retrieved from the server. Nor have I asserted that the pull request is synchronous or asynchronous. Those are irrelevant here since this is still a pull-based approach.

![image](https://user-images.githubusercontent.com/17650005/212478775-5357f729-a45a-4147-9fe9-4e4f14559091.png)


Now it is reasonable to load data or change data asynchronously… using async/await or Promises. Developers may even build a temporary 1-response Observable [like HttpClient] to access remote data.

```ts
/** 
 * Examples of Pull-based Coding  
 * 
 */
 
// Synchronous Pulls

const response = pagination.updatePageSize(10);   // standard function call
const names = = this.cache.users.map(user => {    // array iteration
  return user.fullName;
}
 
// Asynchronous 1-x data Pulls 
 
const userClick = new Promise((resolve) => {      // async listening for event
  onClick = event => {
    window.removeEventListener(onClick);
    resolve(event.target);
  }
  window.addEventListener('click', onClick)
});
const users$: Observable<Users[]> = this.http.get<Users[]>(url);    

```

But those data request are still a single, 1-x pull-request. And Observables used this way are consider temporary streams; discarded after use/completion.

To design our system for long-term data flows, we could provide notification callbacks or even using polling (AngularJS digest cycle). These quickly become messy and [in some cases] even present performance issues.

And if our data is shared between multiple views then other HUGE architectural concerns must be considered:

- “How do 1…n views know when the data is updated?”
- “How do unrelated views get notified that new data is available?”
- “Should uncoupled view components poll for updated data?”
- “Why is my shared data changing? Who is changing the data when?


## Push-Based Architectures
So how do we invert the paradigm and change our coding to use Push-based solutions and architectures?


With RxJS and long-lived Observable streams, developers can implement architectures that PUSH data changes to all subscribers.

Views simply subscribe to desired data streams. When the datasource changes that desired data will be auto-pushed through the specified stream [to 1…n subscribing, interested view components].


```ts
/**
 * Examples of Push-Based Coding
 * Note: everthing uses a stream
 * 
 */
 
// Synchronous Pulls [from facade]
 
const pagination$ = userFacade.pagination$        
const criteria$ = userFacade.updateCriteria(      
  searchCriteria
);
 
 
// Asynchronous Pulls [from source]
 
const users$ = userFacade.users$
const results$ = userFacade.search(criteria);
```


> This approach to using permanent streams to push-data is a fundamental, HUGE paradigm shift from traditional pull-based architectures.

## Benefits
What are the benefits of designing and using Push-based architectures?

- State Management
- Immutability
- Performance
- Reactive Views

### State Management
With Push-based services, direct data access is prohibited. The true source of data is maintained within a virtual vault. The service itself provides an API that is intended to be used by the view layer.

Each Push-based service API has:

Properties: streams that will deliver data whenever that data changes.
Methods: to request changes to the data or request specific custom streams
The actual raw data is only available after it has been pushed through the stream(s).

This protected isolation centralizes all change management and business logic within the service itself and forces view components to simply react passively to incoming pushed-data.

### Immutability
With Push-based services, data (aka state) is always immutable. This effectively means that the data is read-only. Changes to the inside properties are not allowed.

Below is an example of pagination state and how developers would update an immutable object:

```ts
// `pagination` is immutable, so shallow-clone
// the instance with overrides to property values

updatePagination(selectedSize:number) {
  this.pagination = {
    ...this.pagination,
    selectedSize
  }
}
```

> When an immutable object needs to be modified then that object is cloned to a new instance with the desired properties/fields updated. After construction this object is also considered immutable.

With immutable, data change-detection using deep comparisons is not needed!

To detect changes, comparisons simply use === to determine if the data reference is a new instance. And when changes are detected, then the changed data is re-delivered through the streams to any subscribers.

Immutability == fast change-detection == fast performance.


### Reactive Views

With Push-based services that deliver data only through streams, developers are encouraged to create applications composed of passive view components. But what are passive views?

- Views that only render when the data arrives via a push-stream.
- Views that delegate user interactions to a non-view layer
- Views that require no business-logic testing
- Views that require minimal isolated UX testing

With Push-based services, Angular view components that are highly performant use both ChangeDetectionStrategy.OnPush and the async pipe with data delivered via streams.


> There are advanced patterns such as Redux or NgRx (etc) that provide all these features and more. We can, however, simply use RxJS and still build elegant, performant push-based solutions.

These concepts and issues impact developers in ALL technology platforms and frameworks and have huge impacts on developers of web applications. Whether the web app developer uses Angular, React, Vue.js, or another framework… Pull vs Push architectures affect everyone.

In the subsequent sections, let’s dive into Angular code to explore the concepts.



## To be continued...
