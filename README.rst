============
prefect-helm
============

A helm chart for prefect.

Installation
============

You will need a kubernetes cluster and helm (version 3).

Clone this repo and install this chart with the following command-line.

.. code-block:: console

  $ helm install prefect ./charts/prefect

If this chart is useful for you and you are more versed in kubernetes and helm,
please help us by creating a repo for this chart.

Accessing the prefect server
============================

This chart does not expose the prefect user interface or its GraphQL API. After
deploying this chart, you can access them with:

.. code-block:: console

  $ kubectl port-forward service/apollo 4200 &
  $ kubectl port-forward service/ui 8080 &

Alternatively, you can setup a secure access with Pomerium as described in the
following section

Pomerium-based access
---------------------

Pomerium_ is an identity-aware proxy that can be used to access the Prefect
server UI web application and its GraphQL API. The pomerium documentation is
quite extensive, but here are some simplified instructions that have worked
for me.

These instructions assumes that:

* You are using Google as an identity provider (you can use another provider
  but please help us by updating these docs later).

* You want to deploy your UI on ``http://ui.example.com`` (you will change it
  for your own domain, of course).

* You have created certificates for your domain. You can follow the guide on
  https://www.pomerium.io/docs/reference/certificates.html


The procedure is as follows:


1. Deploy prefect server using helm as described in the installation section.

2. Crea

2. Assuming that you want to deploy your UI on ``http://ui.example.com``, then
   you need to create some SSL certificates first.

2. Create an OAuth client credentials by following the Pomerium documentation in
   https://www.pomerium.io/docs/identity-providers/google.html

   You do not need to do the service account part.

   You will need the client ID and secret later.


2. Create and review a ``config.yaml`` file with your Pomerium configuration.
   The important keys are marked with ``# IMPORTANT``

   .. code-block:: yaml

    config:
      rootDomain: example.com                                        # IMPORTANT
      service:
        type: "NodePort"

      policy:
        - from: "https://ui.example.com"                             # IMPORTANT
          prefix: "/graphql"
          to: "http://apollo.default.svc.cluster.local:4200"
          allowed_domains:
            - "my-company.com"                                       # IMPORTANT

        - from: "https://ui.example.com"
          to: "http://example.default.svc.cluster.local:8080"
          allowed_domains:
            - "my-company.com"                                             # IMPORTANT

    authenticate:
      idp:
        provider: "google"
        clientID: "FILL ME.apps.googleusercontent.com"               # IMPORTANT
        clientSecret: "FILL ME"                                      # IMPORTANT
      service:
        annotations:
          "cloud.google.com/app-protocols": '{"https":"HTTPS"}'

    ingress:

      hosts:
        - "*.example.com"                                            # IMPORTANT
      secret:
        name: "pomerium-tls"
      annotations:
        "kubernetes.io/ingress.allow-http": "false"

    proxy:
      service:
        annotations:
          "cloud.google.com/app-protocols": '{"https":"HTTPS"}'

3.

3. (Optional) reserve a static IP address for your


.. _Pomerium: https://www.pomerium.io/
