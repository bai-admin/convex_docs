---
title: "Serving Files"
sidebar_label: "Serve"
sidebar_position: 3
---

import Messages from "!!raw-loader!@site/../demos/file-storage/convex/messages.ts";
import AppTS from "!!raw-loader!@site/../demos/file-storage/src/App.tsx";
import AppJS from "!!raw-loader!@site/../private-demos/snippets/src/fileStorageApp.jsx";
import Http from "!!raw-loader!@site/../demos/file-storage-with-http/convex/http.ts";
import HttpAppTS from "!!raw-loader!@site/../demos/file-storage-with-http/src/App.tsx";
import HttpAppJS from "!!raw-loader!@site/../private-demos/snippets/src/fileStorageWithHttpImage.jsx";

Files stored in Convex can be served to your users by generating a URL pointing
to a given file.

## Generating file URLs in queries

The simplest way to serve files is to return URLs along with other data required
by your app from [queries](/docs/functions/query-functions.mdx) and
[mutations](/docs/functions/mutation-functions.mdx).

A file URL can be generated from a storage ID by the
[`storage.getUrl`](/api/interfaces/server.StorageReader#geturl) function of the
[`QueryCtx`](/api/interfaces/server.GenericQueryCtx),
[`MutationCtx`](/api/interfaces/server.GenericMutationCtx), or
[`ActionCtx`](/api/interfaces/server.GenericActionCtx) object:


```typescript
import { query } from "./_generated/server";

export const list = query({
  args: {},
  handler: async (ctx) => {
    const messages = await ctx.db.query("messages").collect();
    return Promise.all(
      messages.map(async (message) => ({
        ...message,
        // If the message is an "image" its `body` is an `Id<"_storage">`
        ...(message.format === "image"
          ? { url: await ctx.storage.getUrl(message.body) }
          : {}),
      })),
    );
  },
});

```


File URLs can be used in `img` elements to render images:


```typescript
function Image({ message }: { message: { url: string } }) {
  return <img src={message.url} height="300px" width="auto" />;
}

```


In your query you can control who gets access to a file when the URL is
generated. If you need to control access when the file is _served_, you can
define your own file serving HTTP actions instead.

## Serving files from HTTP actions

You can serve files directly from
[HTTP actions](/docs/functions/http-actions.mdx). An HTTP action will need to
take some parameter(s) that can be mapped to a storage ID, or a storage ID
itself.

This enables access control at the time the file is served, such as when an
image is displayed on a website. But note that the HTTP actions response size is
[currently limited](/docs/functions/http-actions.mdx#limits) to 20MB. For larger
files you need to use file URLs as described
[above](#generating-file-urls-in-queries).

A file [`Blob`](https://developer.mozilla.org/en-US/docs/Web/API/Blob) object
can be generated from a storage ID by the
[`storage.get`](/api/interfaces/server.StorageActionWriter#get) function of the
[`ActionCtx`](/api/interfaces/server.GenericActionCtx) object, which can be
returned in a `Response`:


```typescript
http.route({
  path: "/getImage",
  method: "GET",
  handler: httpAction(async (ctx, request) => {
    const { searchParams } = new URL(request.url);
    const storageId = searchParams.get("storageId")! as Id<"_storage">;
    const blob = await ctx.storage.get(storageId);
    if (blob === null) {
      return new Response("Image not found", {
        status: 404,
      });
    }
    return new Response(blob);
  }),
});

```


The URL of such an action can be used directly in `img` elements to render
images:


```typescript
const convexSiteUrl = import.meta.env.VITE_CONVEX_SITE_URL;

function Image({ storageId }: { storageId: string }) {
  // e.g. https://happy-animal-123.convex.site/getImage?storageId=456
  const getImageUrl = new URL(`${convexSiteUrl}/getImage`);
  getImageUrl.searchParams.set("storageId", storageId);

  return <img src={getImageUrl.href} height="300px" width="auto" />;
}

```

