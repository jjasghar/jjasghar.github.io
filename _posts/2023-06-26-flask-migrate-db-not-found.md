---
layout: post
title: "flask migrate db not found"
date: 2023-06-26 12:49:11
categories: python flask
---

I've been getting deeper and deeper into [flask][flask] recently. I really enjoy using [flask-migrate][flask-migrate]
to deal with my Models, and I've discovered I run into this following error more often then I should.

If you see something like the following, here are the steps to help debug:
```bash
(venv) ➜ flask db init
Usage: flask [OPTIONS] COMMAND [ARGS]...
Try 'flask --help' for help.

Error: No such command 'db'.
```

1. Check your virtualenv, believe it or not your `(venv)` might be lying to you.

```bash
(venv) ➜  which flask
/Users/jjasghar/src/local/python/flask-app/venv/bin/flask
(venv) ➜  which pip
/Users/jjasghar/src/local/python/flask-app/venv/bin/pip
```
If this is ok, then go on to the next check.

2. Verify you have `flask-migrate` installed:

```bash
(venv) ➜ pip freeze
Flask-Migrate==4.0.4
```

If you don't you should do `pip install Flask-Migrate` again.

3. Verify that the `flask` cli can't find the `db` subcommand.

```bash
(venv) ➜  /Users/jjasghar/src/local/python/flask-app/venv/bin/flask
Usage: flask [OPTIONS] COMMAND [ARGS]...

  A general utility script for Flask applications.

  An application to load must be given with the '--app' option, 'FLASK_APP'
  environment variable, or with a 'wsgi.py' or 'app.py' file in the current
  directory.

Options:
  -e, --env-file FILE   Load environment variables from this file. python-
                        dotenv must be installed.
  -A, --app IMPORT      The Flask application or factory function to load, in
                        the form 'module:name'. Module can be a dotted import
                        or file path. Name is not required if it is 'app',
                        'application', 'create_app', or 'make_app', and can be
                        'name(args)' to pass arguments.
  --debug / --no-debug  Set debug mode.
  --version             Show the Flask version.
  --help                Show this message and exit.

Commands:
  routes  Show the routes for the app.
  run     Run a development server.
  shell   Run a shell in the app context.
```

If so, start checking your code, open up your `main.py` or `app.py`

4. Verify you have your "migrate" wired up correct, you _should_ have these in your code:

```python
from flask_migrate import Migrate


db = SQLAlchemy(app)
migrate = Migrate(app, db)
```

If you don't the `flask db` commands don't actually now what to migrate. After this verify the `db` sub command is there:

```bash
(venv) ➜  /Users/jjasghar/src/local/python/flask-app/venv/bin/flask
Usage: flask [OPTIONS] COMMAND [ARGS]...

  A general utility script for Flask applications.

  An application to load must be given with the '--app' option, 'FLASK_APP'
  environment variable, or with a 'wsgi.py' or 'app.py' file in the current
  directory.

Options:
  -e, --env-file FILE   Load environment variables from this file. python-
                        dotenv must be installed.
  -A, --app IMPORT      The Flask application or factory function to load, in
                        the form 'module:name'. Module can be a dotted import
                        or file path. Name is not required if it is 'app',
                        'application', 'create_app', or 'make_app', and can be
                        'name(args)' to pass arguments.
  --debug / --no-debug  Set debug mode.
  --version             Show the Flask version.
  --help                Show this message and exit.

Commands:
  db      Perform database migrations.
  routes  Show the routes for the app.
  run     Run a development server.
  shell   Run a shell in the app context.
```

If all of these things are there, you should be able to run the following and get your `init` off the ground:

```bash
(venv) ➜  flask-sqlite-template flask db init
  Creating directory '/Users/jjasghar/src/local/python/flask-app/migrations' ...  done
  Creating directory '/Users/jjasghar/src/local/python/flask-app/migrations/versions' ...  done
  Generating /Users/jjasghar/src/local/python/flask-app/migrations/script.py.mako ...  done
  Generating /Users/jjasghar/src/local/python/flask-app/migrations/env.py ...  done
  Generating /Users/jjasghar/src/local/python/flask-app/migrations/README ...  done
  Generating /Users/jjasghar/src/local/python/flask-app/migrations/alembic.ini ...  done
  Please edit configuration/connection/logging settings in '/Users/jjasghar/src/local/python/flask-app/migrations/alembic.ini'
  before proceeding.
```

[flask]: https://flask.palletsprojects.com/en/2.3.x/
[flask-migrate]: https://flask-migrate.readthedocs.io/en/latest/
