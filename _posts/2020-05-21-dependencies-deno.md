# Handling dependencies with Deno

Github: [deno-check-updates](https://github.com/fzwael/deno-check-updates)

Deno: [deno_check_updates](https://deno.land/x/deno_check_updates)

## Introduction

Deno first stable version is just out and everyone went crazy about it, as shown in [bestofjs](https://bestofjs.org/projects/deno) Deno is getting an average of 700 stars daily!

That's why I gave it a try to see what's all the hype is about. So far I really like how straightforward and simple it is but one thing I didn't like is how handling the dependencies work.

## The easy way

In all docs/tutorials importing any deno dependency (standard or third party) would look something like this:
``` typescript
import { v4 } from "https://deno.land/std/uuid/mod.ts";
```
The code looks simple we are importing the uuid standard library from the given link but what if you look closely you can understand that deno.land is a 'mirror' for the deno project on Github and that link actually points here [mod.ts](https://github.com/denoland/deno/blob/master/std/uuid/mod.ts) and that's simply the code source in the master branch.

So the question here what if we want to use a specific version and not always the latest one ?

Easy! you can just change the import to :
``` typescript
import { v4 } from "https://deno.land/std@0.52.0/uuid/mod.ts";
```
As you can see by adding the @0.52.0 now we are pointing the dependency to the exact version 0.52.0 and this way we will avoid having any breaking changes and we can safely updated the dependencies manually.

## The Import Maps

As simple as it is, the first solution introduces a major inconvenient : If I use one dependency in 20 files, each time I updated I need to update them manually one by one!

To solve this problem Deno introduced an unstable [feature](https://deno.land/manual/linking_to_external_code/import_maps) (for now) which is [import maps](https://github.com/WICG/import-maps) as mentioned in the documentation you can use import maps with the `--importmap=<FILE>` CLI flag.

You will have to create an import file and use it like the following:
Example:

```js
// import_map.json

{
   "imports": {
      "http/": "https://deno.land/std@0.52.0/http/"
   }
}
```

```ts
// hello_server.ts

import { serve } from "http/server.ts";

const body = new TextEncoder().encode("Hello World\n");
for await (const req of serve(":8000")) {
  req.respond({ body });
}
```

```shell
$ deno run --allow-net --importmap=import_map.json --unstable hello_server.ts
```

## Deno check updates

Although the import maps is still unstable and have some limitations (refer to [documentation](https://deno.land/manual/linking_to_external_code/import_maps)) but it makes handling the dependencies fairly easy. One limitation I noticed is that (unlike npm) there is no command to check if the dependencies are up to date or not.

That's why I've been working on [deno-check-updates](https://github.com/fzwael/deno-check-updates). using the module is easy you just need to run the command:

```shell
$ deno run -A --unstable https://deno.land/x/deno_check_updates/main.ts -f import_map.json
```

The script will parse the import_map.json file and list all the dependencies with the latest versions. Something like this :
```json
{
  "imports": {
    "soxa/": "https://deno.land/x/soxa@v1.0/",
    "soxa2/": "https://deno.land/x/soxa@v0.4/",
    "checksum": "https://deno.land/x/checksum@1.2.0",
  }
}
```

| name | module  | url  | version | latest | upToDate
| :---:| :-----: | :--: | :-----: | :----: | :----: |
| soxa  | soxa | "https://deno.land/x/soxa@v1.0/" |  "v1.0" | "v1.0" | true
| soxa2 | soxa | "https://deno.land/x/soxa@v1.0/" |  "v1.0" | "v1.0" | true
| checksum | checksum | "https://deno.land/x/checksum@1.2.0" |  "v1.2.0" | "v1.4.0" | false
| http | std | "https://deno.land/std@0.51.0/http/" |  "v0.51.0" | "v0.52.0" | false

This way you know which dependencies needs an update!