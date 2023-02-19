---
title: "Laravel Database Migration for Slack OAuth Bot Installations"
date: 2021-12-23T00:00:00+00:00
permalink: /laravel-database-migration-for-slack-oauth-bot-installations/
description: "Get the database table you need for your multi-workspace Laravel Slack bot."
tags: [php, laravel, slack]
---

Slack bots are ubiquitous nowadays. If you haven't created one, you should! They're sometimes useful, but always fun to make. There are so many ways to get started from the [series of "Bolt" SDKs from Slack](https://api.slack.com/tools/bolt) to all the great [community contributed packages](https://api.slack.com/community).

For one example, the official Python SDK has a [SQLAlchemy model](https://slack.dev/python-slack-sdk/api-docs/slack_sdk/oauth/installation_store/sqlalchemy/index.html) that is used for the installations database table. This is critical to track your bot installations if you plan on offering it up in more than just your own workspace.

Now I'm no php developer, but have taken a liking to Laravel recently and so am building another bot with it for the learning experience. So for you Laravel users out there, here's that useful SQLAlchemy table translated to a Laravel migration.

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schema;

class CreateSlackInstallationsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('slack_installations', function (Blueprint $table) {
            // PostgreSQL
            $table->uuid('id')->unique()->default(DB::raw('uuid_generate_v4()'));
            // MySQL
            // $table->uuid('id')->unique()->default(DB::raw('(UUID())'));
            $table->primary('id');
            $table->string('client_id', 32);
            $table->string('app_id', 32);
            $table->string('enterprise_id', 32)->nullable();
            $table->string('enterprise_name')->nullable();
            $table->string('enterprise_url')->nullable();
            $table->string('team_id', 32)->nullable();
            $table->string('team_name')->nullable();
            $table->string('bot_token')->nullable();
            $table->string('bot_id', 32)->nullable();
            $table->string('bot_user_id', 32)->nullable();
            $table->text('bot_scopes')->nullable();
            $table->string('bot_refresh_token')->nullable();
            $table->dateTime('bot_token_expires_at')->nullable();
            $table->string('user_id', 32);
            $table->string('user_token')->nullable();
            $table->text('user_scopes')->nullable();
            $table->string('user_refresh_token')->nullable();
            $table->dateTime('user_token_expires_at')->nullable();
            $table->string('incoming_webhook_url')->nullable();
            $table->string('incoming_webhook_channel')->nullable();
            $table->string('incoming_webhook_channel_id')->nullable();
            $table->string('incoming_webhook_configuration_url')->nullable();
            $table->boolean('is_enterprise_install')->default(false);
            $table->string('token_type', 32)->nullable();
            $table->dateTime('installed_at')->default(DB::raw('current_timestamp'));
            $table->timestamps();
            $table->index(['client_id', 'enterprise_id', 'team_id', 'user_id', 'installed_at']);
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('slack_installations');
    }
}
```

If you're encrypting your tokens (which you should!), you should make those fields the `text` type so as they don't overflow the 255 character limit for a default `string`.

Consider using the nice and simple package [`Crudly/Encrypted`](https://github.com/Crudly/Encrypted) for automatically encrypting and decrypting these fields.

```php
$table->text('bot_token')->nullable();
$table->text('bot_refresh_token')->nullable();
$table->text('user_token')->nullable();
$table->text('user_refresh_token')->nullable();
```

That's all for now, but it should give you a start. [Watch here](https://duffn.dev/atom.xml) for more Laravel Slack blog related posts in the future!
