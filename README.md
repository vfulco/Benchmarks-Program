# Shorenstein Center Email Benchmarks

This is a tool developed by the Shorenstein Center at the Harvard Kennedy School to import MailChimp email list data, analyze it, and output the resulting metrics in an email report.

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

* [Python](https://www.python.org), version 3.5+ (3.6+ recommended)
* [RabbitMQ](https://www.rabbitmq.com/) or another AMQP broker
* A relational database, e.g. [SQLite](https://www.sqlite.org) or [PostgresSQL](https://www.postgresql.org/)
* [NodeJS](https://nodejs.org), version 10.11+
* [Amazon SES](https://aws.amazon.com/ses/)

### Local Development

##### Create a new virtual environment

    virtualenv venv
    source venv/bin/activate

##### Install Python dependencies

    pip install -r requirements.txt

##### Set environment variables

* `SECRET_KEY` - Flask secret key.
* `CELERY_BROKER_URI` - The URI of the Celery broker. Default `'amqp://guest:guest@localhost:5672/'` (a broker running locally on port `5672`).
* `SQLALCHEMY_DATABASE_URI` - The URI of the database. Default is a `sqlite` database named `app.db` located at the application root.
* `SERVER_NAME` - the URL for the app. Default `127.0.0.1:5000` (suitable for running locally). Note that the URLs for assets sent via email (images, etc.) are generated using Flask's `url_for()` function. If `SERVER_NAME` is not externally accessible these assets will not send succesfully.
* `AWS_ACCESS_KEY_ID` - AWS Access Key ID for the API.
* `AWS_SECRET_ACCESS_KEY` - AWS Secret Access Key for the API.
* `SES_REGION_NAME` - AWS Simple Email Service region.
* `SES_DEFAULT_EMAIL_SOURCE` - The default email address to send from. This email needs to be verified by SES and active outside the SES sandbox.
* `SES_CONFIGURATION_SET` - SES Configuration set for tracking opens/clicks/etc. Optional.
* `NO_PROXY` - We use proxies to distribute our MailChimp requests across IP addresses. Set this variable to `True` in order to disable proxying, or modify the `enable_proxy` method in `app/lists.py` according to your proxy setup.
* `ADMIN_EMAIL` - Email address to send error emails to. Optional.

##### Upgrade the database

    export FLASK_APP=app.py
    flask db upgrade

##### Install Node dependencies and compile the front-end source

    npm install
    gulp

##### Run the application

    flask run

##### Run Celery

    celery worker -A app.celery --loglevel=INFO

Finally, open a web browser and navigate to the `SERVER_NAME` URI.

## Linting

Lint the backend with `pylint`:

    pylint app

`Gulp` should automatically lint the front-end. Javascript rules are defined in `.eslintrc`.

## Deployment

This app is environment-agnostic. We deployed it on Ubuntu using `gunicorn` and `nginx`, and daemonized `Celery` and `Celery Beat`. Here are a few pointers on what we did.

A sample init script for gunicorn:

    [Unit]
    Description=Gunicorn instance to serve app
    After=network.target

    [Service]
    User=app_user
    Group=www-data
    WorkingDirectory=/path/to/app
    Environment="PATH=/path/to/app/venv/bin"
    ExecStart=/path/to/app/venv/bin/gunicorn --workers 5 --bind unix:email-benchmarks.sock -m 007 app:app

    [Install]
    WantedBy=multi-user.target

A sample init script for nginx:

    server {
        listen 80;
        server_name SERVER_NAME;

        location / {
            include proxy_params;
            proxy_pass http://unix:/path/to/app/email-benchmarks.sock;
        }
    }

Sample init scripts for Celery can be found in the [Celery repo](https://github.com/celery/celery/tree/master/extra/generic-init.d/).

## Authors

* **William Hakim** - [William Hakim](https://github.com/williamhakim10)

## Acknowledgements

This project is generously supported by the [Knight Foundation](https://knightfoundation.org/).

We use [Browserstack](https://www.browserstack.com/) to help ensure our projects work across platforms and devices.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details