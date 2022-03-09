## Develop, test, and deploy Cloudflare Workers with Denoflare

After spending most of my career working with Node.js, I was interested to hear about its counterpart, Deno. [Deno](https://deno.land/) is a different take on server-side JavaScript. Deno was designed [to correct the wrongs of Node.js](https://www.youtube.com/watch?v=M3BM9TB-8yA) and be a runtime that is safe and secure by default. 

## What is Deno?

Deno incorporates a few other aspects, such as a built-in TypeScript compiler, linter, formatter, and package manager. It also supports [ESM](https://blog.logrocket.com/how-to-use-ecmascript-modules-with-node-js/) [](https://blog.logrocket.com/how-to-use-ecmascript-modules-with-node-js/)[modules](https://blog.logrocket.com/how-to-use-ecmascript-modules-with-node-js/), uses the web platform as a base for its standard library, and has a secure-by-default design around IO. 

This results in a drastically simplified development experience when compared to Node.js. I started using Deno for a CLI tool I needed to write (using [deno-cliffy](https://github.com/c4spar/deno-cliffy)) and was pleased with the experience. When I first heard about D[enoflare](https://denoflare.dev/), I thought it would be the ideal opportunity to experiment with [Cloudflare Workers](https://workers.cloudflare.com/) while gaining more experience with Deno.

## Using Deno with Cloudfare Workers

Cloudflare Workers is an alternative to serverless infrastructure when compared to services like AWS Lambda. At its core, it acts like a serverless function, taking the request information, applying logic, and sending a response. 

The expected use-case is different in that it’s meant to act as a middleware, intercepting requests sent to an origin server, and applying logic. Cloudflare behaves similarly to [browser service worker](https://developers.google.com/web/fundamentals/primers/service-workers/)s.

![](https://paper-attachments.dropbox.com/s_BE214C9BDAEB2DFD8E7D323B5801786E774F2116262B2AC48FF5E2854FEFE528_1640978662490_Untitled+Diagram.drawio.png)


Cloudflare Workers is not full of isolated containers being spun up, providing the benefit of zero cold-starts. Some other nice features of Cloudflare Workers are global CDN deployments and paying on a per-request basis.

## What is Denoflare?

Denoflare is a small framework that lets you publish Cloudflare Workers written in Deno. It’s a natural fit, as both Deno and Cloudflare Workers follow standardized web platform runtime APIs.

Denoflare lets you serve your worker locally to test your changes in an isolated environment that is similar to how Cloudflare would run them. It supports a great developer experience with hot-reloading, being able to publish the worker right to the Cloudflare platform, tailing logs in production, and more. 

This has a similar experience to using another worker framework like [Miniflare](https://github.com/cloudflare/miniflare), except it's much simpler because Deno is doing most of the work. For example, instead of depending on Jest as Miniflare does, you can write tests to run with the native Deno test runner.

## Working with Cloudflare and Denoflare

The best way to experience Cloudflare and Denoflare is by going through a real-world example use case. Using minimal code, we will set up [A/B testing](https://en.wikipedia.org/wiki/A/B_testing) for a blog. Half the time, the user will be put into the test group and will see a new header. The others will be put into the control group and will not see the new header. 

Using Cloudflare Workers, this is as simple as intercepting requests to the blog origin server, placing the user into a group based on our split, and setting the response header `Set-Cookie` with the group name. After this is done, our blog can read the cookie to decide which header to show.


> We are omitting the code needed to change the header in the blog as there are several different ways you can do so.

## Setting up your Cloudflare Workers account

The first thing we must do [is sign up for a Cloudflare account with Cloudflare Workers set](https://dash.cloudflare.com/sign-up/workers) [](https://dash.cloudflare.com/sign-up/workers)[up](https://dash.cloudflare.com/sign-up/workers).
Once you are done, create a free .dev subdomain for testing our worker on. We will choose the free plan.

![Creating a free .dev subdomain for testing.](https://paper-attachments.dropbox.com/s_BE214C9BDAEB2DFD8E7D323B5801786E774F2116262B2AC48FF5E2854FEFE528_1640978721614_Xnip2021-12-31_09-33-30.jpg)


Creating a free .dev subdomain for testing.


> Note: You can change this later to be the domain you would use in production.

Denoflare requires an API token to allow us to push our compiled worker to Cloudflare. [Go](https://dash.cloudflare.com/profile/api-tokens) [](https://dash.cloudflare.com/profile/api-tokens)to [the tokens page](https://dash.cloudflare.com/profile/api-tokens) in the Cloudflare dashboard and select **Create Token**. Once there, you should choose **Edit Cloudflare Workers** as the template. Note this token for later.

From the overview screen, copy the account ID for use later.

## Developing with Denoflare

Deno provides a great development experience, which we will keep intact when using Denoflare.


> [Note](https://github.com/aleccool213/denoflare-blog-posthttps://github.com/aleccool213/denoflare-blog-post)[: You can find all the code in this GitHub repo.](https://github.com/aleccool213/denoflare-blog-posthttps://github.com/aleccool213/denoflare-blog-post)

First, set the Deno version and install it on your local machine using [asdf](https://asdf-vm.com/):


    echo "deno 1.16.0" > .tool-versions && brew install asdf && asdf plugin add deno && deno install

Let’s configure our IDE. If you are using Visual Studio Code, you will have a great experience with Deno. Install the Deno extension, [which can be found here](https://marketplace.visualstudio.com/items?itemName=denoland.vscode-deno). It enables type-checking, linting, and more.

To enable it, you must turn it on in the workspace settings file at `.vscode/settings.json`:


    {
      "deno.enable": true
    }

Next, we will install [Velociraptor](https://github.com/jurassiscripts/velociraptor), a script manager for Deno. Velociraptor makes it easy for all developers to use the same scripts when performing common tasks (think npm scripts).

Run this command in your console:


    deno install -qAn vr https://deno.land/x/velociraptor@1.3.0/cli.ts

We will define a Velociraptor script to invoke the Denoflare CLI, allowing us to run Denoflare commands such as `serve` and `push`.


    {
      "scripts": {
        "denoflare": "deno run --unstable --allow-read --allow-net --allow-env https://raw.githubusercontent.com/skymethod/denoflare/v0.3.3/cli/cli.ts"
      }
    }

Now that we have our IDE and environment runtime setup, we can move on to the code. With Deno, it’s common to declare all dependencies in a single file named `deps.ts`. This is because modules are defined by URLs and can become difficult to manage if scattered everywhere in a project.

We need one type of Cloudflare Workers, which Denoflare defines:


    export type { IncomingRequestCf } from "https://raw.githubusercontent.com/skymethod/denoflare/v0.3.0/common/cloudflare_workers_types.d.ts";

Once we have this type, we can write our A/B tester logic in `index.ts`:


    import { IncomingRequestCf } from "./deps.ts";
    
    /**
     * Based on the A/B Testing Cloudflare Worker example.
     * Ref: https://developers.cloudflare.com/workers/examples/ab-testing
     */
    function fetch(request: IncomingRequestCf): Response {
      const NAME = "experiment-0";
    
      const TEST_RESPONSE = new Response("Test group");
      const CONTROL_RESPONSE = new Response("Control group");
    
      // Determine which group this requester is in.
      const cookie = request.headers.get("cookie");
      if (cookie && cookie.includes(`${NAME}=control`)) {
        return CONTROL_RESPONSE;
      } else if (cookie && cookie.includes(`${NAME}=test`)) {
        return TEST_RESPONSE;
      } else {
        // If there is no cookie, this is a new client. Choose a group and set the cookie.
        const group = Math.random() < 0.5 ? "test" : "control"; // 50/50 split
        const response = group === "control" ? CONTROL_RESPONSE : TEST_RESPONSE;
        response.headers.append("Set-Cookie", `${NAME}=${group}; path=/`);
    
        return response;
      }
    }
    
    export default {
      fetch,
    };

The last step is to declare a `.denoflare` file, which Denoflare uses to run and publish your Cloudflare Worker:


    {
      "$schema": "https://raw.githubusercontent.com/skymethod/denoflare/v0.3.3/common/config.schema.json",
      "scripts": {
        "a-b-test-local": {
          "path": "index.ts",
          "localPort": 3030
        }
      },
      "profiles": {
        "account1": {
          "accountId": "INSERT_ACCOUNT_ID_FROM_PREVIOUS_STEP",
          "apiToken": "INSERT_API_TOKEN_FROM_PREVIOUS_STEP"
        }
      }
    }

That’s it! Let’s serve the Cloudflare Workers locally to make sure everything is working properly:


    vr denoflare serve a-b-test-local

This should be the output:


    Compiling https://raw.githubusercontent.com/skymethod/denoflare/v0.3.3/cli-webworker/worker.ts into worker contents...
    Compiled https://raw.githubusercontent.com/skymethod/denoflare/v0.3.3/cli-webworker/worker.ts into worker contents in 277ms
    runScript: index.ts
    Compiled index.ts into module contents in 142ms
    worker: start
    Started in 456ms (isolation=isolate)
    Local server running on http://localhost:3030

Now, open a browser and go to `localhost:3030`. You will see that the response is the group the user is put into:

![We were put in the test group!](https://paper-attachments.dropbox.com/s_BE214C9BDAEB2DFD8E7D323B5801786E774F2116262B2AC48FF5E2854FEFE528_1640978898683_Xnip2021-12-30_14-52-38.jpg)

## Testing the application

Because Cloudflare Workers is a Deno application, we can use its tools to validate and test our code. All of these run automatically as you are developing in Visual Studio Code, thanks to the extension you installed. 

Run the Deno linter with:


    deno lint

Make sure the Deno app compiles with:


    deno compile index.ts

Write a few tests and run them with:


    deno test index.test.ts
## Deploying Cloudflare Workers

You should have no trouble deploying Cloudflare Workers once you have the correct account ID and token in the Denoflare configuration (`.denoflare`). 

By default, Denoflare pushes your worker to the .dev subdomain you have set up for your account.
Deploy it to the Cloudflare instance by running:


    vr denoflare push a-b-test-local

You should see the worker appear instantly on your worker dashboard.

![The Cloudflare Workers dashboard.](https://paper-attachments.dropbox.com/s_BE214C9BDAEB2DFD8E7D323B5801786E774F2116262B2AC48FF5E2854FEFE528_1640978938130_Xnip2021-12-30_08-32-39.jpg)


By default, its public access is disabled. Go to the **service** **details** page and enable the route.

![Enabling the route.](https://paper-attachments.dropbox.com/s_BE214C9BDAEB2DFD8E7D323B5801786E774F2116262B2AC48FF5E2854FEFE528_1640978952334_Xnip2021-12-30_08-34-27.jpg)


When you go this route, you should see a response with what group this request gets put into.

![It works!](https://paper-attachments.dropbox.com/s_BE214C9BDAEB2DFD8E7D323B5801786E774F2116262B2AC48FF5E2854FEFE528_1640979005379_Untitled.png)

## Conclusion

Denoflare is a simple mini-framework built around Deno that allows us to easily publish Cloudflare Workers. 

Due to the way it implements web standards API and the similarity to Cloudflare Workers in the security model, Deno is a natural fit. Cloudflare Workers is a powerful way to deploy middleware logic to the cloud close to where users are using your applications due to their edge CDN strategy, where they deploy workers around the globe. 

[Have a look at other examples the Cloudflare team put together](https://developers.cloudflare.com/workers/examples) so you can get an even better idea of what else it can be used for.

