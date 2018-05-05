[![Build Status](https://travis-ci.org/blue-yonder/azure-cost-mon.svg?branch=master)](https://travis-ci.org/blue-yonder/azure-cost-mon)
[![Coverage Status](https://coveralls.io/repos/github/blue-yonder/azure-cost-mon/badge.svg?branch=master)](https://coveralls.io/github/blue-yonder/azure-cost-mon?branch=master)
[![PyPI version](https://badge.fury.io/py/azure-costs-exporter.svg)](https://badge.fury.io/py/azure-costs-exporter)

ACE azure-costs-exporter
========================

Prometheus exporter for the Microsoft Azure billing API. Check out
[this blog post](https://tech.blue-yonder.com/public-cloud-cost-control/) for
more background about the idea and some nice screenshots.

Description
-----------

**azure-costs-exporter** is a web app, that is intended to be called by [Prometheus](https://prometheus.io) to export billing information from Azure. It will then return the available metrics in Prometheus compatible format.

The billing API in use is part of the "Enterprise Agreement (EA)" Portal. Hence, it is not available for pay-as-you-go 
subscriptions. The configuration requires an active EA with Microsoft.

Configuration
-------------

You need to create an `application.cfg` file. This file is essentially a Python file that defines
variables in regular Python syntax. The following variables need to be defined in order to record various
metrics.

Enterprise billing metrics
~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the following variables to the `application.cfg` file:

    ENROLLMENT_NUMBER="123456"
    BILLING_API_ACCESS_KEY="XXX"
    BILLING_METRIC_NAME="my_metric_name"

- `ENROLLMENT_NUMBER` is the unique ID that identifies a particular EA.
- The `BILLING_API_ACCESS_KEY` can be created in the [EA portal](https://ea.azure.com/) to gain
access to the billing API. Navigate to "Reports > Download Usage" and generate an API Access Key.
- `BILLING_METRIC_NAME` is the name of the time series that will be generated in Prometheus.

Allocated virtual machine metrics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the following variables to the `application.cfg` file:

    APPLICATION_ID='abcd'
    APPLICATION_SECRET='secret'
    AD_TENANT_ID='efgh'

    SUBSCRIPTION_IDS=['abcd', 'efgh']
    ALLOCATED_VM_METRIC_NAME='my_metric_name'

- `APPLICATION_ID` is the application ID of a Microsoft service principal / Azure Active Directory
  application / app registration. The [Microsoft documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)
  explains how all this means and how to obtain one and the corresponding values for `APPLICATION_SECRET`
  and `AD_TENANT_ID`.
- `APPLICATION_SECRET` is the application secret that is created during the Azure app registration.
- `AD_TENANT_ID` is the ID of your Azure Active Directory instance.
- `SUBSCRIPTION_IDS` is a _sequence_ (!) of subscription IDs that shall be monitored.
- `ALLOCATED_VM_METRIC_NAME` is the name of the time series that will be generated for Prometheus

Please note that the application ID requires "Reader" permissions on each subscription that you want to
monitor.

Reserved virtual machine metrics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add the following variables to the `application.cfg` file:

    APPLICATION_ID='abcd'
    APPLICATION_SECRET='secret'
    AD_TENANT_ID='efgh'

    RESERVED_VM_METRIC_NAME='my_metric_name'

- `APPLICATION_ID` is the application ID of a Microsoft service principal / Azure Active Directory
  application / app registration. The [Microsoft documentation](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-create-service-principal-portal)
  explains how all this means and how to obtain one and the corresponding values for `APPLICATION_SECRET`
  and `AD_TENANT_ID`.
- `APPLICATION_SECRET` is the application secret that is created during the Azure app registration.
- `AD_TENANT_ID` is the ID of your Azure Active Directory instance.
- `RESERVED_VM_METRIC_NAME` is the name of the time series that will be generated for Prometheus

Please note that the application requires "Reader" permissions on _each individual_ reservation _order_ to be
able to retrieve the information of the actual reservations within the reservation orders. That's permissions
for you :-D.


Deployment
----------

E.g. via `gunicorn`. In an activated `virtualenv`, you can do:

    pip install azure-costs-exporter gunicorn
    cp /path/to/application.cfg .
    gunicorn azure_costs_exporter:app

Tests
-----

The used python testing tool is [pytest](https://docs.pytest.org/en/latest/).
To run the tests:

```bash
mkvirtualenv billing
pip install -r requirements_dev.txt
pip install -e .
py.test
```
