# The Solo Developer’s Secret Weapon: Self-Hosted Automation with n8n

## Automating My Life: How I Use n8n Instead of Custom Ruby Scripts

Very often I find myself falling into the classic developer trap.

I need a system that checks my Stripe account every morning, finds any failed payments, matches them to a user in my Rails database, and sends an alert to a private Discord channel.

My immediate instinct as a developer is: _“I can build that in an hour.”_

So, I create a new `StripeAlertJob` in Rails. I write a custom HTTP request to Discord. I schedule it with Sidekiq. It works perfectly.

But six months later, the Discord API changes. Or Stripe updates their gem. Or the job silently fails and I don’t notice. Suddenly, I am spending my Saturday morning debugging “glue code” that has absolutely nothing to do with my actual core product.

As a solo developer, your time is your most precious asset. You should not be writing and maintaining code for basic integrations.

In 2026, my solution to this is **n8n**. It is a visual, node-based automation tool (like Zapier), but built for developers. Here is why I stopped writing custom Ruby scripts and moved all my “glue” logic to n8n.

## What is n8n? (And why not Zapier?)

If you are a developer, you probably hate Zapier. It is insanely expensive (often costing hundreds of dollars a month for a busy app), the error logs are terrible, and doing complex logic like loops or custom HTTP requests feels clunky.

**n8n is different.**

1.  It is “fair-code” open source. You can self-host it on a cheap $5 Hetzner VPS using Docker.
2.  It allows you to write raw JavaScript inside the nodes if you need custom logic.
3.  It has built-in nodes for almost everything (Stripe, GitHub, Postgres, Discord, OpenAI).

Here is how I use it to replace my custom Ruby scripts.

## USE CASE 1: The “New User” Onboarding Flow

When a user signs up for my Rails SaaS, I want a few things to happen:

1.  Add them to a Beehiiv newsletter list.
2.  Send me a Slack/Discord notification.
3.  If their email ends in `@google.com` or `@microsoft.com`, flag them as a high-value lead in a Google Sheet.

**The Old Way (Rails):** I would install three different gems (`beehiiv`, `slack-notifier`, `google-drive-ruby`). I would write a massive `UserOnboardingService.rb` class and manage API keys for all three services in my `.env` file.

**The n8n Way:** I write exactly one line of code in my Rails app. I trigger an empty Webhook.

```
# app/controllers/users/registrations_controller.rb
def create
  @user = User.new(user_params)
  if @user.save
    # Just throw the user data at n8n and forget about it
    HTTP.post("https://n8n.mydomain.com/webhook/new-user", json: @user.as_json)
    
    redirect_to root_path
  end
end
```

Inside the n8n visual editor, I catch that Webhook. I drag and drop a line to the **Beehiiv node**, a line to an **If node** (checking for `@google.com`), and a line to a **Discord node**.

If any of those APIs fail or change, n8n handles the retry logic and sends me an error alert. My Rails app stays incredibly lightweight and completely unaware of external marketing tools.

## USE CASE 2: The Nightly Database Report

I love getting a daily summary of my business metrics (New users, MRR, Active sessions).

Normally, I would write a custom Rake task (`rake reports:daily`) and use a Cron job on my server to trigger it.

With n8n, I don’t even touch the Rails app. n8n has a native **PostgreSQL node**.

1.  I set a **Schedule Trigger** for 8:00 AM.
2.  I add a **Postgres node**. I give it read-only access to my production Rails database.
3.  I write raw SQL directly inside the n8n node: `SELECT COUNT(*) FROM users WHERE created_at > NOW() - INTERVAL '1 day';`
4.  I pass the result to an **OpenAI node** and tell the AI: _“Summarize these metrics into a friendly morning greeting.”_
5.  I pass the AI’s output to a **Telegram node** which messages my phone.

Zero Ruby code written. Zero gems installed.

## The Self-Hosted Workflow

Because n8n is just a Docker container, deploying it alongside your Rails app is incredibly easy, especially if you are using Kamal.

I just spin up a tiny $5 VPS specifically for my internal tools. I use Docker Compose to run n8n, point a subdomain to it (`automations.mycompany.com`), and I have an enterprise-grade automation engine running for the price of a cup of coffee.

## Summary

As engineers, our ego often tells us that writing code is always the best solution.

But every line of code you write is a line of code you have to maintain, test, and debug. If you are building a feature that provides unique value to your customers, write the Ruby code.

If you are just moving data from Point A to Point B, generating alerts, or syncing marketing lists? **Use n8n.**

Offloading the “glue work” keeps your Rails monolith clean, keeps your mind focused, and lets you automate your business visually.