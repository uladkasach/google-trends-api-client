# google trends api client

An fully typed and easy to use client for the google trends api.

# overview

While google has no official client to their google trends api, they made it easy for folks to reverse engineer one.

This implementation
- 💪 is fully typed - for autocompletion, composability, and maintainability
- 📝 is jsdoc'ed - for docs-on-hover through your ide
- ✨ is simple - simple to use and simple to maintain

# usage

```
npm install --save google-trends-api-client
```

# examples

### autocomplete your input

When you open `trends.google.com` and begin typing in the "add a search term" field, google will autocomplete your search term.

Note: there are two types of `X` you can choose to get trends on:
- `search term`, a query google has compiled search trends for
- `topic`, a group of search terms which share the same concept or entity

```ts
import { trends } from 'google-trends-api-client';

const suggestions = await trends.getAutocompleteSuggestions({
  keyword: 'mattre',
});
expect(suggestions.length).toBeGreaterThan(0);
```

### get interest over time

```ts
const interest = await trends.getInterestOverTime({
  keywords: ['mattress recycling'],
});
expect(interest.length).toBeGreaterThan(0);
```

### get related queries
```ts
const queries = await trends.getRelatedQueries({
  keywords: ['mattress recycling'],
});
expect(queries.top.length).toBeGreaterThan(0);
expect(queries.rising.length).toBeGreaterThan(0);
```

### get related topics
```ts
const topics = await trends.getRelatedTopics({
  keywords: ['mattress recycling'],
});
expect(topics.top.length).toBeGreaterThan(0);
expect(topics.rising.length).toBeGreaterThan(0);
```

# Error 429 (Too Many Requests)

Google _will_ rate limit you if you attempt to make too many requests. The threshold seems related to
- ip-address
- cookies
- user-agent

The limit seems to be 1400 requests per ~24 hours.

You should not abuse google's api. It's a privilage that they have exposed it and made it public. There are some valid use cases, however, which can still reach the default rate limiting threshold.

To help fulfill valid use cases within the limits of the google api, we can:
- decrease the number of requests we make
- changing the rate limit that google grants us

Decreasing the number of reqeusts we make can be done by
- 1. using request caching, to prevent duplicate requests

Changing the rate limit that google grants us can be done by
- 2. changing your ip address with a `proxy` (i.e., reset your rate limit)
- 3. letting google track your agent across requests with `cookies` (i.e., increase your rate limit)
- 4. look like your reqeusts came from a real browser with `user-agent` + related headers (i.e., increase your rate limit)

This library makes it easy to do all of the above.

### 1. using request caching, to prevent duplicate requests

Using a cache to prevent duplicate requests is an easy way to prevent exceeding your rate

To do so, simply set the `requestCache` on the `agentOptions`. This library takes care of defining a unique key for each request.

```ts
import { createCache } from 'simple-on-disk-cache';

const suggestions = await getAutocompleteSuggestions(
  {
    keyword: 'mattress',
  },
  {
    requestCache: createCache({
      directory: { mounted: { `${__dirname}/__tmp__/requests` } }, // save in files in a directory called __tmp__/requests
      defaultSecondsUntilExpiration: 60 * 60 * 24 * 7, // expire the results after 7 days, for example
    })
  },
);
```

### 2. changing your ip address, with a proxy

Using a proxy allows you to change the ip address that the origin sees for requests from your agent.

Simply set the `proxyConfig` on the `agentOptions` to use it
```ts
const suggestions = await getAutocompleteSuggestions(
  {
    keyword: 'mattress',
  },
  {
    proxyConfig: config.proxy, // shape { username: string, password: string, host: string, port: number }
  },
);
```

### 3. letting them track your agent across requests, with cookies

In order to use cross request tracking, you need to enable saving the cookies that the origin tries to assign to you.

You can do this by defining a `cookiesCache` on the `agentOptions`.

```ts
import { createCache } from 'simple-on-disk-cache';

const suggestions = await getAutocompleteSuggestions(
  {
    keyword: 'mattress',
  },
  {
    cookiesCache: createCache({
      directory: { mounted: { `${__dirname}/__tmp__/cookies` } }, // save in files in a directory called __tmp__/cookies
      defaultSecondsUntilExpiration: 60 * 60 * 24 * 7, // expire the results after 7 days, for example
    })
  },
);
```

Note:
- your cache should persist the data to disk, so that it can be used across subsequent invocations of your code._

### 4. making your requests look like they come from a browser

This library uses [got-scraping](https://github.com/apify/got-scraping) under the hood which takes care of this for us.

However, if you would like to manually choose one, you can do so too, by setting it on the `agentOptions`

```ts
const suggestions = await getAutocompleteSuggestions(
  {
    keyword: 'mattress',
  },
  {
    userAgent: '...your user agent string...',
  },
);
```

Note:
- by default, a random user agent is generated on every request by got-scraping
- however, if you use a `cookiesCache`, this library will set a `sessionToken` that ensures the same user agent is reused until you clear your cookies

# references

- [google trends faq (support.google.com)](https://support.google.com/trends/answer/4365533?hl=en)
- [how to use trends for keyword research (seoscout.com)](https://seoscout.com/google-trends-seo-explained-how-to-use-trends-for-keyword-research)


# development

Google has made it easy for folks, assumingly on purpose, to reverse engineer their api, because:
- it's a clientside app
  - this means it makes it's requests to google's api from your browser
  - making it to see the requests it makes for each action
- there's no authentication
  - removing the need for any api keys or more complicated reverse engineering

To add an api call to this library:
- open up `trends.google.com`
- open up your devtools console
- open the requests tab
- conduct some action in the ui
- copy the request that your browser sent to their api
- replicate that request in node.js
- ???
- profit 😄