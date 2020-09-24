# Tokens, Secrets, and Passwords

Passwords, secrets, and tokens should _never_ be written in code, included in source control, or included in built artifacts (such as docker containers).

How, then, can we do things such as call private API's and authenticate???

The most canonical way to deal with this problem is **environment variables**. Remember those, [from the brushup](https://github.com/bgsedatascience/module-computing#environment-variables)? They allow us to pass information from the environment (the _process_ in which the code is running on the computer on which it is running) into the code itself.

This is safe because, unlike with your code and build artifacts (which are sent around the internet and hosted various places), the environment is only visible on the server itself. Thus, for a malicious attacker to steal your passwords, they would need to have full access to your server.

Once you have an environment variable set, for example `MY_PASSWORD`, you can read them into your program like so:

``` python
import os

password = os.getenv('MY_PASSWORD')
```

This now gives you code that's safe to share and can be public. Nobody reading the code knows what your password is!

How do you set your environment variables? There are a lot of strategies, depending on your deployment platform.

1. You can create a file, traditionally named `.env`, and put it in the root of your project where you have all your variables. The file looks like this:

```
FOO=https://myfoobar.com?what=yes
BAR=08234lijlkfsj9u23
```
(remember, environment variables are traditionally uppercased, but that's just a convention)

Now you need to make sure this file exists both locally (in your "development" or "dev" environment) and on your server/deployment environment (remember, **don't include this file in source control -- add it to your .gitignore to be safe. You should also not include the file in docker containers you build and therefore should add it to .dockerignore as well**)

Once you have the file, you need to get the variables into the actual environment. There are several ways to do this:

* https://pypi.org/project/python-dotenv/. Many languages have a similar package. It's called "dotenv" because it is setup, be default, to look for a file called `.env`! It then loads the file and loads all the variables into the environment in which the code is running.

* Docker supports env files, in the same format as above, through a `--env-file` CLI flag (https://docs.docker.com/compose/env-file/).

* Just use the command `env $(cat .env | xargs) python myfile.py` when you develop or run your application. This should not be done from inside a Docker container, which should never contain your passwords and secrets inside of it, but is great for local development.

2. Docker has great support for environment variables. In addition to using it with and env-file, you can also declare individual environment variables via the `-e` flag (https://docs.docker.com/engine/reference/commandline/run/).

3. Most deployment platforms provide a way to provide environment variables, from [AWS Lambdas](https://docs.aws.amazon.com/lambda/latest/dg/env_variables.html) to [Kubernetes](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables) to [Heroku](https://devcenter.heroku.com/articles/config-vars).
