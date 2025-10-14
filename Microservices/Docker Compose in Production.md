# Why I Like Using Docker Compose in Production — Nick Janetakis

## Here's why I used Docker Compose in 2015 and still use it today to run a majority of the apps I build and deploy.

**Quick Jump:**

-   -   [Success Stories](#success-stories)
    -   [Questions and Answers](#questions-and-answers)
    -   [Isn’t Docker Compose Meant for Development?](#isnt-docker-compose-meant-for-development)
    -   [Does it Scale?](#does-it-scale)
    -   [How Do You Handle Deployments?](#how-do-you-handle-deployments)
    -   [How’s the Experience Been?](#hows-the-experience-been)
    -   [Demo Video](#demo-video)

I’ve been using Docker since about 2015 and even back then I was using Docker Compose in production. Fast forward 10+ years and I’m happily using it.

Just a heads up, this isn’t going to be an end to end how-to post. You can always check my example Docker starter apps for [Flask](https://github.com/nickjj/docker-flask-example), [Rails](https://github.com/nickjj/docker-rails-example), [Django](https://github.com/nickjj/docker-django-example), [Node](https://github.com/nickjj/docker-node-example) and [Phoenix](https://github.com/nickjj/docker-phoenix-example). They are set up to work in development and production with Docker Compose.

Funny enough 95% of the moving parts on deploying your app with Docker Compose is unrelated to Docker Compose, it’s about setting up a Linux server and picking a mechanism on how you you want to trigger deployments (`git push`, etc.).

The goal of this post is to plant a seed on using and trusting Docker Compose as well as going over a few things I really like about Docker Compose and why it’s my default pick to deploy an app to production.

There’s a time and place for Kubernetes and I’ve used it for quite a few things over the years but I typically reach for it when it really makes sense for a project, not by default.

### [#](#success-stories) Success Stories

Over the last decade, here’s a couple of types of apps that I helped deploy for clients. I can’t list every single one, instead I’ll pick a couple and include a few bits of info about the project:

-   Rails app that did a lot of data analysis, at its peak 40 AWS EC2 instances were used
    -   Postgres, nginx, Rails, Redis
    -   It was a ton of background work processing for a multi-million dollar business
-   Fintech company running a few Python + PHP services on individual servers
    -   MySQL, Apache, PHP FPM, Flask, Redis, Memcached
    -   The company did well for ~12 years and eventually got acquired by a ~$500 million dollar company
-   Flask app to collect info from data sources, unify it and sell it as a service
    -   Postgres, nginx, Flask, Redis
    -   A solo dev had a passion for a niche field and built up a side business which pays his NYC rent
-   Flask app to help folks find good items to drop ship
    -   Postgres, nginx, Flask, Redis
    -   A solo dev started this one and in 6 months was making $90,000 USD / month with a 95% profit margin and $100 / month hosting bill

The Flask ones make me happy because they were both developers who took my [Build a SAAS App with Flask](https://buildasaasappwithflask.com?utm_source=nj&utm_medium=website&utm_campaign=/blog/why-i-like-using-docker-compose-in-production) course, had a good idea, executed it and were rewarded for their efforts. All my course did was break down how to build an app with Flask, they did the rest.

All of these apps and more have a few things in common:

-   `git push` code to a Linux server
-   Run a few lines of shell script after pushing the code
-   Wait a few seconds
-   The app is deployed

Some of them ran their database in Docker as part of the Docker Compose project, others used a managed database. Some of them chose to split their web and worker onto separate servers. Using Docker Compose doesn’t always mean you’re locked into literally 1 server.

### [#](#questions-and-answers) Questions and Answers

Over the years, folks who have wrote in or took my Flask / Docker courses have asked questions about Docker Compose. I’m going to do my best to answer those questions here.

-   Why would you use Docker Compose, isn’t it meant for development?
-   Does it scale?
-   How do you handle deployments?
-   How’s the experience been?

Let’s go over these.

### [#](#isnt-docker-compose-meant-for-development) Isn’t Docker Compose Meant for Development?

At some point super early in Docker’s life, there was wording in the documentation that mentioned Docker Compose wasn’t ready for production but that’s long gone.

For me, it’s always been about the results. If the thing works, it works. With that said, I’m not a reckless maniac. I did put it through the ringer the best I could for the apps I was building and deploying, it worked. I felt confident enough to put myself on the line and use it for client work fairly early on too.

My definition of “works” is:

-   Can I spin up multiple containers in a way they stay up?
-   Has it consistently worked?
-   Is it not crashing in unpredictable ways?
-   Is it stable on multiple systems?
-   Does it have all of the features I need?

Sometime in 2015 all of those checkboxes were checked for me. One of the very first apps I deployed was an early stage portal to let folks buy the v1 version of my Flask course and it handled distributing the video course in a ~1gb zip file after confirmation of payment.

This was way different than the current platform which streams video but it was a real app that took a $199 payment from folks and gave them a digital product. I didn’t mention this app earlier in the success stories.

I wanted to call that out here because I’m a firm believer of using and trying things out for my own projects before I risk using them for clients. Of course exceptions are made if a client asks me to learn and use something on the job.

With all of that said, it works fine in production, pre-prod, CI or any other environments you want to spin up. I’d say this is a huge advantage for Docker Compose in general because there’s parity between all environments. I’ll make a separate post about that in the future.

### [#](#does-it-scale) Does it Scale?

This one comes up a lot.

I think for a large tier of applications, it doesn’t matter and you can vertically scale as needed which goes a long ways. That’s adding more CPU and memory resources to a server.

The default case is spinning up your Docker Compose project on 1 server. Maybe you use a managed database, maybe you don’t, it’s up to you. Most web apps are I/O bound, typically waiting for results back from a database.

If you can serve your p95 (95%th percentile latency) traffic under 100-150ms on a single $20 / month server, that could be good enough right?

Whether your traffic is 200 requests a day or 20,000 you can always dial the vertical scaling knob up if you need more.

DigitalOcean has servers up to 32 vCPUs with 256 GB of memory. Most of the apps I deploy run fine to serve production traffic on servers with 2-4 CPUs and 4-8 GB of memory.

If DO is too expensive for you on the higher end of resources, places like Hetzner can give you 48 vCPUs with 192 GB of memory and an NVME SSD for $320 / month (your prices may vary, I took this from their site in mid-2025). That’s a massive amount of compute resources!

Imagine a scenario where let’s say you have 10,000 paying customers. Depending on your stack and your app’s traffic characteristics you might be able to host that on a server with 2 CPU cores and 4 GB of memory. Even if you had to double that, that’s still totally reasonable if you’re making $20 / month for each customer ($200,000 / month total).

#### What About Unpredictable Large Traffic Spikes?

The above is great if you have reasonably predicable traffic and you don’t want to over provision your server and increase costs. If your app normally gets 1,000 requests a day but you also need to handle spikes up to a million requests a day that can come at random’ish times then maybe you could look into using something more cost effective than running a higher priced machine 24 / 7 to handle your max load.

The interesting thing there is though, something like Kubernetes is not a silver bullet. If you run Kubernetes on a cloud provider, let’s say AWS, you still end up having nodes that need to spin up and join your cluster before your app is capable of handling that load.

It doesn’t matter if you’re using managed nodes with auto-scaling groups, Karpenter or even Fargate. The end result is compute resources need to be available in your cluster, then your app needs to be rolled out to them and this takes time.

The out of the box experience might be many minutes before it’s ready.

All of this is possible to achieve where you can balance costs and lag time before resources are available but this is going to take a pretty big engineering effort to pull off. The takeaway here is don’t expect to pick Kubernetes, roll your face on the keyboard and have that solution in 2 hours of vibe coding.

### [#](#how-do-you-handle-deployments) How Do You Handle Deployments?

One of the easiest ways in my opinion is to use git along with a post receive hook.

Basically you set up a bare git repo on your server and now you can `git push prod main` to push a branch to your server. This can be done from your dev box or CI pipeline, it’s up to you. It works the same in both cases.

Your server receives that push and that kicks off a script of your choosing to do the things you need to do such as pull a new image, up your project and maybe run a database migration. You can even send a Slack notification out with the results.

The beauty of this process is everything is 95% the same across projects. That benefit is huge. It really is.

Since we’re dealing with Docker Compose commands it doesn’t matter if you’re deploying a Flask, Rails, Django, Node or whatever app.

It also doesn’t matter if your app is just a web app server or maybe it’s a web + worker + websocket + db + cache + full-text-search stack. These are all implementation details of your Docker Compose file. The deploy process is basically the same, and for the parts that are different you can add hooks in your process to do custom things before or after a deployment happens.

My go-to deploy stack is:

-   Terraform to create resources (servers, DNS records, etc.)
-   Ansible to get a Linux (Debian / Ubuntu) server configured / provisioned
-   Docker to run the apps
-   Git to perform the deploys

### [#](#hows-the-experience-been) How’s the Experience Been?

It’s been really good.

Not only has everything worked out well for the dozens of apps I’ve deployed over the years but the complexity has remained fairly low.

If I want to deploy a completely fresh new app to a server and have it secured over HTTPS the only code I have to write is configuring a few things with Ansible and waiting about 5 minutes for everything to spin up before I `git push prod main`, here’s the full config:

```
git_deploy__projects:
  - name: "hello"
    local_git_repo_path: "~/src/open-source/docker-rails-example"
    post_deploy_commands: ["./run rails db:migrate"]

pki__realms:
  - domains: ["hello.example.com"]

nginx__vhosts:
  - server_name: "hello.example.com"
    root: "/srv/hello/public"
    proxy_pass_url: "http://127.0.0.1:8000"
    cache_assets: true
```

Deploying a Flask app is no different really. I just modify the repo path and switch the `post_deploy_commands` to use `["./run flask db migrate"]`.

I like running nginx on my host directly btw. I’ve [written about that in the past](/blog/why-i-prefer-running-nginx-on-my-docker-host-instead-of-in-a-container).

The general purpose Ansible playbooks and roles I’ve created over the last 10 years have allowed that, but they are not magic. They install and configure all of the moving parts and set up automated processes to keep your server up to date, secure and healthy. This could be done with anything.

One day I’ll release them if I ever finish my [Deploy to Production](/courses/deploy-to-production) course. If you want to see that course, let me know because otherwise I don’t know if it will be released since AI is killing organic traffic to this site in a way where I’m not sure making courses will be sustainable moving forward. That is partly why I focused more on contract work instead of making new courses over the last few years.

Anyways, consider using Docker Compose everywhere, it’s great!
