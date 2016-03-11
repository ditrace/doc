Manual Installation
===================

.. _golang: https://golang.org/doc/install
.. _nginx: http://nginx.org/en/download.html
.. _elasticsearch: https://www.elastic.co/products/elasticsearch
.. _config.json: https://github.com/ditrace/web/config.json

There are following components you need to install before running DiTrace gate and UI:

1. golang_ version 1.5 or higher
3. elasticsearch_ version 2.2
4. web server e.g. nginx_

Install DiTrace gate
--------------------

.. code-block:: bash

   export GOPATH=<your gopath>
   go get github.com/ditrace/ditrace

Download Web UI Application
---------------------------

https://github.com/ditrace/web/releases/latest

Configure
---------

1. Place configuration file to the default location, ``/etc/ditrace/config.yml``

You can dive into :doc:`/installation/configuration` syntax on a separate page.

2. Place nginx configuration file to ``/etc/nginx/conf.d/ditrace.conf``

.. code-block:: text

    # elasticsearch cluster for traces
    upstream elastic {
        server vm-ditrace1:9200;
        server vm-ditrace2:9200;
        server vm-ditrace3:9200;
    }

    server {
        listen       0.0.0.0:80;
        server_name  vm-ditrace1;
        
        location / {
            root /var/local/www/ditrace/web/static/;
            index  index.html;
        }

        location /elasticsearch/ {
            rewrite      /elasticsearch/(.*)  /$1  break;
            proxy_pass http://elastic;
        }
    }

3. Place UI config.json_ file to ``/var/local/www/ditrace/config.json``

Run
---

1. Run nginx
2. Run elasticsearch
3. Setup indices template
   
.. code-block:: bash

   curl -XPUT http://elasticsearch:9200/_template/traces --data-binary @template.json
   
:download: <template.json>
       
4. Run ditrace gate

.. code-block:: bash

   $GOPATH/bin/ditrace --config=/etc/ditrace/config.yml