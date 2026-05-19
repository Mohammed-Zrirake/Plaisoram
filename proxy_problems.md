# Next.js API Proxy & Mixed Content Nightmares (And How We Solved Them)

This document serves as a post-mortem for a particularly frustrating bug in the Plaisoram architecture where media uploads would succeed, but the dashboard frontend stubbornly displayed "No media in this folder". 

This was caused by a combination of three distinct, overlapping issues that masked each other.

---

## 1. Next.js 14 Aggressive Fetch Caching (`X-Vercel-Cache: HIT`)

### The Problem
Next.js 14 App Router aggressively caches `fetch()` requests by default. 
When a user navigated to the media page for the first time before uploading any images, the Next.js API Proxy (`route.ts`) fetched the list of media from the DigitalOcean Symfony backend. Since the database was empty, it returned an empty array `[]`. Next.js immediately **cached** that empty array indefinitely.

When the user subsequently uploaded new files (which bypassed the cache because they were `POST` requests), the upload succeeded. However, when the dashboard refreshed and requested `GET /api/media` again, Vercel intercepted the request and returned the **cached empty array**, completely ignoring the new data in the database.

### The Solution
We explicitly disabled fetch caching inside the Vercel API proxy (`src/app/api/[...slug]/route.ts`) by passing `cache: 'no-store'` to the `RequestInit` options.

```typescript
  const fetchOptions: RequestInit = {
    method: request.method,
    headers,
    redirect: 'manual',
    cache: 'no-store', // This forces Next.js to bypass the cache
  };
```

---

## 2. Load Balancer SSL Termination (Mixed Content Error)

### The Problem
DigitalOcean App Platform sits behind a load balancer that terminates SSL. This means the external traffic is `https://`, but the internal traffic routed to the Symfony PHP container is `http://`.

Because Symfony wasn't explicitly configured to trust the internal IP addresses of the DigitalOcean load balancers, `$request->getSchemeAndHttpHost()` generated `http://` base URLs. When the user uploaded an image, the database recorded the URL as `http://plaisoram-server-gpdoc...`.

If the Next.js frontend (running securely on `https://`) attempted to render an `<img>` tag pointing to an `http://` source, the browser's strict security policies blocked it entirely (Mixed Content), resulting in broken images even if the array had loaded properly.

### The Solution
We added a forced override inside `MediaController.php` to guarantee the generated URLs use `https://` when running in the DigitalOcean environment:

```php
        $baseUrl = $request->getSchemeAndHttpHost();
        if (str_starts_with($baseUrl, 'http://') && str_contains($baseUrl, 'ondigitalocean.app')) {
            $baseUrl = str_replace('http://', 'https://', $baseUrl);
        }
        $media->setUrl($baseUrl . '/api/media/serve/' . $newFilename);
```

---

## 3. The Silent `net::ERR_CONTENT_DECODING_FAILED` Proxy Bug

### The Problem
Even after fixing the cache and the HTTPS URLs, the frontend still failed to render the images, and the network tab showed that the `GET /api/media` request was failing silently.

When the Next.js API Proxy requested data from the Symfony backend, Symfony returned the JSON payload compressed, including the HTTP header `Content-Encoding: gzip` (or `br`). 
Under the hood, the Node.js `fetch` API is smart enough to automatically decompress the response body for us. 

However, our API Proxy in `route.ts` was doing this:

```typescript
    const nextResponse = new NextResponse(response.body, {
      status: response.status,
      headers: response.headers, // DANGEROUS!
    });
```

Because we blindly copied **all** headers from Symfony, we passed `Content-Encoding: gzip` back to the browser. But because Node's `fetch` had already unzipped the payload, we were sending **plain text JSON** labeled as **gzipped**. 
When the browser received the response, it tried to unzip plain text, failed instantly, and threw a `net::ERR_CONTENT_DECODING_FAILED` error, causing the Javascript `fetch()` call to silently abort into the `catch` block.

### The Solution
We modified the API proxy to explicitly strip out problematic transport headers (like `content-encoding`, `content-length`, and `transfer-encoding`) before proxying the headers back to the browser. Next.js and Vercel will safely recalculate and re-encode these headers for the final leg of the journey.

```typescript
    const responseHeaders = new Headers(response.headers);
    
    // Crucial: Delete headers that conflict with Node's automatic decompression
    responseHeaders.delete('content-encoding');
    responseHeaders.delete('content-length');
    responseHeaders.delete('transfer-encoding');

    const nextResponse = new NextResponse(response.body, {
      status: response.status,
      headers: responseHeaders,
    });
```
