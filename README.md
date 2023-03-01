# ![Eulerian logo](/img/eulerian_logo.png) 
# Install Eulerian on your Next.js application
This tutorial will be covering how to add Eulerian Analytics to your Next.js application. 
The implementation is straightforward and doesn't require any third-party library.

In 2 easy steps, this will show you how to get Eulerian Analytics set up.

### Prerequisites
For this tutorial, you will need:
* a Next.js project
* a Eulerian Analytics account

## Steps
### 1. Inject Eulerian Analytics tracking code into your Next.js application
First, you need to inject and configure Eulerian Analytics' Global site tag into your browser window.

Access the head element in your Next.js application and integrate the 'eulerian-analytics' script in it.
You could also create a custom document that you invoke from your head element.  
Please note that the tracking subdomain is generic in the following snippet ($tracking_subdomain), you need to replace it with your own domain.

Then, the simplest way to log pageviews in your Next.js app is to subscribe to your router and listen for the routeChangeComplete event.

To do so, go into your *_app.js* file and with useEffect, check for new events happening with your router. There are many types of events but we only care when users navigate to a new page successfully (routeChangeComplete).

You'll note that we pass 2 variables - *router* and *pageProps* - to our *etag* function (we'll define this function in the next step). Both variables can be useful for Eulerian data collection. Suit yourself if you need extra variables in there, just add them in the datalayer to have them at your disposal in the etag function.

  ```javascript

import { useEffect } from 'react'
import { useRouter } from 'next/router'

import * as etag from '../lib/etag'

export default function MyApp({ Component, pageProps }) {
  const router = useRouter()

  useEffect(() => {
    const datalayer = [ router, pageProps ];
    const handleRouteChange = (datalayer) => {
      etag.pageview(datalayer)
    }
    router.events.on('routeChangeComplete', handleRouteChange(datalayer))
    return () => {
      router.events.off('routeChangeComplete', handleRouteChange(datalayer))
    }
  }, [router.events])


  return (
    <>
      <Head />
      <ManagedUIContext>
    <Script
      id='eulerian-analytics'
      strategy="afterInteractive"
      dangerouslySetInnerHTML={{
        __html: `
(function(e,a){var i=e.length,y=5381,k='script',s=window,v=document,o=v.createElement(k);for(;i;){i-=1;y=(y*33)^e.charCodeAt(i)}y='_EA_'+(y>>>=0);(function(e,a,s,y){s[a]=s[a]||function(){(s[y]=s[y]||[]).push(arguments);s[y].eah=e;};}(e,a,s,y));i=new Date/1E7|0;o.ea=y;y=i%26;o.async=1;o.src='//'+e+'/'+String.fromCharCode(97+y,122-y,65+y)+(i%1E3)+'.js?2';s=v.getElementsByTagName(k)[0];s.parentNode.insertBefore(o,s);})('${tracking_subdomain}','EA_push');
        `,
        }}
    />
        <Layout pageProps={pageProps}>
          <Component {...pageProps} />
        </Layout>
      </ManagedUIContext>
    </>
  )
}
	
  ```

### 2. Step 2: Custom functions to track pageviews and events
Let's move on to the tracking piece of Eulerian Analytics. In order to correctly track your user's behaviours, you will need to log page views and optionally, specific events triggered in your application.

To do so, we suggest you create a lib folder where you will put all your code related to third-party libraries and inside, a etag.js file with two functions: one to log pageviews and the other events.

The pageview *datalayer* argument will simply hold variables passed to it in the previous step, so for us these are *router* and *pageProps*.

  ```javascript
// log the pageview with their datalayer
export const pageview = (datalayer) => {
  const [ router, pageProps ] = datalayer;
  const EA_data = [
    'url', router.pathname
  ];
  if (pageProps.product) {
        EA_data.push('pagegroup','product');
        EA_data.push('prdref',pageProps.product.id);
  }
  if (pageProps.products) {
        EA_data.push('pagegroup','category');
  }     
  window.EA_push(EA_data);
}

// log specific events happening.

export const event = (eventName) => {
  window.EA_push('event',[eventName]);
}
```

This is a very simple example of how the variable mapping can be done according to what is contained in pageProps.
A real life example will include more customizations for basket and order pages for instance.

### 3. Log specific events in your Next.js app

```javascript
import { useState } from 'react'

import * as etag from '../lib/etag'

export default function Home() {
  const [query, setQuery] = useState("");

  const search = () => {
    etag.event("search")
  }

  return (
    <div>
        <div>
          <input type="text" onChange={(event) => setQuery(event.target.value)}></input>
        </div>
        <div>
        	<button onClick={() => search()}>Search</button>
        </div>
    </div>
  )
}
```

**Congratulations! You have installed Eulerian in your Next.js app !**
		
## Licence
See the LICENCE file for details.
