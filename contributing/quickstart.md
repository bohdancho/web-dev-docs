---
title: Quickstart
parent: Contributing
layout: default
nav_order: 1
---

{: .info}
> This assumes you are an experienced developer with the necessary dependencies - refer to [Detailed Contributing Guide](./detailed_contributing_guide) if you need more information.

There are 3 steps to run the WCA website locally with docker[^1]:

## (1) GET THE SERVER RUNNING
1. Clone and navigate to the repo
2. Run using docker[^2] with `docker-compose up` (or `docker compose up`, depending on your docker compose config)

Now the website will run locally. You have some options on how to set up the database and run tests.

## (2) DATABASE SETUP
There are two options here - a "test" database with generated data, or the production database (with sensitive information removed).
1. **Test database**: Quick to set up, but may not mirror real-world data well. `docker exec -it rails bash -c "bin/rake db:reset"`
2. **Exported database**: Takes 60-90 minutes to set up, but uses actual data from the live website. `docker exec -it rails bash -c "bin/rake db:load:development[reload]"`

## (3) RUNNING TESTS
Two options - running while your docker server is running, or running tests independently. 

### **RUNNING WHILE SERVER IS LIVE**
With your docker server running:
1. Create `WcaOnRails/.env.test.local` with the following contents (this overrides `.env.test`, which is set up for testing in Github Actions, which uses a different db configuration): 
    ```bash
    DATABASE_HOST=127.0.0.1
    DATABASE_PASSWORD=
    ```
1. Connect to the `rails` container using `docker exec -it rails bash`
2. You'll now be in the terminal shell - run `RAILS_ENV=test rspec` 

You may want to consider the following options to make the test suite run faster: 
- Use `spring` to preload Rails with `bin/spring` - this all test runs after the first significantly faster
- Set `SKIP_PRETEST_SETUP=true` in `.env.test.local` (you may need to create this file)
    - This will skip database pre-population with seed data, which is used by legacy tests and is not needed if you are running an isolated set of tests which only rely on factories. WE SHOULD NOT BE WRITING ANY NEW TESTS WHICH USE THE SEED DATA! Use factories instead.
- Add `--fail-fast` to have the suit terminate after the first failure
- Specify a folder/filename to limit how many tests get executed

A full command using these options would look like: `RAILS_ENV=test bin/spring rspec spec/features/register_for_competition_spec.rb --fail-fast`

### **RUNNING INDEPENDENTLY**
```
docker-compose exec wca_on_rails bash -c "RAILS_ENV=test bin/rake db:reset && RAILS_ENV=test bin/rake assets:precompile && bin/rspec"
```

### COMMON ISSUES WITH TESTING

#### 1. `id not found: 333`, full error output as follows: 
```
An error occurred while loading ./spec/models/light_result_spec.rb.
Failure/Error: self.c_find(id) || raise("id not found: #{id}")

RuntimeError:
  id not found: 333
```

Run `RAILS_ENV=test bin/rake db:reset` in your docker container - the error indicates an improperly populated database, which this command will resolve.


----

[^1]: The website can also be run with `rails server` - see [Running with Rails](/guides/running_with_rails)
[^2]: If you're using Visual Studio Code to develop, you can [attach it to the Docker container](https://code.visualstudio.com/docs/remote/containers) so that your extensions can take advantage of the Ruby environment, and so the terminal runs from inside the container
