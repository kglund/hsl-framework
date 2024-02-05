# hsl-framework
An iRule framework to stand up application High Speed Logging on F5 Big-IP load balancers with a variety of profiles

## prerequisites

1. A vip must have either a TCP profile or an HTTP profile applied to process this framework.
2. These rules assume the existence of a HSL log publisher: https://my.f5.com/manage/s/article/K50040950
3. These rules are built with the assumption that the following string datagroups exist:
    - global_config -- general configurations, including HSL on/off switch
    - hsl_log_headers -- list of URIs OR VIPs on which header logging should be enabled
    - hsl_general_logger -- list of URIs OR VIPs on which detailed logging should be enabled
    - hsl_errlog -- list of URIs OR VIPs on which server errors should be logged
    - preferred_client_ciphers -- list of preferred ciphers for client-side handshake
    - preferred_server_ciphers -- list of preferred ciphers for server-side handshake

## applying the framework
This framework assumes the existence of the below data-groups.  Here are sample set-up commands to get off the ground:
```
    tmsh create ltm data-group internal global_config { type string records add { hsl_enabled { data false } hsl_slow_response_threshold { data 1000 } hsl_errlog_404 { data false } } }

    tmsh create ltm data-group internal hsl_log_headers { type string records add { / } }

    tmsh create ltm data-group internal hsl_general_logger { type string records add { / } }

    tmsh create ltm data-group internal hsl_errlog { type string records add { / } }

    tmsh create ltm data-group internal preferred_client_ciphers { type string records add { AES128-GCM-SHA256 { } AES256-GCM-SHA384 { } AES256-SHA256 { } ECDHE-RSA-AES128-GCM-SHA256 { } ECDHE-RSA-AES256-GCM-SHA384 { } ECDHE-RSA-CHACHA20-POLY1305-SHA256 { } TLS13-AES128-GCM-SHA256 { } TLS13-AES256-GCM-SHA384 { } TLS13-CHACHA20-POLY1305-SHA256 { } } }

    tmsh create ltm data-group internal preferred_server_ciphers { records add { AES128-GCM-SHA256 { } AES128-SHA { } AES128-SHA256 { } AES256-GCM-SHA384 { } AES256-SHA { } AES256-SHA256 { } ECDHE-ECDSA-AES128-GCM-SHA256 { } ECDHE-ECDSA-AES128-SHA { } ECDHE-ECDSA-AES128-SHA256 { } ECDHE-ECDSA-AES256-GCM-SHA384 { } ECDHE-ECDSA-AES256-SHA { } ECDHE-ECDSA-AES256-SHA384 { } ECDHE-RSA-AES128-CBC-SHA { } ECDHE-RSA-AES128-GCM-SHA256 { } ECDHE-RSA-AES128-SHA256 { } ECDHE-RSA-AES256-CBC-SHA { } ECDHE-RSA-AES256-GCM-SHA384 { } ECDHE-RSA-AES256-SHA384 { } ECDHE-RSA-CHACHA20-POLY1305-SHA256 { } TLS13-AES128-GCM-SHA256 { } TLS13-AES256-GCM-SHA384 { } } type string }
```

A VIP should have either the `hsl_tcp` or `hsl_http` irule applied to log HSL events.  For HTTP vips, you can additionally add `http_ssl` if you have SSL profiles applied, or `hsl_cache` if you have a caching profile applied.  These will enrich the log events defined in `hsl_http`.

## event catalog
Here is a quick description of application logging events exposed out-of-the-box
