============
prefect-helm
============

A helm chart for prefect.


Accessing the prefect server
============================

This chart does not expose the prefect user interface or its GraphQL API. After
deploying this chart, you can access them with:

.. code-block:: console

  $ kubectl port-forward service/apollo 4200 &
  $ kubectl port-forward service/ui 8080 &

Alternatively, you can setup a secure access with Pomerium as follows:

1.
